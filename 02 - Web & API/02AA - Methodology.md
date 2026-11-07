---
status: working
tags:
  - web
  - api
  - methodology
  - process
---
> *Web & API Methodology:* The two-pass sweep, the hidden-surface pass that keeps getting skipped, and the signal catalog that stops the brain from discarding findings as benign. This document is [[00G - Methodology & Mindset]] applied to web and API targets. Individual exploitation technique lives in the `02x` notes.

# Pass 1 - The Visible Surface

Everything the app shows you:
- Crawl all linked UI (`katana` + manual)
- Capture every API call the frontend makes (Burp proxy)
- Enumerate roles and user types, map what each can reach

# Pass 2 - The Hidden Surface

The pass that often gets skipped. Run it every time, as a literal checklist.
- **Hidden Dirs/Files:** Content discovery with a *good* wordlist (Assetnote + raft, not `common.txt`), recursive, stack-matched extensions. Backups (`.bak`, `~`, `.old`, `.swp`), VCS leaks (`/.git/`, `/.svn/`), `/.env`, `.DS_Store`.
- **Hidden API Routes:** Parse every JS bundle, the frontend knows endpoints the UI never shows. `katana`/`hakrawler` to crawl, then `LinkFinder` or grep budnles for path strings. Pull **source maps** (`.js.map`) when present, they reconstruct original source and routes. `kiterunner` to brute API paths against common schemas.
- **Hidden UI/Slate:** Elements gated behind role checks or feature flags. Disabled buttons, `display:none`, commented markup, client-side role logic (`if (user.isAdmin)`) the server may not enforce. This is where client-side authz bypass lives.
- **Hidden Params:** `arjun` or Burp `param-miner`. Debug flags (`?debug=true`, `?test=1`), format toggles (`?format=xml`).
- **Old API Versions:** See `/v2/`, try `/v1/` and `/v0/`. Older versions are frequently less protected and still live.
- **Different Verbs:** An endpoint that 405s on GET may accept PUT/DELETE/PATCH with no authz.

# Signal Catalog

The direct fix for discarding things as benign. When you see the left, you run the right, every time, no judgement about whether it "seems useful".

| Signal                                                                 | Reflex                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **JWT**                                                                | 1) Decode header + payload.<br>2) Check `alg`: try `none`, try HS256/RS256 confusion (sign with the public key as HMAC secret).<br>3) Crack weak HS256 (`hashcat -m 16500`).<br>4) Check `kid` for traversal/SQLi/cmdi, `jku`/`x5u` for SSRF to an attacker key.<br>5) Strip signature, check expiry + `alg` validation.<br><br>Tool: `jwt_tool` |
| **Base64/Hex Blob** in cookie or param                                 | 1) Decode it.<br>2) Starts `rO0` (Java), `AAEAAD` (.NET), `a:1:{`/`O:` (PHP) → serialized object → deserialization candidate.<br><br>Tool: `ysoserial`                                                                                                                                                                                           |
| **Numeric/Sequential ID** e.g., (`api/users/1043`)                     | IDOR/BOLA. The #1 API bug class, easiest to miss because it looks mundane.<br><br>1) Increment, decrement, try other users' IDs with you token.<br>                                                                                                                                                                                              |
| **Reflected Input**                                                    | Do not stop at XSS.<br><br>Test SSTI `{{7*7}}`, `${7*7}`, SQLi, cmd injection.<br><br>Reflection means input reaches a sink.                                                                                                                                                                                                                     |
| **`url=`, `next=`, `redirect=`, `returnTo=`, `callback=`**             | Open redirect, then escalate to SSRF. Server-side fetch → point at `169.254.169.254` (cloud metadata) or internal hosts.                                                                                                                                                                                                                         |
| **Verbose Error/Stack Trace**                                          | Tech + version fingerprint, internal path, disclosure, sometimes creds. Feed version to CVE lookup.                                                                                                                                                                                                                                              |
| **`Server`/`X-Powered-By`/Framework Headers**                          | Version → CVE search. Trivial, routinely skipped.                                                                                                                                                                                                                                                                                                |
| **File Upload**                                                        | Extension bypass (double ext, null byte, case, `.phtml`), `Content-Type` spoof, magic bytes, path traversal in filename, and where it lands (web-reachable = potential RCE). See [[02E - File Upload to Webshell]]                                                                                                                               |
| **GraphQL Endpoint**                                                   | Introspection query first. Disabled → field suggestion/`clairvoyance` to rebuild schema. Check batching + per field authz.<br><br>Tools: `graphw00f`, `InQL`                                                                                                                                                                                     |
| **CORS (`ACAO`) Reflecting Your `Origin` + `Allow-Credentials: true`** | Cross-origin data theft.                                                                                                                                                                                                                                                                                                                         |
| **Cookie Missing `HTTPOnly`/`Secure`/`SameSite`**                      | Note it, it changes exploitability of any XSS found.                                                                                                                                                                                                                                                                                             |
| **Secrets in JS Bundles**                                              | `trufflehog`/`gitleaks` on the JS + manual grep for `apiKey`, `AKIA`, `AIza`, bearer tokens.<br><br>Devs leak keys in the frontend constantly.                                                                                                                                                                                                   |
| **S3/GCS/Azure Blob URL in a Response**                                | Bucket/container enum for pulic read/write/listing. Pivot to [[04AA - Methodology]]                                                                                                                                                                                                                                                              |
| **WebSocket**                                                          | AuthZ is often checked on connect, but not per-message. Replay and tamper individual messages.                                                                                                                                                                                                                                                   |
# Highest-Value Bug Classes

