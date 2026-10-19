---
status: working
tags:
  - active-directory
  - network
  - methodology
  - process
---
> *Network & AD Methodology:* How to run an internal/AD engagement for coverage, and the current tooling that has moved on since the OSCP-era notes. The operating model is [[00G - Methodology & Mindset]], the step-by-step technique is in the `03x` notes.

# The AD Coverage Model

BloodHound CE is the come-back-and-check everything tool for AD. It *is* the breadth mechanism from [[00G - Methodology & Mindset]], applied to a domain. Collect early, mark owned nodes as you go, and let path queries tell you what you have not yet used.

The chain, each step linking to its technique note:
1. **Enumerate** the domain ([[03D - AD Enumeration]]) and collect with SharpHound/BloodHound CE.
2. **Roast and spray** ([[03E - AD Attacks]]): AS-REP (no creds needed), Kerberoast, password spray after checking lockout.
3. **Abuse ACL edges** ([[03E - AD Attacks]]) that BloodHound draws from owned nodes.
4. **Move laterally** ([[03F - AD Lateral Movements & Persistence]]) with the hash or ticket.
5. **Dump and reach the DC** ([[03F - AD Lateral Movements & Persistence]]): secretsdump, DCSync, shadow-copy NTDS.
6. **Pivot** to the next subnet ([[03G - Pivoting & Tunneling]]) and recurse.

> [!WARNING] Tooling Currency
> The OSCP course makes use of older tools. This is the legacy-to-modern mapping:
> - **`NetExec` (`nxc`)** is the maintained successor to **CrackMapExec**, which is deprecated. New notes should prefer `nxc`.
> - **BloodHound Community Edition** + SharpHound. The legacy neo4j BloodHound is dead, use CE.
> - **`Certipy` for ACDS** is a huge modern surface people miss, **ESC1 through ESC16**. Worth its own technique note under 03 later.
> - Supporting tools: The **Impacket** suite, `Rubeus` ([[11B - Rubeus]]), `bloodyAD` ([[11C - BloodyAD]]), `Kerbrute`, `ldapdomaindump`, `PingCastle`, and `adPEAS` for a fast config-health sweep.

# On-Host Enumeration

First move on any shell is **PEASS-ng** (`linpeas`/`winpeas`). Then the targeted tools:
- **Linux:** `pspy` (watch cron/processes without root), **GTFOBins** (can I abuse this binary) → [[05B - Linux Privilege Escalation]]
- **Windows:** `Seatbelt`, `PowerUp`, `PrivescCheck`, **LOLBAS** (the GTFOBins equivalent), `WES-NG` for missing-patch → CVE → [[05C - Windows Privilege Escalation]]

# Pivoting

`ligolo-ng` is the current favorite (clean tun-based routing, every tool and scan type works unmodified), with `chisel`, `sshuttle` and `proxychains` as alternates. Full detail in [[03G - Pivoting & Tunneling]].

# See Also

- [[00G - Methodology & Mindset]] for the operating model
- [[03D - AD Enumeration]], [[03E - AD Attacks]], [[03F - AD Lateral Movements & Persistence]], [[03G - Pivoting & Tunneling]], [[03B - Service Enumeration (Ports)]], [[03C - SNMP, LDAP, RPC]]
- [[05B - Linux Privilege Escalation]] , [[05C - Windows Privilege Escalation]] for on-host work
- [[11B - Rubeus]], [[11C - BloodyAD]] for the two this tool leans on
- [[00B - References]] for the AD/internal toolkit
