---
status: working
tags:
  - stuffs
  - methodology
  - process
  - mindset
---
> *Operating Model & Mindset:* The process layer that forces breadth-before-depth and guarantees I circle back. Technique lives in the individual discipline folders. This note is how I drive an engagement so the outcome is coverage and not a single lucky path.

# Why I Made This

The primary reason is that having a methodology truly does ensure the best coverage across all engagements. It prevents me from rabbit-holeing on interesting vectors right as I see them, or later when I still haven't fully enumerated the platform.

The other reason is to attempt to undo what studying for the OSCP exam has done to me. That exam rewards cutting off vectors early when they don't look as promising for the sake of time. This is where the opposite end of my issues exist. Where I see things that seem unlikely during an engagement and just ignore them for most of the engagement.

# The Operating Model

**1. Fire the slow scans first, then poke**
The moment I have scope, launch everything long-running in the background so it works while I explore by hand:
- `nmap` full TCP (`-p-`) + top UDP, then `-sC -sV` on discovered ports (see [[01B - Network Scanning]])
- Content discovery (`ffuf`/`feroxbuster`/Burp Intruder against each web root)
- `nuclei` for a known-CVE and misconfig baseline
- Subdomain/vhost discovery if in scope

**2. The Lead Book**
This will serve as my anti-rabbit-hole constraint. When I see something interesting, I do **not** dive in right away. I write one line and keep sweeping:
```
[ ] P2 - JWT on /api/auth uses HS256, check weak secret
[ ] P1 - /admin returns 403 not 401, authz may be path-based
[ ] P3 - url= param on /redirect → open redirect/SSRF
[x] P1 - IDOR confirmed on /api/orders/{id}
```
> [!NOTE] Priority Tags
> Above, I utilize a `P#` format to indicate the priority investigating a particular finding is.

**3. Timebox every dive**
When exploitation does start, set a time limit that reflects the engagement depth, complexity, and time constraints (e.g., 45 min). No progress in the window means the lead goes back in the boox with a note and I move to the next one.

**4. Two-Pass Model**
Structure every target as two explicit passes. This is the fix for "I forgot to check what is hidden."
- **Pass 1 (Visible Surface):** Everything the app shows. Crawl all linked UI, map every endpoint the frontend calls, enumerate obvious directories.
- **Pass 2 (Hidden Surface):** Self-explanatory. Hunt down the non-obvious. Use a variety of wordlists across different surfaces. The web-specific version lives in [[02AA - Methodology]].

# See Also

- [[01AA - Methodology]], [[02AA - Methodology]], [[03AA - Methodology]], [[04AA - Methodology]]
- [[00C - Master Routing Table]] for finding-to-action once a lead is live
