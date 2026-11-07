---
status: working
tags:
  - recon
  - web
  - enumeration
---
> *Web Attack-surface Enumeration:* Fingerprint the stack, then discover hidden content, vhosts, parameters, and CMS weaknesses. Reach for this the moment any HTTP/S port is open. Web is the most common foothold surface, so enumerate it thoroughly before calling it a dead end. Everything here feeds [[02G - SQL Injection]], [[02C - LFI, RFI & Path Traversal]], [[02E - File Upload to Webshell]], and the login/upload forms that lead to a shell.

# Fingerprint Before You Fuzz

Know what you are attacking before brute-forcing anything. The tech stack decides which extensions to fuzz and which exploits apply.

`whatweb` is a plugin-driven web fingerprinter and **the first thing you should run when you find a web application**. You can run it to get a one-line summary of everything recognized (which is often the web server, the CMS or framework, JavaScript libraries, analytics, and sometimes a version number):
```bash
whatweb http://$IP
```

> [!TIP] Adjusting Aggression
> You can set the aggression level of a `whatweb` operation with the `-a #` switch. 

Below is a raw-headers view (HTTP HEAD request):
```bash
curl -sI http://$IP
```
In the output, look for:
- `Server` and `X-Powered-By` for obvious stack tells (e.g., nginx vs Apache vs IIS)
- `Set-Cookie` for session cookie names that give away details, such as:
	- `PHPSESSID` = PHP
	- `JSESSIONID` = Java stack
	- `ASP.NET_SessionId` = .NET
- `Location` for server redirect details
	- A redirect to a name rather than an IP is a signal that the app is virtual-host routed, meaning you'd need to add that name to your `hosts` file
- Security headers that are absent, such as no HSTS, `Content-Security-Policy`, no `X-Frame-Options` and so on

> [!TIP] Server Handling of HTTP HEAD
> Some servers treat HEAD differently from GET, or refuse it entirely, and you get a thin or empty respond. When that happens, switch to a GET that discards the body but keeps the headers:
> ```sh
> curl -s -o /dev/null -D - http://$IP
> ```

The following command will pull the actual page body and grep it for the version tells that do not show up in headers:
```bash
curl -s http://$IP | grep -iE 'generator|powered|version'
```

> [!HELP] What Each Term Above is Fishing For
> **generator:** A classic keyword that appears in meta generator tags emitted by many CMSes, such as `<meta name="generator" content="WordPress 6.1">`. Hands you a version on a plate.
> **powered:** Footer and header text along the lines of "Powered by" some product.
> **version:** Inline version strings scattered through the markup, often in script or stylesheet paths like `/app.js?ver=2.3.1`, which also leak plugin and theme versions on WordPress.

`nikto` is a signature-based web server scanner. It'll throw thousands of known-bad requests at the target looking for dangerous files, default and backup files, outdated server versions, interesting or dangerous headers, and common misconfigurations:
```bash
nikto -h http://$IP
```

> [!WARNING] False Positives
> While `nikto` is great at surfacing the obvious fast, it also commonly returns false positives. Treat the output as a list of leads to verify, rather than confirmed vulnerabilities.

**Beyond Tools:** The following manual checks are also essential when encountering any web application...
- Browsing the site and viewing the page source
- Reading `robots.txt` and `sitemap.xml`
- Noting every login page, upload form, and other input fields 

> [!TIP] Good Habit: Passing the Page Source Through Burp Suite
> On a custom or empty-looking app, inspect-element and view-source every page and watch Burp for POST requests, cookies, and hidden parameters. The manual habit catches what scanners miss.

# Fuzzing Techniques
## Directory and File Discovery

Especially useful for finding hidden paths, such as admin panels, backups, dev files.

Match the `-x`/`-e` extensions to the stack you fingerprinted or you will miss the interesting files. Run this in the background while you manually browse.

`feroxbuster` is my personal first choice, as it's recursive by default and I prefer the formatting of the output:
```bash
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,html,bak -t 50
```

If necessary, you can fall back to things like `ffuf`, which is useful for its response filtering capabilities (`-mc` /  `fc`):
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -e .php,.bak,.old,.zip -mc all -fc 404 -t 60
```

The classic and simple tool is `gobuster`:
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt -t 50
```

> [!INFO] Extension Guide
> PHP box → `php`
> IIS → `aspx,asp`
> Tomcat → `jsp`.

> [!INFO] ffuf Filtering Guide
> `-mc all` matches every status code, then `-fc 404` hides not-founds, and `-fs <size>` hides a repeating default-page size.

## Virtual Host / Subdomain Discovery

When the server responds differently based on the `Host` header, or you have learned a domain name. Different vhosts often serve entirely different apps.

`ffuf` Variant:
```bash
ffuf -u http://$IP -H "Host: FUZZ.<domain>" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <default_size>
```

> [!TIP] Tip on Using Filter Size
> Set `-fs <default_size>` to the byte-size of the default response so only different vhosts show up. Add every discovered vhost to `/etc/hosts` or your tools cannot resolve it.

`gobuster` Variant:
```bash
gobuster vhost -u http://<domain> -w <wl> --append-domain
```

## Parameter Discovery

Find hidden GET/POST parameters that expose an LFI, SQLi, or command-injection sink the visible UI does not show:
```bash
ffuf -u "http://$IP/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <size>
```

> [!note]
> A newly discovered parameter that changes the response is your lead into the injection notes ([[02C - LFI, RFI & Path Traversal]], [[02G - SQL Injection]], [[02D - Command Injection]]).

# CMS-specific Scanners

When whatweb identifies a known CMS, purpose-built scanners enumerate users, plugins, and vulnerable versions far faster than generic fuzzing.

For WordPress:
```bash
wpscan --url http://$IP --enumerate u,ap,at --plugins-detection aggressive
```

> [!TIP] Scanning WordPress with an API Token
> If you have an **API token**, you can get a more thorough scan:
> ```bash
> wpscan --url http://$IP --disable-tls-checks --api-token <token>
> ```

For Drupal (also supports WordPress, Joomla, SilverStripe, and Moodle):
```bash
droopescan scan drupal -u http://$IP
```

For Joomla (OWASP solution):
```bash
joomscan -u http://$IP
```

# Where to Go Next

From the findings above, there are various things to try next:
- A login page to [[09B - Brute Forcing & Spraying]] and [[02G - SQL Injection]]
- An upload form to [[02E - File Upload to Webshell]]
- A version string to [[01C - CVE Triage]]
- WordPress: once authenticated, editing a plugin or theme PHP (`akismet.php`, `themes/<t>/404.php`) with a webshell is the classic RCE, see [[02E - File Upload to Webshell]]

# See Also

- [[01B - Network Scanning]] for the port scan that gets you here
- [[02G - SQL Injection]], [[02C - LFI, RFI & Path Traversal]], [[02D - Command Injection]], [[02E - File Upload to Webshell]], [[02H - Other Injection Classes]] for turning surface into a shell
- [[02F - Admin Panels & Default Creds]] for panels you discover
- Source: *OSCP Field Manual* section 2.2
