---
status: working
tags:
  - recon
  - web
  - enumeration
---
> *Web Attack-surface Enumeration:* Fingerprint the stack, then discover hidden content, vhosts, parameters, and CMS weaknesses. Reach for this the moment any HTTP/S port is open. Web is the most common foothold surface, so enumerate it thoroughly before calling it a dead end. Everything here feeds [[02G - SQL Injection]], [[02C - LFI, RFI & Path Traversal]], [[02E - File Upload to Webshell]], and the login/upload forms that lead to a shell.

## Fingerprint Before You Fuzz

Know what you are attacking before brute-forcing anything. The tech stack decides which extensions to fuzz and which exploits apply.

```bash
whatweb http://$IP                 # stack, framework, versions
curl -sI http://$IP                # raw headers: server, redirects, cookies
nikto -h http://$IP                # quick vuln/misconfig sweep (exam-legal)
curl -s http://$IP | grep -iE 'generator|powered|version|comment'   # version string hiding in HTML
```

> [!note]
> Also just browse the site and view source. Read `robots.txt` and `sitemap.xml`, and note every login page and upload form, those are your two highest-value targets. On a custom or empty-looking app, inspect-element and view-source every page and watch Burp for POST requests, cookies, and hidden parameters. The manual habit catches what scanners miss.

## Directory and File Discovery

Find hidden paths: admin panels, backups, dev files. Match the `-x` extensions to the stack you fingerprinted or you will miss the interesting files. Run this in the background while you manually browse.

```bash
# feroxbuster, recursive by default
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,html,bak -t 50
# gobuster fallback
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt -t 50
# ffuf with response filtering
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -e .php,.bak,.old,.zip -mc all -fc 404 -t 60
```

> [!note]
> Extension guide: PHP box → `php`; IIS → `aspx,asp`; Tomcat → `jsp`. Always add `bak,old,zip` to catch backup files that leak source or creds. ffuf filtering: `-mc all` matches every status code, then `-fc 404` hides not-founds, and `-fs <size>` hides a repeating default-page size.

## Virtual Host / Subdomain Discovery

When the server responds differently based on the `Host` header, or you have learned a domain name. Different vhosts often serve entirely different apps.

```bash
ffuf -u http://$IP -H "Host: FUZZ.<domain>" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <default_size>
gobuster vhost -u http://<domain> -w <wl> --append-domain
```

> [!note]
> Set `-fs <default_size>` to the byte-size of the default response so only different vhosts show up. Add every discovered vhost to `/etc/hosts` or your tools cannot resolve it.

## Parameter Discovery

Find hidden GET/POST parameters that expose an LFI, SQLi, or command-injection sink the visible UI does not show.

```bash
ffuf -u "http://$IP/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <size>
```

> [!note]
> A newly discovered parameter that changes the response is your lead into the injection notes ([[02C - LFI, RFI & Path Traversal]], [[02G - SQL Injection]], [[02D - Command Injection]]).

## CMS-specific Scanners

When whatweb identifies a known CMS, purpose-built scanners enumerate users, plugins, and vulnerable versions far faster than generic fuzzing.

```bash
wpscan --url http://$IP --enumerate u,ap,at --plugins-detection aggressive
wpscan --url http://$IP --disable-tls-checks --api-token <token>     # fuller vuln data
droopescan scan drupal -u http://$IP
joomscan -u http://$IP
```

> [!note]
> Results route onward: a login page to [[09B - Brute Forcing & Spraying]] and [[02G - SQL Injection]], an upload form to [[02E - File Upload to Webshell]], a version string to [[01C - CVE Triage]]. WordPress: once authenticated, editing a plugin or theme PHP (`akismet.php`, `themes/<t>/404.php`) with a webshell is the classic RCE, see [[02E - File Upload to Webshell]].

## See Also

- [[01B - Network Scanning]] for the port scan that gets you here
- [[02G - SQL Injection]], [[02C - LFI, RFI & Path Traversal]], [[02D - Command Injection]], [[02E - File Upload to Webshell]], [[02H - Other Injection Classes]] for turning surface into a shell
- [[02F - Admin Panels & Default Creds]] for panels you discover
- Source: *OSCP Field Manual* section 2.2
