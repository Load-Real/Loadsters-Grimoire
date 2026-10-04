---
status: solid
tags:
  - stuffs
  - conventions
  - index
---
![[spellbook.jpg|center|375]]

---
This vault is basically just every single thing that is relevant to my pentesting, red teaming, or whatever else in offensive security I wanna do. I'll build it all up over time, referencing other people's massive knowledge dumps so I don't reinvent the wheel.

Everything listed is not just an observation, it's the *how* behind everything. What exactly I run, click, and watch for. Why I do it.

I don't like scanning my notes for eons trying to find things, so I want to establish the structure up front. As such, I presently have three **MAJOR** buckets
- **Surfaces:** Reusable attack mechanics, the bulk of the vault. Recon, Web, API, Cloud, AD, Host Privesc, Post-Ex, Cracking.
- **Platforms:** A specific product's quirks, whether SaaS or on-prem. ServiceNow, Salesforce, M365. Stays thin. Holds only what is specific to that product and links to the Surface notes for the mechanics.
- **Tooling:** How to drive the instruments I test _with_. Burp, Horizon3, Terra, the my own machine environment. Also thin, links back to Surfaces for the "what."

**How I Add to this Grimoire:**
1. Mid-engagement, drop a rough capture in `00C - Inbox`. Do not stop to format.
2. Paste the raw command or finding, plus a link to the source (HackTricks page, PayloadsAllTheThings, the platform's own docs).
3. Later, during a cleanup pass, rewrite it into _my_ commands with my real flags, fit it to the note template, file it in the right bucket, and delete the Inbox copy.

**How I Keep This Grimoire Tidy:**
- Set `status:` at the top of each note: `stub`, `working`, or `solid`.
- Run a pass at the end of each engagement or once a month: promote stubs, split bloated notes, fix dead links, clear the Inbox.
- Moving notes is cheap. Obsidian updates `[[links]]` on move, so file it roughly right and relocate later once the shape is obvious. Do not agonize over perfect placement.