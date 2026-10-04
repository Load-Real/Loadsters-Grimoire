---
status: working
tags:
  - tooling
  - reference
---
> *Tool Map:* Which tool for which phase, and for the two big target types (Linux vs Windows/AD) what each tool does and when you reach for it. A reference you scan, not read.

## By Phase

| Phase | Tools |
|---|---|
| Port/host scan | `nmap`, `rustscan`, `masscan`, `autorecon` |
| Web enum | `feroxbuster`, `gobuster`, `ffuf`, `whatweb`, `wpscan`, `nikto`, Burp Free |
| SMB/AD enum | `crackmapexec`/`netexec`, `enum4linux-ng`, `smbclient`, `smbmap`, `rpcclient`, `ldapsearch`, `bloodhound-python`, `kerbrute`, `Snaffler`, `adidnsdump` |
| Cred attacks | `hydra`, `hashcat`, `john`, `impacket-GetUserSPNs`, `impacket-GetNPUsers` |
| Exploitation | `searchsploit`, `msfvenom`, `metasploit` (one target), `impacket-*` |
| Linux privesc | `linpeas.sh`, `lse.sh`, `pspy`, `getcap`, GTFOBins |
| Windows privesc | `winPEAS`, `PowerUp.ps1`, `Seatbelt`, `accesschk`, `GodPotato`, `PrintSpoofer`, `JuicyPotatoNG` |
| AD attack | Impacket suite, `Rubeus`, `mimikatz`, `evil-winrm`, `bloodyAD`, `PowerView` |
| Pivoting/transfer | `chisel`, `ligolo-ng`, `proxychains`, `socat`, `sshuttle` |

## Linux Targets: What & When

| Tool | What it does | When |
|---|---|---|
| `hydra` | Online brute/spray of network logins | Login interface + user/pass lists ([[Brute Forcing and Spraying]]) |
| `hashcat` | GPU offline cracking, every mode | Recovered a hash, have a GPU ([[Hash Cracking]]) |
| `john` | CPU cracking + `*2john` converters | No GPU, or odd formats |
| `searchsploit` | Local Exploit-DB search/read | Have a version string ([[CVE Triage]]) |
| `msfvenom` | Payload generator (exam-legal) | Need a reverse/bind ELF or PHP payload ([[msfvenom Payloads]]) |
| `nc`/`socat` | Listeners + port relays | Catch shells, relay a port through a pivot |
| `linpeas.sh` | Automated Linux privesc enum | First thing on any Linux shell ([[Linux Privilege Escalation]]) |
| `pspy` | Watch processes + cron live, no root | Catch a hidden root cron job |
| `getcap` | List file capabilities | Hunt `cap_setuid`/`cap_dac_*` |
| GTFOBins | SUID/sudo/capability escape lookups | Map any allowed binary to a root shell |

## Windows / AD Targets: What & When

| Tool | What it does | When |
|---|---|---|
| `crackmapexec`/`netexec` | SMB/WinRM/MSSQL swiss-army: spray, PtH, dump, enum | Spray across a subnet, test where you are admin |
| `impacket-secretsdump` | Dump SAM/LSA/cached/NTDS hashes | After local/domain admin ([[AD Lateral Movement and Persistence]]) |
| `responder` | LLMNR/NBT-NS poisoning (BANNED on exam) | Lab/practice only ([[Responder and NTLM Relay]]) |
| `impacket-psexec`/`wmiexec`/`smbexec` | Remote exec/shell with creds or hash | Shell on a host you are admin on |
| `evil-winrm` | Interactive WinRM shell, in-memory tool loading | Port 5985 + a Windows credential |
| `winPEAS` | Automated Windows privesc enum | First thing on any Windows shell ([[Windows Privilege Escalation]]) |
| `PowerUp.ps1` | Service/registry misconfig checks | Targeted service/registry privesc |
| `accesschk` | ACL checks, "can I write this?" | Confirm service/binary/file write rights |
| GodPotato / PrintSpoofer / JuicyPotatoNG | Abuse `SeImpersonate` → SYSTEM | `whoami /priv` shows an impersonate token |
| `bloodhound-python`/`SharpHound` | Collect the AD graph | Map the path to DA ([[AD Enumeration]]) |
| `impacket-GetUserSPNs`/`GetNPUsers` | Kerberoast / AS-REP from Kali | Remote roasting ([[AD Attacks]]) |
| `Rubeus` | On-host Kerberos multitool | From a Windows foothold ([[Rubeus]]) |
| `bloodyAD`/`PowerView` | Execute an ACL-edge abuse | Turn a BloodHound edge into action ([[BloodyAD]]) |
| `mimikatz` | Dump LSASS, DCSync, pass-the-ticket | Windows host with admin |

## See Also

- [[Environment Setup]] for installing and configuring these
- [[Reference Sites]] for where to grab them
- Source: [[OSCP Field Manual]] sections 10.0 to 10.0.2
