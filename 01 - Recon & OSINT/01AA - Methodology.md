---
status: working
tags:
  - recon
  - methodology
  - process
---
> *Recon Methodology:* How to open an engagement so the long-running work is already finishing while I explore by hand. This is the operating model from [[00G - Methodology & Mindset]] applied to the recon phase. The technique detail lives in [[01B - Network Scanning]] and [[01C - CVE Triage]].

# Instant Fire-and-Forget

The instant the scope is fully established and the testing window begins, start everything slow in the background. **Do this before any manual poking.**

```
[ ] nmap -p- -sC -sV against each host, plus top-UDP
[ ] ffuf / feroxbuster content discovery per web root
[ ] nuclei baseline (CVEs, misconfig, exposures)
[ ] subdomain/vhost enum if in scope
```

> [!TIP] Wordlist Quality
> Use good lists, not `common.txt`. SecLists plus the Assetnote wordlists are the quality tier. A recursive `feroxbuster` with a strong list finds what a lazy list never will.

# Fingerprint, Then Triage

Every version string is a lead for [[01C - CVE Triage]]. Pull them deliberately:
- `whatweb`/Wappalyzer, grab `Server`, `X-Powered-By`, framework headers
- Feed each version into a known-CVE lookup ([[01C - CVE Triage]], [[00D - Famous Exploits Lookup]])
- Identify the auth mechanism early (session cookie/JWT/SSO/API key), it shapes every later authz test

# Everything Becomes a Lead, Not a Detour

Findings from the sweep do not get chased here. They go into the Lead Book ([[00G - Methodology & Mindset]]) with a priority tag, and the dive happens only once the breadth baseline is done.

> [!NOTE] Tooling Currency
> The **ProjectDiscovery** suite (`subfinder`, `httpx`, `katana`, `nuclei`, `naabu`) is the modern recon backbone and worth standardizing on. `ffuf`/`feroxbuster` remain the fuzzing workhorses.

# See Also

- [[00G - Methodology & Mindset]] for the operating model it applies
- [[01B - Network Scanning]] for the scan commands
- [[01C - CVE Triage]] for turning a version into an exploit
- [[02AA - Methodology]]
