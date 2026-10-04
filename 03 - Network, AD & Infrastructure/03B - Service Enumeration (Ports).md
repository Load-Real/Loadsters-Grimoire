---
status: working
tags:
  - network
  - enumeration
  - services
---
> *Per-port Service Playbooks:* What to do with each open port from [[Network Scanning]]. The rule that keeps you honest, every service must yield either a version (feeds [[CVE Triage]]) or an interface to test creds against (feeds [[Brute Forcing and Spraying]]). If it gives neither yet, you have not enumerated it enough. This note is a lookup, jump to the port you are looking at.

## FTP - 21

Always try anonymous first, it is a free win when it lands.

```bash
nc -nv $IP 21                                        # banner (version feeds CVE Triage)
ftp $IP                                              # creds: anonymous / anonymous, or blank
wget -m --no-passive ftp://anonymous:anonymous@$IP   # mirror everything
```

> [!note]
> In-session: `binary` before pulling executables/archives or they corrupt, `ls -la`, `get`, `mget *`, `put <shell>`, `!ls` to run a local command, `passive` to toggle if transfers hang. Key pivot: if the FTP root is writable and maps to the web root, upload a webshell and browse to it ([[File Upload to Webshell]]).

## SSH - 22

SSH itself is rarely the vulnerability, it is how you use creds found elsewhere.

