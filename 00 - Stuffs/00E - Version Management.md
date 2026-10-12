---
status: solid
tags:
  - meta
  - git
  - github
  - workflow
  - tooling
---
# Git-based Management

I am leverage two layers in this design:
- **Git** is the version-control engine.
- **GitHub** is the remote hub. Each machine holds a *full* clone.

## A. Repeatable Playbook (any new repo)

### A1. One-time Per Machine

Do this once on each computer, not one per repo.

**Set an explicit Git identity** so Git never auto-derives a wrong one from `username@hostname` (if forgotten, refer to the Tailnet gotcha in the Reference section):
```bash
git config --global user.name "Louden"
git config --global user.email "ID+username@users.noreply.github.com"
```

> [!TIP]
> Use GitHub's no-reply email (Settings → Emails → "Keep my email addresses private") so your real address is never stamped into commit history. A commit email is visible to anyone who can see the repo and lives in history forever.

**Generate an SSH key** and register it with GitHub so you authenticate with a key, not a password:
```bash
ssh-keygen -t ed25519 -a 100 -C "repo-<machine>"
cat ~/.ssh/id_ed25519.pub
```

Paste the printed public key into GitHub → Settings → SSH and GPG keys → New SSH key. Name the key per machine so a lost laptop means revoking only that one key. Then test:
```bash
ssh -T git@github.com
```

> [!note] 
> `-a 100` raises the KDF work factor so a stolen encrypted key file is harder to brute-force. Use a long random passphrase stored in your password manager, not a memorized one. If `~/.ssh/id_ed25519` already exists, save to a distinct filename (e.g. `id_ed25519_github`) and point SSH at it with a `~/.ssh/config` entry:
> 
> ```
> Host github.com
>     HostName github.com
>     User git
>     IdentityFile ~/.ssh/id_ed25519_github
>     IdentitiesOnly yes
> ```

**Load the key into the agent** once per session so you are not retyping the passphrase on every push:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

## A2. Create the Repo (first machine only)

```bash
cd "/path/to/project"
git init
git branch -m main
# create a .gitignore first (see Part B for the Obsidian one)
git add -A
git commit -m "Initial commit"
```

On `github.com`: New repository, Private, no README/`.gitignore`/license. The connect and push:
```bash
git remote add origin git@github.com:YOURNAME/<repo-name>.git
git push -u origin main
```

`-u` sets `origin/main` as the upstream, so future `git push`/`git pull` needs no arguments. Confirm with `git status` ("up to date with 'origin/main'").

## A3. Daily Loop

From inside the repo, after making changes:
```bash
git add -A                               # stage everything changed
git commit -m "what I did, not a timestamp"
git push
```

> [!NOTE]
> Check before committing if unsure: `git status` (changed files) or `git diff` (actual line changes).

> [!NOTE]
When switching machines, **pull before editing:** `git pull` in order to avoid conflicts.

## A4. Adding a New Machine

Do A1 on that machine first (identity + SSH key), then clone instead of init:
```bash
git clone git@github.com:YOURNAME/<repo-name>.git
```

From then on, it is a peer.

### A4.a Adding a Windows Machine

Most of this document is written in a Linux context, but I do occasionally use Windows for my work. This section explains how to add Windows as a peer.

**Install Git for Windows**
Not preinstalled. Install via:
```powershell
winget install Git.Git
```

This puts `git` and the OpenSSH client on PATH (which the Obsidian Git plugin also needs).

**Identity**
Same as any machine. Re-run:
```powershell
git config --global user.name "Louden"
git config --global user.email "ID+username@users.noreply.github.com"
```

**SSH Key**
Run the usual keygen and utilize the following PowerShell one-liner for transferring the public key to the clipboard:
```powershell
ssh-keygen -t ed25519 -a 100 -C "grimoire-<windows-machine-name>" Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

From there, paste into GitHub → Settings → SSH and GPG keys → New SSH key.

**Enable the SSH Agent (Windows-specific step)**
> [!WARNING]
> Changing a service startup type requires an **elevated** shell. In a normal PowerShell these commands fail with "Access is denied." Open an admin window first: `Start-Process powershell -Verb RunAs`.

From the **admin** PowerShell (one time, persists across reboots):
```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Then, from a **normal** PowerShell (avoid running Git or key ops as admin):
```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
ssh -T git@github.com
```

**Line Endings**
> [!WARNING]
> Windows can swap LF for CRLF and make whole files show as "changed" when nothing was edited. The vault is LF (Obsidian), so keep LF everywhere.

The robust fix is the repo-level `.gitattributes` (`* text=auto eol=lf`) from the *References* section, which enforces LF on every machine at once. As a per-machine backstop on Windows:
```powershell
git config --global core.autocrlf false
```

**Clone Into Vaults Folder**
Run from inside the parent folder where your other vaults live, `git clone` creates the subfolder (do not pre-create it):
```powershell
cd "$env:USERPROFILE\Documents\Obsidian Vaults"
git clone git@github.com:YOURNAME/Loadsters-Grimoire.git "Loadster's Offensive Security Grimoire"
cd "Loadster's Offensive Security Grimoire"
git status
```

> [!NOTE]
> The final argument is just this machine's folder name and can be anything, the repo contents are identical regardless. Double quotes handle the space in "Obsidian Vaults" and the apostrophe in "Loadster's" with no escaping.

## A5. Conflicts

