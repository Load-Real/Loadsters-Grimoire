---
status: solid
tags:
  - stuffs
  - routing
  - index
  - network
  - active-directory
---
> *Master Routing Table:* Finding → next action. A cross-discipline index you scan when you have observed something and want the fastest pointer to the note that handles it. This is the navigation hub that keeps you out of "scan the whole vault" mode.

| You observed…                          | Do this / go to                                                                                                                     |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Port 21 FTP                            | anon login → [[05D - Finding Credentials]] loot; FTP==webroot → [[02E - File Upload to Webshell]]                                   |
| Port 22 SSH + key/creds                | [[03B - Service Enumeration (Ports)]]; `chmod 600` key; `ssh2john` if passphrased                                                   |
| Port 25 SMTP                           | `smtp-user-enum` → userlist → [[09B - Brute Forcing & Spraying]]                                                                    |
| Port 53 / a domain                     | `dig axfr`; names to /etc/hosts                                                                                                     |
| Port 80/443/8080                       | [[02B - Web Enumeration]] → the injection notes                                                                                     |
| **Port 88 Kerberos**                   | **it's a DC → [[03D - AD Enumeration]]**                                                                                            |
| Port 139/445 SMB                       | null shares/users → [[09B - Brute Forcing & Spraying]] / [[05D - Finding Credentials]]; ms17-010 → [[00D - Famous Exploits Lookup]] |
| Port 161/udp SNMP                      | community strings → snmpwalk → [[03C - SNMP, LDAP, RPC]]                                                                            |
| Port 389 LDAP                          | anon bind dump → [[03C - SNMP, LDAP, RPC]] → [[03D - AD Enumeration]]                                                               |
| Port 1433 MSSQL                        | `sa` → `xp_cmdshell` RCE → [[03B - Service Enumeration (Ports)]]                                                                    |
| Port 2049 NFS                          | `showmount -e`; no_root_squash → [[05B - Linux Privilege Escalation]]                                                               |
| Port 3306 MySQL                        | default/blank root → [[03B - Service Enumeration (Ports)]]                                                                          |
| Port 3389 RDP + creds                  | `xfreerdp` GUI                                                                                                                      |
| Port 5432 PostgreSQL                   | `postgres:postgres`; superuser → RCE                                                                                                |
| Port 5985 WinRM + creds/hash           | `evil-winrm` → [[03F - AD Lateral Movements & Persistence]]                                                                         |
| **A login page**                       | default creds → manual SQLi bypass [[02G - SQL Injection]] → hydra [[09B - Brute Forcing & Spraying]]                               |
| **An upload form**                     | webshell upload → [[02E - File Upload to Webshell]]                                                                                 |
| `?param=file` / `include(`             | LFI/RFI → RCE → [[02C - LFI, RFI & Path Traversal]]                                                                                 |
| **Any credential, anywhere**           | **SPRAY everywhere → [[09B - Brute Forcing & Spraying]]**                                                                           |
| An old service version                 | `searchsploit` → [[01C - CVE Triage]] / [[00D - Famous Exploits Lookup]]                                                            |
| `id_rsa`/`.kdbx`/`.env`                | crack/use → [[09C - Hash Cracking]] / privesc                                                                                       |
| Low-priv shell (Linux)                 | [[05B - Linux Privilege Escalation]]                                                                                                |
| Low-priv shell (Windows)               | **`whoami /priv` first** → [[05C - Windows Privilege Escalation]]                                                                   |
| `SeImpersonate`/`SeAssignPrimaryToken` | **Potato → SYSTEM → [[05C - Windows Privilege Escalation]]**                                                                        |
| Box is domain-joined                   | [[03D - AD Enumeration]]                                                                                                            |
| Owned a box, need next subnet          | [[03G - Pivoting & Tunneling]]                                                                                                      |

## See Also

- [[03B - Service Enumeration (Ports)]] for the per-port detail behind these pointers
- **Source:** *OSCP Field Manual* section 2.5 (master routing table)
