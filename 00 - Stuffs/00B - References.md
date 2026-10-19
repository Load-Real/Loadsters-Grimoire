---
status: working
tags:
  - stuffs
  - reference
---
# Useful Dumps to Steal From

- The Hacker Recipes: https://www.thehacker.recipes/
	- Strong on AD/internal, but has other stuff
- HackTricks: https://hacktricks.wiki/
	- Pretty much everything, but not as much depth as others
- PayloadsAllTheThings: https://github.com/swisskyrepo/payloadsallthethings
	- Has a folder-per-vuln class, with payload files underneath
- InternalAllTheThings: https://github.com/swisskyrepo/InternalAllTheThings
	- Same as above, but with specialized focus on AD, cloud, and internal
- Pentest Book (six2dez): https://www.pentest-book.com/
	- Recon-heavy gitbook
- Red Team Notes (ired.team): https://www.ired.team/
	- Organized red tream technique notes in a MITRE-like way; useful for tagging notes to ATT&CK

# Tooling Links

- **GTFOBins** — Linux SUID/sudo/capability escapes: https://gtfobins.github.io
- **LOLBAS** — Windows living-off-the-land binaries: https://lolbas-project.github.io
- **WADComs** — Windows/AD command lookup by context: https://wadcoms.github.io
- **revshells.com** — reverse shell generator/encoder: https://www.revshells.com
- **HackTricks** — methodology per port/service/technique: https://book.hacktricks.xyz
- **PEASS-ng (linpeas/winpeas)** — https://github.com/carlospolop/PEASS-ng
- **Impacket** — https://github.com/fortra/impacket
- **BloodHound docs / Cypher** — https://bloodhound.readthedocs.io (+ search "hausec cypher cheatsheet")
- **hashcat example hashes** (mode lookup) — https://hashcat.net/wiki/doku.php?id=example_hashes
- **Report templates** — whoisflynn: https://github.com/whoisflynn/OSCP-Exam-Report-Template ; CPTC: https://github.com/nationalcptc/report_examples

## Methodology & Checklists

- **OWASP WSTG (Web Security Testing Guide)** — the category-by-category web checklist, the backbone for 02: [https://owasp.org/www-project-web-security-testing-guide/](https://owasp.org/www-project-web-security-testing-guide/)
- **PortSwigger Web Security Academy** — technique + free labs: [https://portswigger.net/web-security](https://portswigger.net/web-security)
- **OWASP API Security Top 10** — the API mental model (BOLA, broken auth, BOPLA): [https://owasp.org/www-project-api-security/](https://owasp.org/www-project-api-security/)

## Cloud

- **HackTricks Cloud** — per-provider methodology: [https://cloud.hacktricks.xyz](https://cloud.hacktricks.xyz)
- **ScoutSuite** (multi-cloud posture): [https://github.com/nccgroup/ScoutSuite](https://github.com/nccgroup/ScoutSuite)
- **Prowler** (posture, AWS/GCP/Azure): [https://github.com/prowler-cloud/prowler](https://github.com/prowler-cloud/prowler)
- **GCPBucketBrute**: [https://github.com/RhinoSecurityLabs/GCPBucketBrute](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- **GCPGoat** (practice): [https://github.com/ine-labs/GCPGoat](https://github.com/ine-labs/GCPGoat)
- **ROADtools / roadrecon** (Entra enum): [https://github.com/dirkjanm/ROADtools](https://github.com/dirkjanm/ROADtools)
- **AzureHound** (Entra attack paths → BloodHound): [https://github.com/BloodHoundAD/AzureHound](https://github.com/BloodHoundAD/AzureHound)
- **MicroBurst / PowerZure / AADInternals** (Azure/Entra tradecraft)
- **o365spray** (Entra user enum + spray): [https://github.com/0xZDH/o365spray](https://github.com/0xZDH/o365spray)

## AD / Internal (current tooling)

- **BloodHound Community Edition** + `SharpHound` (legacy neo4j build is dead): [https://github.com/SpecterOps/BloodHound](https://github.com/SpecterOps/BloodHound)
- **NetExec (`nxc`)** — maintained successor to CrackMapExec: [https://github.com/Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)
- **Certipy** — ADCS ESC1–ESC16: [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)
- **Kerbrute**: [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)
- **PingCastle** (AD config health): [https://www.pingcastle.com](https://www.pingcastle.com)
- **adPEAS**: [https://github.com/61106960/adPEAS](https://github.com/61106960/adPEAS)

## On-host Privesc

- **Seatbelt / PowerUp / PrivescCheck** (Windows), **WES-NG** (missing-patch → CVE)
- **pspy** (watch cron/processes, no root): [https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)

## Pivoting

- **ligolo-ng** (tun-based, current favorite): [https://github.com/nicocha30/ligolo-ng](https://github.com/nicocha30/ligolo-ng)
- `chisel`, `sshuttle`, `proxychains` as alternates

## Recon / Content Discovery

- **ProjectDiscovery suite** (`subfinder`, `httpx`, `katana`, `nuclei`, `naabu`): [https://github.com/projectdiscovery](https://github.com/projectdiscovery)
- **ffuf** / **feroxbuster** for fuzzing
- **SecLists** + **Assetnote wordlists** (quality matters more than tool choice)
- **kiterunner** (API route brute), **arjun** (param discovery), **LinkFinder** / **hakrawler** (JS endpoint parsing)
- **trufflehog** / **gitleaks** (secrets in JS/code), **graphw00f** / **InQL** / **clairvoyance** (GraphQL), **jwt_tool**, **ysoserial** (deserialization)
# See Also

Add to meeee