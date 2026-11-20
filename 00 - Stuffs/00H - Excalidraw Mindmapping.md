# Why?

Because I want to create a series of different mind maps for different functionalities, and the Excalidraw plug-in lets me embed and integrate with my Grimoire directly.

# Excalidraw Usage Rules in My Grimoire

> [!TODO]
> Write up the rules on how, why, and when I use Excalidraw in my notes. Specifically, a standard.

# Excalidraw Usage Reference

> [!TIP] Useful Course
> There is a useful course centered around the usage of the plug-in, called "Welcome to Excalidraw Essentials". It contains video demonstrations of the various commonly used features within Obsidian.
> 
> Link: https://community.sketch-your-mind.com/t/welcome-to-excalidraw-essentials/722

> [!TODO]
> Spruce up these notes from the Excalidraw Essentials course.

Interact with Excalidraw through the command palette (`CTRL + P`).

Type "Create" to find the various drawing creation methods. You can use the embed ones to immediately embed into the current working document.

Can make direct links to drawings, just like other MDs, with the double square brace syntax.

Can insert other images, either from the vault or system, into an Excalidraw drawing. It won't be a part of .excalidraw file, and is instead referenced. This means you can leverage this to point to more diagrams as you reach the end of one. For example, you finish up the remote enumeration -> foothold mindmap and need to shift to the local enumeration -> privilege escalation mindmap.

Idea: You can embed MDs as interactive boxes, and literally make edits to the document from the drawing. KInd of like an iframe. With that in mind, I can create a scratch_note document so that when I identify new nodes to add to my mindmaps, I can temporarily write them up within the actual drawing file.

# Useful Known Mindmaps

Orange Cyberdefense Mindmaps: https://orange-cyberdefense.github.io/ocd-mindmaps/

C0nd4's Linux/Windows OSCP Privilege Escalation Mindmaps: https://github.com/C0nd4/OSCP-Priv-Esc

# My Mindmap Structure

I will be creating a series of mindmaps for certification exams and personal use. I want easy maintenance, so I won't be bloating up a single map too much.

**OSCP Mindmaps:**
- Reconnaissance and Remote Enumeration → Initial Access/Foothold
	- Standalone map for Web Enumeration → Initial Access/Foothold
- Host Enumeration → Lateral Movement & Privilege Escalation
- Active Directory Enumeration → Active Directory Abuse & Privilege Escalation
- Tunneling/Pivoting map for all others to reference; serves as an "overlay"
	- > [!NOTE] May put equivalents in the original mindmaps for pivot variants
- Utilities map (e.g., file transfer, shell stabilization, payload/listener prep) since these repeat; makes more sense to link to one
- Credential map, which contains the "what does this unlock?", spraying, and other details based on the TYPE of cred (secret vs NTLM hash vs Kerberos TGT/TGS, for example)
	- Also should include the files that likely house them, that you haven't extracted from yet (e.g., backup)
- Reminders:
	- "Stuck?" node within each mindmap that includes things like "re-enum with fresh creds", "check UDP and full port range", "revisit skipped services"
	- Proof-capture checkpoint nodes, reminding me to grab the flag; at the end of each priv change

