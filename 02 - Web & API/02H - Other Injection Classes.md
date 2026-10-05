---
status: working
tags:
  - web
  - xxe
  - ssti
  - injection
---
> *Other Injection Classes:* XXE and SSTI, plus the confirm-before-you-invest discipline they demand. Reach for these when the common web paths are exhausted and the app takes structured input (XML, templates, serialized objects).

## XXE (the app parses XML you control)

Turns an XML parser into a file-reader, and sometimes into SSRF or RCE. Swap the `SYSTEM` URI to read whatever config you want.

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<element>&xxe;</element>
```

## SSTI (template injection)

Test with a math probe first, then escalate only once you have confirmed the engine is evaluating your input.

```
{{7*7}}      -> 49  means a template engine is evaluating your input (Jinja2/Twig/etc.)
${7*7}       -> 49  (Java/EL-style engines)
```

Below is a useful testing tree to identify the specific templating engine behind the scenes.
![[SSTI_Tree.png]]

> [!note]
> SSTI is engine-specific once confirmed. Jinja2 has a well-known `__globals__` → `os.popen` chain to RCE, with WAF-avoiding payloads on PayloadsAllTheThings ([[00B - References]]). On OSCP, treat SSTI as a rabbit-hole signal unless `{{7*7}}` clearly returns `49`. Confirm before investing time, this is exactly the kind of path that eats an hour for nothing.

## See Also

- [[02B - Web Enumeration]] for finding the input
- [[05D - Finding Credentials]] for what XXE file-read should hunt for
- Source: *OSCP Field Manual* section 3.6
