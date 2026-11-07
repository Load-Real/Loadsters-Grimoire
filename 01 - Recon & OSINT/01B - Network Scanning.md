---
status: working
tags:
  - recon
  - scanning
  - enumeration
---
> *Host and Service Discovery:* How to go from an IP or a subnet to a mapped set of open ports and versioned services. This is the first note you open on any engagement or box. The output of everything here is either a version string (feeds [[01C - CVE Triage]]) or an interface to test creds against (feeds [[09B - Brute Forcing & Spraying]]). Most failed boxes are failed enumeration, not failed exploitation.

> [!DANGER] Before Even Throwing a SINGLE Scan
> Network scans can topple sensitive systems. With IoT/OT/SCADA systems, it is **imperative** that, if you perform a network scan, it is **heavily throttled** and **controlled**.

> [!TIP]
> Set the target once so every command is copy-paste safe:
> ```sh
> export IP=10.10.10.10
> mkdir -p nmap
> ```

# Staged TCP Scan

**The pattern is two passes:** (1) A fast pass to learn which ports are open at all, (2) then a slow, detailed pass against only those ports. Running version and script detection against all 65535 ports wastes time. Splitting it keeps the slow pass small.

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

# NSE Vuln Scan

> [!NOTE] OSCP Note
> Automated vuln scanners like Nessus and OpenVAS are off-limits on the OSCP, but Nmap's `vuln` script category is allowed and catches the classic exam CVEs.

```bash
nmap -p$ports --script vuln -oN nmap/vuln.txt $IP
```

> [!note]
> Flags well-known issues like `smb-vuln-ms17-010` (EternalBlue) and various `http-*` checks. Narrow it when you know the surface: `--script "smb-vuln-*"` or `--script "http-enum"`.

# UDP Scan

UDP services never appear in a TCP scan. Scan the top UDP ports only, full UDP is far too slow to be worth it most of the time:
```bash
sudo nmap -sU --top-ports 100 -T4 -oN nmap/udp.txt $IP
```

> [!note]
> Watch for 53 DNS, 69 TFTP, 123 NTP, 161 SNMP, 500 IKE, 2049 NFS. Routing: 161 open goes to [[03C - SNMP, LDAP, RPC]], 2049 goes to [[03B - Service Enumeration (Ports)]] (NFS), 69 is `tftp get/put`.

# Internal / Post-pivot Discovery

Once you are inside a new network through a pivot, find live hosts before scanning each one. There are a couple of options available.

Using `nmap` with host discovery only, with no port scanning (`-sn`), and saving it to an output file:
```bash
nmap -sn 10.10.10.0/24 -oN nmap/hostsweep.txt
```
Use the above when:
- You are on the local segment and want authoritative results (ARP). This is the default choice right after landing on a box that is directly on the subnet.
- You want discovery that survives ICMP filtering, because the TCP probes pick up hosts that block `ping`.

Using `fping` for pure parallel ICMP echo sweep:
```sh
fping -asgq 172.16.5.0/23
```
Use the above when:
- You want speed and a large range swept as a first pass
- ICMP echo is not filtered on the internal network

> [!INFO] Flag Explanation
> `-g` = Generate the target list from the CIDR/range (sweep mode)
> `-a` = Print only alive hosts
> `-s` = Print cumulative stats at the end
> `-q` = Quiet, suppress the per-probe noise


> [!WARNING] When Proxying...
> Over a SOCKS proxy you must use TCP-connect scans (`nmap -sT -Pn`), because SYN scans do not traverse a proxy. A real route (ligolo, sshuttle) lets every scan type work unmodified.
> 
> See [[03G - Pivoting & Tunneling]].

# Faster Front-ends

`nmap` is thorough, but it's also slow. A full `-p-` scan across even a handful of hosts can be painful to sit through. These are VERY useful for finding open ports quickly, then pass off to `nmap` for the targeted deep scanning.

> [!WARNING] The Trade-offs to Speed
> Running these scans can knock things over. Especially fragile devices, printers, old appliances, and anything in an OT or ICS segment.
> 
> Accuracy potentially plummets too, meaning open ports can be missed. So use these as a lead, but trust the `nmap` output.
> 
> Neither of these go through a proxy very well, so save these for when you have a direct and unproxied foothold on the segment.
> 
> These are also **VERY** loud. Any IDS will light up like a Christmas tree. Very useful for internal assessments where stealth is not needed, but terrible for a covert red team engagement.

`rustscan`'s whole job is to rip through all 65535 ports in seconds and hand the open ones straight to `nmap`:
```bash
rustscan -a $IP --range 1-65535 --ulimit 5000 -- -sC -sV -oN nmap/rustscan.txt
```

> [!NOTE]
> `--ulimit 5000` is the fix for "why is rustscan slow." Everything after `--` passes through to nmap. Use rustscan/masscan for discovery speed and nmap `-sCV` / `--script vuln` for depth. For a single box, `nmap -p- --min-rate 10000` alone is usually enough and keeps you in one tool.

`masscan` is a different animal. It uses its own stateless TCP/IP stack and does asynchronous SYN scanning (similar design idea to `zmap`). Use this over plain `nmap` for scale and breadth. It primarily answers the question of "who on this entire range has port X open" and does it quickly.
```bash
# Single host, full port
sudo masscan -p1-65535 $IP --rate=1000 -oL nmap/masscan.txt

# Subnet range, but targeted ports
sudo masscan 10.10.10.0/24 -p80,443,445 --rate=10000
```

> [!WARNING] Privilege Requirement!
> Due to the fact that `masscan` crafts raw packets, it does require `sudo` permissions to run.

# See Also

- [[03B - Service Enumeration (Ports)]] for what to do with each open port
- [[02B - Web Enumeration]] when any web port is open
- [[01C - CVE Triage]] once you have a version string
- Source: *OSCP Field Manual* section 2.0
