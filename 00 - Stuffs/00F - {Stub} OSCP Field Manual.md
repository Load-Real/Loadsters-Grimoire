---
status: stub
tags:
  - stuffs
  - oscp
  - study
---
> *OSCP Field Manual:* Landing note for the intact exam-study manual. Drop the full `OSCP_Field_Manual_v3.md` here (replace this stub, or paste its body below this line) so the many `[[OSCP Field Manual]]` links across the vault resolve to it.

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

So the manual is the exam-prep tool, and the disciplines are the career-wide
reference it seeded. As you deepen a discipline note, the manual can link down to
it, but the manual itself stays intact.

> [!note]
> The exam-agnostic content already lives, expanded, across the disciplines:
> §2.0 → [[Network Scanning]]; §2.1 → [[Service Enumeration]]; §2.2 → [[Web Enumeration]];
> §2.3 → [[Brute Forcing and Spraying]] + [[Hash Cracking]]; §2.4 → [[Finding Credentials]];
> §2.5 → [[SNMP LDAP RPC]] + [[Master Routing Table]]; §3 → the [[SQL Injection|Web & API]] notes;
> §4.0 → [[Reverse Shells]]; §4.1 → [[msfvenom Payloads]]; §4.2 → [[Famous Exploits Lookup]];
> §5 → [[Linux Privilege Escalation]]; §6 → [[Windows Privilege Escalation]];
> §7 → [[AD Enumeration]] + [[AD Attacks]] + [[AD Lateral Movement and Persistence]] + [[Rubeus]] + [[BloodyAD]];
> §7.7 → [[Responder and NTLM Relay]]; §8 → [[Post-Exploitation Loot Loop]];
> §9 → [[Wordlists and Generation]] + [[Reference Sites]]; §10 → [[Tool Map]] + [[Environment Setup]] + [[High-Frequency One-Liners]];
> §11 → [[Shell Stabilization]] + [[File Transfer]] + [[Pivoting and Tunneling]].
