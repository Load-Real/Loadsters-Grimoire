---
status: working
tags:
  - network
  - snmp
  - ldap
  - rpc
  - enumeration
---
> *SNMP, LDAP, and RPC Enumeration:* Three protocols that leak users, groups, processes, and sometimes credentials with little or no authentication. Split out from [[Service Enumeration]] because each has a deep command set worth its own reference.

## SNMP - 161/udp

Leaks users, running processes (sometimes with credentials on the command line), and listening ports, a lot of free recon.

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP  # find a community string
snmpwalk -v2c -c public $IP                              # dump everything
snmpwalk -v2c -c public $IP 1.3.6.1.4.1.77.1.2.25        # Windows usernames
snmpwalk -v2c -c public $IP 1.3.6.1.2.1.25.4.2.1.2       # running processes (may leak cmdline creds)
snmp-check $IP -c public                                 # prettified summary
```

> [!note]
> `public` (read) and `private` (write) are the most common community strings, `onesixtyone` brute-forces others. Usernames and process lists are the two most useful subtrees. Any usernames feed [[Brute Forcing and Spraying]].

## LDAP - 389 / 636 / 3268 / 3269

Usually a DC. An anonymous bind sometimes dumps the whole directory.

```bash
ldapsearch -x -H ldap://$IP -s base namingcontexts       # find the base DN
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com"          # anonymous full dump
# authenticated, filtered to users + the attributes that leak creds:
ldapsearch -x -H ldap://$IP -D 'user@corp.com' -w 'pass' -b "DC=corp,DC=com" "(objectClass=user)" sAMAccountName description
# clean username list for spraying/roasting:
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com" "(&(objectclass=user))" sAMAccountName | grep sAMAccountName | cut -d' ' -f2
# password policy over LDAP:
ldapsearch -x -H ldap://$IP -b "DC=corp,DC=com" | grep -m1 -B10 pwdHistoryLength
```

> [!note]
> `-x` is simple (anonymous) auth. Pull the `description` and `info` attributes specifically, creds hide there. `(userAccountControl:1.2.840.113556.1.4.803:=8388608)` filters for password-not-required accounts. `windapsearch --dc-ip $IP -u user@corp.com -p pass -U` is the friendlier wrapper. Continue in [[AD Enumeration]].

## RPC - 135 / 139 / 593

A null RPC session can enumerate domain users and groups with no credentials.

```bash
rpcclient -U "" -N $IP            # null-session connect
rpcclient -U 'user%pass' $IP      # authenticated
```
High-value in-session commands:
```
enumdomusers                 -- all domain users (+ RIDs)
enumdomgroups                -- all groups
queryuser <RID>              -- detail on one user (last logon, flags, description!)
querygroupmem <group-RID>    -- who's in a group
lookupnames <user>           -- name -> SID
lookupsids <SID>             -- SID -> name (RID-cycle usernames)
getdompwinfo                 -- password policy (lockout!) BEFORE spraying
querydominfo                 -- domain / server role summary
enumprivs                    -- privileges
createdomuser / setuserinfo  -- if you have write rights
```

> [!example] rpcclient one-liner password spray (only hits shown)
> ```bash
> for u in $(cat users.txt); do rpcclient -U "$u%Welcome1" -c "getusername;quit" $IP 2>/dev/null | grep Authority; done
> ```

> [!note]
> The `description` field from `queryuser` is a classic place admins leave passwords. Run `getdompwinfo` before any spray so you know the lockout threshold.

## See also

- [[Service Enumeration]] for the SMB workflow these sit alongside
- [[AD Enumeration]] for where this leads on a domain
- [[Brute Forcing and Spraying]] for the username lists you build here
- Source: [[OSCP Field Manual]] section 2.5
