---
status: working
tags:
  - credentials
  - cracking
  - hashcat
  - john
---
> *Offline Hash Cracking:* Turning a recovered hash into a plaintext password. Trigger: you pulled a hash from a dump, a file, or a roast. Identify it, then crack with the right mode. Discipline, the moment a password cracks, stop and spray it everywhere ([[Brute Forcing and Spraying]]) before continuing.

## Hashcat (GPU, the default)

```bash
hashid '<hash>'                                                    # identify the type
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt        # crack
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule   # + mutations
```

> [!note]
> Common modes: `0` MD5, `100` SHA1, `1000` NTLM, `1800` sha512crypt, `3200` bcrypt, `5600` NetNTLMv2, `13100` Kerberoast TGS, `18200` AS-REP. Full list on the hashcat example-hashes page ([[Reference Sites]]). Format-specific `*2john` helpers turn files into crackable hashes: `zip2john`, `ssh2john`, `keepass2john`, `office2john`, `rar2john`.

## John (CPU fallback)

Better than hashcat on CPU, and it handles the odd formats the `*2john` tools produce.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt          # auto-detect format
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt   # force a format
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=Jumbo hash.txt # + rules
john --show hash.txt                                                # show cracked results
john --incremental hash.txt                                         # pure brute force (last resort)
```

> [!note]
> Common `--format` values: `NT` (NTLM), `netntlmv2` (Responder captures), `krb5tgs` (Kerberoast), `krb5asrep` (AS-REP), `sha512crypt` (Linux `/etc/shadow`), `md5crypt`. `john --list=formats` if unsure. Combine `/etc/passwd` + `/etc/shadow` with `unshadow` before cracking Linux hashes: `unshadow passwd shadow > crack.me`.

## See Also

- [[Brute Forcing and Spraying]] for what to do the instant a hash cracks
- [[Wordlists and Generation]] for the lists and rules
- [[AD Attacks]] (Kerberoast/AS-REP produce `-m 13100`/`-m 18200`), [[Finding Credentials]] (SAM, shadow, KeePass)
- Source: [[OSCP Field Manual]] section 2.3
