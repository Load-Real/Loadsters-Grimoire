---
status: working
tags:
  - recon
  - cve
  - searchsploit
  - exploitdb
  - exploitation
---
> *CVE Triage:* Turning "I know the service version" into "I have a working exploit." Reach for this the moment enumeration gives you any version string. This is the bridge between [[Network Scanning]] / [[Service Enumeration]] and actually landing a foothold.

## The `searchsploit` Workflow

```bash
searchsploit <product> <version>     # search the local Exploit-DB mirror
searchsploit -m <id>                 # copy an exploit into the working dir
searchsploit -x <id>                 # read it before running it
```

> [!warning]
> Always read a public exploit before running it. Fix the `RHOST`/`LHOST`/paths/offsets, understand what it does, and confirm it will not trash the box. Confirm the version precisely, a near-miss version often is not vulnerable, and chasing a patched CVE is a classic rabbit hole. Kernel exploits are a last resort ([[Linux Privilege Escalation]], [[Windows Privilege Escalation]]) because they can crash the box.

## Windows: version to missing-patch mapping

`systeminfo` gives the OS build and hotfix list. Feed it to WES-NG (`windows-exploit-suggester-ng`) to map missing patches to kernel and privilege exploits. This is a last-resort privesc path, configuration bugs come first.

## See Also

- [[Famous Exploits Lookup]] for the curated table of classics you actually reach for, by target
- [[Network Scanning]] and [[Service Enumeration]] for the version strings
- [[Reference Sites]] for Exploit-DB, hashcat modes, and PoC repos
- Source: [[OSCP Field Manual]] sections 3.5 and 4.2
