---
status: working
tags:
  - credentials
  - wordlists
  - resources
---
> *Wordlists and Custom List Generation:* The on-disk lists you reach for and how to build a target-specific one. On Kali most live under `/usr/share/`.

## On-Disk Wordlists

```
# Passwords
/usr/share/wordlists/rockyou.txt                     # THE list, gunzip rockyou.txt.gz first
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
/usr/share/seclists/Passwords/Common-Credentials/best110.txt          # quick spray
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt
# Web content / directories / files
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # default dir list
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt        # files not dirs
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt                   # quick pass
/usr/share/wordlists/dirb/big.txt
# Subdomains / DNS / vhosts
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
# Usernames
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt
# Parameters / injection payloads
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
# SNMP community strings
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

> [!note]
> If SecLists is not installed: `sudo apt install seclists` or clone `github.com/danielmiessler/SecLists`. rockyou is your default cracking list, the top-1000 / best110 lists are for fast spraying where you do not want to burn time.

## Custom Wordlist Generation

When a target-specific vocabulary would beat a generic list (company terms, product names on the site).

```bash
cewl -d 2 -m 5 -w custom.txt http://$IP                       # scrape site words (2 links deep, >=5 chars)
cewl http://$IP -d 4 --with-numbers -w cewlpasswords.txt      # include numbers
```

> [!note]
> Feed the result to hashcat with a rule (`-r best64.rule`) to cover capitalization and `!`/number suffixes ([[Hash Cracking]]). For AD username formats, generate permutations (`first`, `flast`, `f.last`, `firstl`) from real names, `username-anarchy` automates it.

## See Also

- [[Hash Cracking]] and [[Brute Forcing and Spraying]] which consume these
- [[Web Enumeration]] for the discovery lists
- [[Reference Sites]] for SecLists and PayloadsAllTheThings
- Source: [[OSCP Field Manual]] sections 9.0, 9.1
