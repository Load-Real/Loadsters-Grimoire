---
status: working
tags:
  - recon
  - scanning
  - enumeration
---

> *Host and Service Discovery:* How to go from an IP or a subnet to a mapped set of open ports and versioned services. This is the first note you open on any engagement or box. The output of everything here is either a version string (feeds [[CVE Triage]]) or an interface to test creds against (feeds [[Brute Forcing and Spraying]]). Most failed boxes are failed enumeration, not failed exploitation.

Set the target once so every command is copy-paste safe:
```bash
export IP=10.10.10.10
mkdir -p nmap
```

## Staged TCP Scan (never one giant blocking scan)

**The pattern is two passes:** (1) A fast pass to learn which ports are open at all, (2) then a slow, detailed pass against only those ports. Running version and script detection against all 65535 ports wastes many minutes. Splitting it keeps the slow pass small.

Pass one, fast full-port sweep:
```bash
nmap -p- --min-rate 10000 -T4 $IP -oN nmap/allports.txt
```

Pass two, deep scan of only the open ports:
```bash
ports=$(grep -oP '^\d+(?=/tcp\s+open)' nmap/allports.txt | paste -sd,)
nmap -p$ports -sC -sV -O -oN nmap/deep.txt $IP
```

> [!note]
> `-p-` scans all 65535 ports. Never settle for the default top-1000, intended paths hide on high ports. `--min-rate 10000` sets a packet-per-second floor, aggressive but fine on a lab LAN and something you would throttle on a real engagement. `-sV` is the single most important flag because the version string is what you feed to searchsploit. `-sC` runs the safe default NSE scripts (banners, titles, anon-access checks). `-O` is OS detection and needs root. `-A` bundles `-sC -sV -O --traceroute` if you prefer one flag. Use `-oA nmap/all` instead of `-oN` when you want the `.gnmap` and `.xml` for later parsing or reporting.

## NSE Vuln Scan (the allowed substitute for banned scanners)

Automated vuln scanners like Nessus and OpenVAS are off-limits on the OSCP, but Nmap's `vuln` script category is allowed and catches the classic exam CVEs.
```bash
nmap -p$ports --script vuln -oN nmap/vuln.txt $IP
```

> [!note]
> Flags well-known issues like `smb-vuln-ms17-010` (EternalBlue) and various `http-*` checks. Narrow it when you know the surface: `--script "smb-vuln-*"` or `--script "http-enum"`.

## UDP Scan (the step everyone forgets)

UDP services never appear in a TCP scan, and forgetting them costs hours. Scan the top UDP ports only, full UDP is far too slow to be worth it.

```bash
sudo nmap -sU --top-ports 100 -T4 -oN nmap/udp.txt $IP
```

> [!note]
> Watch for 53 DNS, 69 TFTP, 123 NTP, 161 SNMP, 500 IKE, 2049 NFS. Routing: 161 open goes to [[SNMP LDAP RPC]], 2049 goes to [[Service Enumeration]] (NFS), 69 is `tftp get/put`.

## Internal / Post-pivot Discovery

Once you are inside a new network through a pivot, find live hosts before scanning each one.
```bash
nmap -sn 10.10.10.0/24 -oN nmap/hostsweep.txt          # ping sweep, no port scan
fping -asgq 172.16.5.0/23                                # fast alternative sweep
```

> [!note]
> Over a SOCKS proxy you must use TCP-connect scans (`nmap -sT -Pn`), because SYN scans do not traverse a proxy. See [[Pivoting and Tunneling]]. A real route (ligolo, sshuttle) lets every scan type work unmodified.

## Faster Front-ends

```bash
# rustscan: fast port discovery, hands the open ports straight to nmap
rustscan -a $IP --range 1-65535 --ulimit 5000 -- -sC -sV -oN nmap/rustscan.txt
# masscan: raw discovery across large ranges, no version detection (nmap the hits after)
sudo masscan -p1-65535 $IP --rate=1000 -oL nmap/masscan.txt
sudo masscan 10.10.10.0/24 -p80,443,445 --rate=10000    # spray a few ports across a subnet
```

> [!note]
> `--ulimit 5000` is the fix for "why is rustscan slow." Everything after `--` passes through to nmap. Use rustscan/masscan for discovery speed and nmap `-sCV` / `--script vuln` for depth. For a single box, `nmap -p- --min-rate 10000` alone is usually enough and keeps you in one tool.

## Handy Flags Worth Remembering

```bash
-Pn            # skip host discovery when a box you KNOW is up shows "host down" (ICMP filtered)
-sT            # TCP connect scan, REQUIRED through proxychains
--top-ports N  # scan the N most common ports (fast triage)
-sV --version-intensity 9   # squeeze harder on stubborn version detection
--open         # only show open ports
-6             # scan an IPv6 target
-A -iL hosts.txt -oN out    # full detail across a host list (internal sweep)
```

## See Also

- [[Service Enumeration]] for what to do with each open port
- [[Web Enumeration]] when any web port is open
- [[CVE Triage]] once you have a version string
- Source: [[OSCP Field Manual]] section 2.0