Only happens when two machines change the same lines without syncing between. Git marks the file with `<<<<<<<`/`=======`/`>>>>>>>`. For Markdown, open the file, keep the text you want, delete the markers, save, then `git add` + `git commit`. The pull/push habit prevents nearly all of these.

# B. Grimoire-specific Config

## B1. Metadata

- Repo name = `Loadsters-Grimoire`
- Accessibility = `Private`
- Remote = `git@github.com:YOURNAME/Loadsters-Grimoire.git`
- Branch = `main`
## B2. The Boundary Rule

This vault holds only **public-safe** content. On-the-job engagement learnings get refactored into generic technique notes here (e.g., "pentesting OMS applications"), never the exact company-/org-specific details.

I wrote this in to provide coverage for my own ass.

## B3. The `.gitignore` (Obsidian-specific)

Tracks the portable config so a clone behaves the same, ignores per-machine changes that cause noise and conflicts:
```.gitignore
# Obsidian per-machine state and cache (noise, causes conflicts)
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/cache
.trash/

# OS cruft
.DS_Store
Thumbs.db
desktop.ini

# Editor leftovers
*.tmp
*.swp
```

The above are tracked so that plugins, theme, and hotkeys travel with the vault.

## B4. Obsidian Git plugin (optional convenience)

Install from Community Plugins ("Obsidian Git" by Vinzent). Effectively does the above, but with some conveniences:
- Pull on startup
- Push on commit
- Auto-commit

## B5. Images and Binaries

Image files like `spellbook.jpg` and `SSTI_Tree.png` are tracked so clones render correctly. Git stores each new version of a binary in full rather than as a diff, so if large screenshots pile up and get re-edited often, the repo can bloat. **Not a concern at the current scale, but if it becomes one, this is a reminder to move binaries to Git LFS.**

# References (Gotchas)

> [!WARNING] Committer Auto-derived from Hostname
> If you commit before setting `user.name`/`user.email`, Git invents an identity from `username@hostname` (this once produced an `louden@thechunk.crt.tail` from the tailnet MagicDNS domain). That identity is baked into the commit object permanently. Fix for a single unpushed commit:
> ```bash
> git commit --amend --reset-author --no-edit
> ```
> For many pushed commits it is a full history rewrite (`git filter-repo` + a force-push), so set the identity (A1) before the first commit.

> [!WARNING] History is Forever
> A secret commited and then deleted still lives in history and every clone. If one is ever pushed, treat it as burned and rewrite history (`git filter-repo`/BFG) + a force-push. 
> 
> **Defense:** `git status` before every commit, and the B2 boundary. A pre-commit `gitleaks` scan is useful.

> [!NOTE] Ignoring an Already-tracked File
> Adding a path to `.gitignore` does not untrack a file already committed. Stop tracking it with:
> ```bash
> git rm --cached <file>
> ```
> ...then commit. (Deleting `.git` and re-initializing sidesteps this, but only before the first push)

**Verify Identity on a Commit:**
```bash
git log --format='%an <%ae>  /  %cn <%ce>'
```

# Version History

| Version                  | Date       | Changes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------ | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v0.0.1 (Genesis)         | 09/26/2026 | Document Creation<br>11 - Tooling<br>-- Salesforce                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| v0.0.2 (OSCP Update)     | 09/30/2026 | Integration with OSCP Field Manual v3...<br>- 00 - Stuffs<br>-- Inbox<br>-- Famous Exploits Lookup<br>-- Master Routing Table<br>-- {Stub} OSCP Field Manual<br>-- Version Management<br>- 01 - Recon & OSINT:<br>-- Network Scanning<br>-- CVE Triage<br>- 02 - Web & API<br>-- Web Enumeration<br>-- LFI, RFI & Path Traversal<br>-- Command Injection<br>-- File Upload to Webshell<br>-- Admin Panels & Default Creds<br>-- SQL Injection<br>-- Other Injection Classes<br>- 03 - Network, AD & Infrastructure<br>-- Service Enumeration (Ports)<br>-- SNMP, LDAP, RPC<br>-- AD Enumeration<br>-- AD Attacks<br>-- AD Lateral Movements & Persistence<br>-- Pivoting & Tunneling<br>- 05 - Host & Post-Exploitation<br>-- Post-Exploitation Loot Loop<br>-- Linux Privilege Escalation<br>-- Windows Privilege Escalation<br>-- Finding Credentials<br>-- Reverse Shells<br>-- Shell Stabilization<br>-- File Transfer<br>- 06 - Red Team Operations<br>-- Responder & NTLM Relay<br>- 08 - Exploit & Payload Development<br>-- MSFvenom Payloads<br>- 09 - Credential Access & Cracking<br>-- Brute Forcing & Spraying<br>-- Hash Cracking<br>-- Wordlists & Generation<br>11 - Tooling<br>-- High Frequency One-Liners<br>-- Environment Setup<br>-- Reference Sites<br>-- Tool Map<br>-- Rubeus<br>-- BloodyAD |
| v0.0.3 (HyperGit Update) | 10/05/2026 | Modification to this MD. I moved the table down and added in the Git-based management steps above.<br><br>Also went and fixed ***nearly*** every Obsidian hyperlink in the vault. The only ones left are links to the now-embedded OSCP Field Manual v3.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| v0.0.4 (WinGit Update)   | 10/12/2026 | Modification to this MD. Addition of section *A4.a Adding a Windows Machine*.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| *Future release*         |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| *Future release*         |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| *Future release*         |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| *Future release*         |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| *Future release*         |            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

