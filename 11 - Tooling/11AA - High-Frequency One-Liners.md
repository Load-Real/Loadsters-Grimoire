---
status: working
tags:
  - tooling
  - one-liners
  - reference
---
> *High-frequency One-liners:* The handful of commands you type constantly, kept in one place so they are always a copy away.

```bash
python3 -m http.server 80                       # serve files to a victim (HTTP)
impacket-smbserver share . -smb2support         # SMB drop point (Windows transfers)
rlwrap nc -lvnp 443                             # listener with history/arrow keys
base64 -w0 file                                 # encode for copy-paste transfer (base64 -d on the other side)
```

> [!note]
> The first two are your constant companions, one HTTP and one SMB drop point. Which you use depends on the victim's egress ([[File Transfer]]). `-smb2support` is required for modern Windows to talk to the impacket server.

## See Also

- [[05G - File Transfer]] for choosing between HTTP and SMB
- [[05E - Reverse Shells]] for the listener's other half
- Source: [[OSCP Field Manual]] section 10.1
