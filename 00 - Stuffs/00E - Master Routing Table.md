---
status: solid
tags:
  - stuffs
  - routing
  - index
---
> *Master Routing Table:* Finding → next action. A cross-discipline index you scan when you have observed something and want the fastest pointer to the note that handles it. This is the navigation hub that keeps you out of "scan the whole vault" mode.

| You observed… | Do this / go to |
|---|---|
| Port 21 FTP | anon login → [[Finding Credentials]] loot; FTP==webroot → [[File Upload to Webshell]] |
| Port 22 SSH + key/creds | [[Service Enumeration]]; `chmod 600` key; `ssh2john` if passphrased |
| Port 25 SMTP | `smtp-user-enum` → userlist → [[Brute Forcing and Spraying]] |
| Port 53 / a domain | `dig axfr`; names to /etc/hosts |
| Port 80/443/8080 | [[Web Enumeration]] → the injection notes |
| **Port 88 Kerberos** | **it's a DC → [[AD Enumeration]]** |
| Port 139/445 SMB | null shares/users → [[Brute Forcing and Spraying]] / [[Finding Credentials]]; ms17-010 → [[Famous Exploits Lookup]] |
| Port 161/udp SNMP | community strings → snmpwalk → [[SNMP LDAP RPC]] |
| Port 389 LDAP | anon bind dump → [[SNMP LDAP RPC]] → [[AD Enumeration]] |
| Port 1433 MSSQL | `sa` → `xp_cmdshell` RCE → [[Service Enumeration]] |
| Port 2049 NFS | `showmount -e`; no_root_squash → [[Linux Privilege Escalation]] |
| Port 3306 MySQL | default/blank root → [[Service Enumeration]] |
| Port 3389 RDP + creds | `xfreerdp` GUI |
| Port 5432 PostgreSQL | `postgres:postgres`; superuser → RCE |
| Port 5985 WinRM + creds/hash | `evil-winrm` → [[AD Lateral Movement and Persistence]] |
| **A login page** | default creds → manual SQLi bypass [[SQL Injection]] → hydra [[Brute Forcing and Spraying]] |
| **An upload form** | webshell upload → [[File Upload to Webshell]] |
| `?param=file` / `include(` | LFI/RFI → RCE → [[LFI RFI and Path Traversal]] |
| **Any credential, anywhere** | **SPRAY everywhere → [[Brute Forcing and Spraying]]** |
| An old service version | `searchsploit` → [[CVE Triage]] / [[Famous Exploits Lookup]] |
| `id_rsa`/`.kdbx`/`.env` | crack/use → [[Hash Cracking]] / privesc |
| Low-priv shell (Linux) | [[Linux Privilege Escalation]] |
| Low-priv shell (Windows) | **`whoami /priv` first** → [[Windows Privilege Escalation]] |
| `SeImpersonate`/`SeAssignPrimaryToken` | **Potato → SYSTEM → [[Windows Privilege Escalation]]** |
| Box is domain-joined | [[AD Enumeration]] |
| Owned a box, need next subnet | [[Pivoting and Tunneling]] |

## See Also

- [[README - Using the Grimoire]] for how the vault is organized
- [[Service Enumeration]] for the per-port detail behind these pointers
- Source: [[OSCP Field Manual]] section 2.5 (master routing table)