This is a brief list of where to best spend your testing timeboxes during each engagement.
- **IDOR/BOLA sweep** on every ID-bearing endpoint
- **Vertical privesc**, low-priv token reaching an admin function
- **Session/JWT analysis**, run the JWT reflex above
- **Password reset/registration/MFA logic flaws**
- **Input per sink**:
	- [[02G - SQL Injection]]
	- [[02D - Command Injection]]
	- [[02C - LFI, RFI & Path Traversal]]
	- [[02H - Other Injection Classes]] (SSTI/XXE)
	- [[02E - File Upload to Webshell]]
	- SSRF
	- Deserialization

# Worked Checklist

```
RECON (fire and forget → 01AA)
[ ] nmap -p- -sCV, top-UDP
[ ] ffuf/ferox per web root (recursive, stack-matched exts)
[ ] nuclei baseline
[ ] Subdomain/vhost if in scope

FINGERPRINT
[ ] whatweb/wappalyzer, grab Server/X-Powered-By/framework
[ ] Version → CVE lookup
[ ] Identify auth mechanism (cookie/JWT/SSO/API key)

MAP (visible surface)
[ ] Crawl all linked UI (katana + manual)
[ ] capture every frontend API call (Burp)
[ ] enumerate roles, map what each can reach

HIDDEN SURFACE (Pass 2 - mandatory)
[ ] Parse JS bundles for endpoints (LinkFinder), pull source maps
[ ] kiterunner brute API routes
[ ] arjun/param-miner for hidden params
[ ] backup/VCS/.env/.git, old API versions, alternate verbs
[ ] Hidden/disabled UI + client-side authz checks

AUTH & AUTHZ (highest value)
[ ] IDOR/BOLA sweep on every ID-bearing endpoint
[ ] Vertical privesc (low-priv → admin function)
[ ] Session/JWT analysis (run the JWT reflex)
[ ] Password reset/registration/MFA logic

INPUT (per sink)
[ ] SQLi, SSTI, cmd injection, XSS, SSRF, XXE, file upload, deserialization

SERVER
[ ] Service enum from nmap (→ 03B), default creds (→ 02F), known CVEs
[ ] If containerized: Metadata endpoints, cloud creds, SSRF to control plane (→ 04AA)
```

> [!NOTE] API Mental Model
> Carry the **OWASP API Security Top 10** (BOLA, broken auth, BOPLA, etc.) as the lens for API targest. `kiterunner` for route discovery, `arjun` for params, Burp/Postman for the manual work.

# See Also
- [[00G - Methodology & Mindset]] for the operating model
- [[02B - Web Enumeration]] for the enumeration commands
- [[02G - SQL Injection]], [[02D - Command Injection]], [[02C - LFI, RFI & Path Traversal]], [[02E - File Upload to Webshell]], [[02F - Admin Panels & Default Creds]], [[02H - Other Injection Classes]] for the exploitation detail
- [[03B - Service Enumeration (Ports)]] for the server underneath, [[04AA - Methodology]]
- [[00B - References]] for OWASP WSTG, PortSwigger, and the API resources
