---
status: working
tags:
  - active-directory
  - enumeration
  - bloodhound
---
> *AD Enumeration:* Mapping the domain from a given credential (or none) so you can see the path to Domain Admin before you touch anything. This is the first move of the AD chain, foothold → enumerate → get creds → move laterally → reach the DC. Trigger: port 88/389 anywhere, or a domain-joined box. Treat all machines in the set as one objective.

> [!danger] OSCP Note
> Responder poisoning and all LLMNR/NBT-NS/mDNS answering are exam-banned. The relay flows live in [[Responder and NTLM Relay]] and are lab-only. Everything in this note is exam-legal.

## Unauthenticated (DC IP known)

Some enumeration works before you even use the given creds.

```bash
kerbrute userenum -d corp.com --dc $IP /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
crackmapexec smb $IP -u '' -p '' --shares        # null SMB session
rpcclient -U "" -N $IP -c 'enumdomusers'         # null RPC user enum
nslookup -type=srv _ldap._tcp.dc._msdcs.corp.com $IP   # find DCs via DNS SRV
```

> [!note]
> `kerbrute userenum` is safe to run first, it validates usernames against Kerberos without triggering lockouts, giving you a user list for spraying and AS-REP roasting ([[AD Attacks]]).

## Authenticated: BloodHound Collection

With any domain credential, collect the whole graph. BloodHound then shows the shortest path to Domain Admin.

```bash
bloodhound-python -u user -p 'pass' -d corp.com -ns $IP -c all --zip   # remote from Kali
```
```cmd
.\SharpHound.exe -c All --zipfilename loot                              # on a Windows foothold
```

> [!note]
> `-c all` gathers every collection method, the `.zip` imports into the BloodHound GUI (setup in [[Environment Setup]]). Mark owned nodes (right-click → Mark as Owned) so path queries start from what you control.

## Manual Authenticated Enumeration (fallback + confirmation)

```bash
crackmapexec smb $IP -u user -p pass --users --groups --shares --pass-pol
crackmapexec smb $IP -u user -p pass --loggedon-users     # hunt admin sessions
crackmapexec smb $IP -u user -p pass --pass-pol           # lockout BEFORE spraying
```
```cmd
.\Snaffler.exe -d CORP.LOCAL -s -v data                   # hunt creds across readable shares
```

> [!note]
> `--pass-pol` tells you the lockout threshold before you spray, critical so you do not lock accounts. `--loggedon-users` finds a box where a Domain Admin is logged in, a credential-dump target ([[AD Lateral Movement and Persistence]]). Full domain-user/group enumeration lives in [[SNMP LDAP RPC]] (rpcclient). Snaffler is the fastest way to surface passwords in shares from a foothold.

## BloodHound + Cypher

Run the prebuilt analytics first, Shortest Path to Domain Admins, Find Principals with DCSync Rights, Find Kerberoastable Accounts, Shortest Paths from Owned Principals. Then reach for raw Cypher:

```cypher
// mark a node owned
MATCH (u {name:"USER@CORP.COM"}) SET u.owned=true RETURN u
// all kerberoastable users (have an SPN)
MATCH (u:User {hasspn:true}) RETURN u.name
// all AS-REP roastable users (no preauth)
MATCH (u:User {dontreqpreauth:true}) RETURN u.name
// shortest path from ALL owned nodes to Domain Admins
MATCH (o) WHERE o.owned=true MATCH p=shortestPath((o)-[*1..]->(g:Group))
WHERE g.name STARTS WITH "DOMAIN ADMINS" RETURN p
// who can DCSync the domain
MATCH p=(n)-[:GetChanges|GetChangesAll*1..]->(d:Domain) RETURN p
// where can my owned users admin / RDP / PSRemote?
MATCH p=(u {owned:true})-[:AdminTo|CanRDP|CanPSRemote]->(c:Computer) RETURN p
// dangerous ACL edges FROM anything I own
MATCH p=(u {owned:true})-[:GenericAll|GenericWrite|WriteDacl|WriteOwner|ForceChangePassword|AddMember]->(t) RETURN p
// simple sanity
MATCH (c:Computer) RETURN c.name
MATCH (u:User) RETURN u.name
```

> [!tip]
> `bloodhound.readthedocs.io` and the hausec Cypher cheatsheet are the readable references. `wadcoms.github.io` and `thehacker.recipes` map each BloodHound edge to the exact command that abuses it, that mapping is the whole point of [[AD Attacks]].

## See Also

- [[AD Attacks]] for turning enumeration into creds and privilege
- [[AD Lateral Movement and Persistence]] for moving once you have a hash
- [[Service Enumeration]] and [[SNMP LDAP RPC]] for the port-level entry points
- Source: [[OSCP Field Manual]] sections 7.0 and 7.0.1