```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" user@$IP
ssh -i id_rsa user@$IP                                # recovered private key
ssh2john id_rsa > id_rsa.hash && john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

> [!note]
> `chmod 600 id_rsa` before using a recovered key or SSH refuses it as too open. An encrypted key becomes a crackable hash via `ssh2john`, see [[Hash Cracking]].

## SMTP - 25 / 465 / 587

Enumerate valid usernames, which feed spraying and AD attacks.

```bash
nc -nv $IP 25                                         # banner + manual interaction
smtp-user-enum -M VRFY -U users.txt -t $IP           # automated user enum
```
Raw SMTP verbs inside the `nc` session:
```
HELO x
VRFY root            -- 250/252 = user exists, 550 = no
EXPN <list>          -- expand a mailing list to members
RCPT TO:<user>       -- alternate user-existence check
```

> [!note]
> `RCPT` and `EXPN` are fallbacks when VRFY is disabled. Swaks is the scriptable option for open-relay testing: `swaks --to user@corp.com --from x@x --server $IP`.

## DNS - 53

A zone transfer can hand you the whole internal host list at once.

```bash
dig axfr @$IP <domain>            # attempt a zone transfer
dnsenum <domain>                  # broader automated enumeration
nslookup ns1.<domain> $IP         # map a nameserver / discover DC hostnames
```

> [!note]
> Zone transfers are usually blocked, but always try. Add any hostnames you discover to `/etc/hosts` so name-based vhosts resolve ([[Web Enumeration]]).

## HTTP(S) - 80 / 443 / 8080 / 8443

Any web port goes to [[Web Enumeration]]. It is the most common foothold surface.

## Kerberos - 88

Port 88 open means this host is a Domain Controller. Treat the whole AD set as one objective, go to [[AD Enumeration]].

## RPC / NetBIOS / SMB - 111 / 135 / 139 / 445

A huge exam surface: shares, users, null sessions, EternalBlue.

```bash
nmap -p445 --script "smb-protocols,smb2-security-mode,smb-os-discovery,smb-enum-shares,smb-vuln-*" $IP
crackmapexec smb $IP                                  # quick OS/domain/signing glance
crackmapexec smb $IP -u '' -p '' --shares            # null-session share perms
enum4linux-ng -A $IP                                  # kitchen sink: users, shares, groups, pw policy
smbclient -L //$IP/ -N                                # list shares (null)
smbmap -H $IP -u '' -p ''                             # READ/WRITE per share
smbclient //$IP/share -N                              # connect and loot
crackmapexec smb $IP -u guest -p '' --rid-brute      # RID-cycle usernames when listing is blocked
```

> [!example] Authenticated SMB (re-run everything once you have a credential)
> ```bash
> smbclient -L //$IP/ -U 'CORP/forend%Klmcargo2'
> smbclient //$IP/Dev-share -U 'forend%Klmcargo2'
> smbmap -u forend -p Klmcargo2 -d CORP.LOCAL -H $IP
> smbmap -u forend -p Klmcargo2 -d CORP.LOCAL -H $IP -R SYSVOL --dir-only
> crackmapexec smb $IP -u forend -p Klmcargo2 --shares
> crackmapexec smb $IP -u forend -p Klmcargo2 -M spider_plus --share 'Dev-share'
> ```
> Inside smbclient: `recurse ON; prompt OFF; mget *` pulls a share recursively.

> [!note]
> `-N` is a null session. `--rid-brute` recovers usernames even when you cannot list shares. SMB signing disabled signals relay potential ([[Responder and NTLM Relay]], lab only). `smb-vuln-ms17-010` → EternalBlue ([[Famous Exploits Lookup]]). Run several tools, they disagree: crackmapexec → smbmap → smbclient → enum4linux-ng. Deeper RPC-based user/group enumeration lives in [[SNMP LDAP RPC]]. Loot readable/writable shares via [[Finding Credentials]].

## LDAP(S) - 389 / 636 / 3268 / 3269

Usually a DC. Anonymous bind sometimes dumps the directory.

```bash
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com"      # anonymous full dump
```

> [!note]
> Full command reference (base-DN discovery, authenticated user filter, the `description`/`info` cred hunt) is in [[SNMP LDAP RPC]]. Then continue in [[AD Enumeration]].

## MSSQL - 1433

A privileged DB user often goes straight to command execution.

```bash
crackmapexec mssql $IP -u sa -p '<pw>' --local-auth
impacket-mssqlclient sa:'<pw>'@$IP -windows-auth
```

> [!note]
> Inside the client, `enable_xp_cmdshell` (or `EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;`) turns on command exec, then `xp_cmdshell 'whoami'`. That runs as the SQL service account, which very often holds `SeImpersonate` → straight to [[Windows Privilege Escalation]].

## MySQL / MariaDB - 3306

Default/blank `root` is common.

```bash
mysql -h $IP -u root -p
mysql -h $IP -u root -pPassword123        # inline password, no space after -p
```
```sql
SHOW DATABASES; USE <db>; SHOW TABLES; DESCRIBE <table>; SELECT * FROM users;
SELECT user,authentication_string FROM mysql.user;   -- dump credential hashes
SELECT @@version; SELECT @@secure_file_priv;         -- version + whether file-write is allowed
SELECT @@datadir;
```

> [!note]
> `mysql.user` holds crackable account hashes ([[Hash Cracking]]). `secure_file_priv` empty/NULL means `LOAD_FILE`/`INTO OUTFILE` are usable for file read/write, see [[SQL Injection]].

## PostgreSQL - 5432

Try `postgres:postgres` first.

```bash
psql -h $IP -U postgres
psql "host=$IP user=postgres password=postgres dbname=postgres"
```
```
\l   list databases   \c <db>   connect   \dt   list tables   \du   roles (look for Superuser)
\d <table>   describe   SELECT * FROM pg_shadow;   -- hashes if superuser   SELECT version();
```

> [!note]
> If your role is Superuser, `COPY ... FROM PROGRAM` gives command execution, the standard Postgres RCE path.

## NFS - 2049

Mountable exports leak files or enable a root escalation.

```bash
showmount -e $IP
sudo mount -t nfs $IP:/export /tmp/nfs -o nolock
```

> [!note]
> An export marked `no_root_squash` is the prize, files you create as root on your box keep root ownership on the share, enabling the local-root technique in [[Linux Privilege Escalation]]. Even without it, exports often hold SSH keys, configs, backups ([[Finding Credentials]]).

## RDP 3389 / WinRM 5985 / VNC 5900 (creds-in-hand)

These turn a found credential into access.

```bash
xfreerdp /v:$IP /u:user /p:pass +clipboard /dynamic-resolution /cert:ignore
evil-winrm -i $IP -u user -p 'pass'
evil-winrm -i $IP -u user -H <NTLM>                  # pass-the-hash
evil-winrm -i $IP -u user -p 'pass' -s /opt/scripts/ -e /opt/exes/   # preload tool dirs
```

> [!note]
> WinRM (5985) plus a valid credential or hash is one of the easiest shells there is, always test it when you have a Windows credential. With `-s`, use `Invoke-Binary` / dot-source PowerShell (PowerView, Rubeus) straight into memory. See [[Rubeus]].

## See Also

- [[Network Scanning]] for the scan that populates this
- [[SNMP LDAP RPC]] for those three protocols in depth
- [[Master Routing Table]] for the finding → action index
- Source: [[OSCP Field Manual]] section 2.1
