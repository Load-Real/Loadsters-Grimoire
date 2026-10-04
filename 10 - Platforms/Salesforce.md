---
status: solid
tags:
  - todo
---
Salesforce is a multi-tenant SaaS Customer Relationship Management (CRM) platform. When you are testing a Salesforce org, you are almost always testing one of three surfaces:
- **Internal Salesforce (Lightning Experience):** The primary UI for employees and agents, accessed at `https://<orgname>.my.salesforce.com` or a custom domain. Only accessible by licensed users.
- **Experience Cloud sites:** Externally-facing portals and community sites built on top of Salesforce data, accessed at `https://<orgname>.my.site.com/<sitepath>` or a custom vanity domain. Lower-trust users (customers, vendors, partners) interact here. Each site has its own guest profile and member profiles.
- **API Surface:** The Salesforce REST API (`/services/data/vXX.X/`), SOAP API, Bulk API, and custom REST endpoints (`/services/apexrest/`) are all reachable by authenticated API clients and services accounts.

> [!NOTE]
> The raw Salesforce origin hostnames (`*.lightning.force.com`, `*.my.salesforce.com`, `*.my.site.com`) are always publicly reachable regardless of any CDN or edge controls placed in front of a custom vanity domain. **Network-layer controls applied at a CDN only cover the vanity domain path.**

---
# Setup
This is the Salesforce administrative console, accessible via the gear icon at the top right of Lightning Experience or by navigating to `/lightning/setup/SetupOneHome/home`. It is the single most important screen for configuration review during a pentest.

![[Salesforce_Setup_Location.png|337]]

> [!NOTE]
> You need an internal user account with at minimum *View Setup and Configuration* permission to access it.

> [!TIP]
> Use Setup's quick-find search box (top left of the Setup sidebar) to jump directly to any section by keyword. This is faster than navigating the tree.

![[Salesforce_Setup_QuickFind_Location.png]]
## Sharing Settings
**Path:** Setup → quick-find "Sharing Settings"

**What it Shows:** The Organization-Wide Defaults (OWD) table listing every object's Default Internal Access and Default External Access. Also shows all configured sharing rules per object, org-wide sharing preferences, and guest user settings.

**What to Look For:**
The OWD table is the foundation of the entire record access model.

> [!NOTE]
> Read the Default Internal Access and Default External Access columns separately as they are independent settings and an object can be open internally and Private externally or vice versa.

Access levels to understand:
- **Private:** Only the record owner and users above them in the role hierarchy can access the record.
- **Public Read Only:** All users can read, only the owner can edit.
- **Public Read/Write:** All users can read and edit.
- **Public Read/Write/Transfer:** Same as above plus ownership transfer (Lead and Case only).
- **Controlled by Parent:** Access is inherited from a parent object's OWD.
- **Public Full Access:** All users can read, edit, delete, and transfer (Campaign only).

For external-facing sites, the Default External Access column is what governs community and guest user baseline record visibility. Private external with Public Read/Write internal is the expected and correct posture for most sensitive objects. If any sensitive object shows *Public Read/Write* or *Public Read Only* external access, that is a high-priority finding.

Sharing rules section shows anything that widens access beyond the OWD baseline. Rules only ever grant more access, never restrict. Look for rules that share sensitive objects broadly with all internal users, all portal users, or guest users.

Org-wide sharing preferences to check:
- **Secure guest user record access:** Should be enabled. If off, guest users can potentially be granted broad record access through sharing rules.
- **Portal User Visibility:** If enabled, portal users can see each other. Usually should be off.
- **Site User Visibility:** Same for Experience Cloud site users.
- **Grant site users access to related cases:** Enables community users to see cases related to their account. Confirm this is intentional and correctly scoped.
## Profiles
**Path:** *Setup* → quick-find "Profiles"

**What it shows:** Every profile in the org. Profiles define the baseline set of permissions for a user including object CRUD permissions, field-level security, Apex class access, page layout assignments, and system permissions.

**What to look for:**
Distinguish standard profiles (shipped by Salesforce, cannot be deleted) from custom profiles (org-created, marked with a checkmark in the Custom column). Custom profiles are where over-provisioning most commonly hides.

For each relevant profile, click into it and review each of the following:
**System permissions:**
Look specifically for *View All Data*, *Modify All Data*, *Author Apex*, *Manage Users*, *Customize Application*, and *API Enabled*. ***View All Data* and *Modify All Data* bypass the entire sharing model. A user with either of these can read or write every record in the org regardless of OWD, sharing rules, or role hierarchy. API Enabled controls who can hit the REST and SOAP APIs.**

**Object permissions:**
Per-object CRUD grants. Look for *View All* and *Modify All* at the object level, which bypass sharing for that specific object without needing the org-wide system permissions above.

**Apex Class Access:**
Which Apex classes this profile can execute. This is the gate for `@AuraEnabled` and `@RestResource` methods. If a guest or community profile has access to a sensitive class, its methods are reachable by that audience.

**Field permissions:**
Per-field read and edit grants. This is where FLS is configured per profile. Checking a handful of sensitive fields here is faster than the full *Field Accessibility* tool for a quick spot-check.

> [!Note: High Value-Profiles to Audit]
> 1) The guest user profile for each Experience Cloud site (found via the site's Public Access Settings rather than the main Profiles list)
> 2) Community/portal member profiles (Customer Community, Partner Community, or custom)
> 3) Any API-only or integration user profiles
> 4) Custom internal profiles that may have been over-provisioned
## Permission Sets and Permission Set Groups
**Path:** *Setup* → quick-find "Permission Sets"

**What it shows:** Modular permission bundles that are layered on top of a user's profile. A user's effective permissions are their profile plus all assigned permission sets combined.

**What to look for:**
Permission sets follow the same internal structure as profiles (system permissions, object permissions, field permissions, Apex class access). Review any permission set that grants View All Data, Modify All Data, API Enabled, or Apex access to sensitive classes, and then check who it is assigned to via the Manage Assignments button.

Permission Set Groups combine multiple permission sets and support muting (removing specific permissions from the combined set). The effective permissions of a group member are the union of all included sets minus any muted permissions. This layering can produce effective permissions that are larger than any individual set.
## Field Accessibility
**Path:** Setup → quick-find "Field Accessibility"

**What it shows:** A grid view of field-level security across all profiles for a given object and field combination.

**What to look for:**
Select the object first, then choose View by Fields to see every field's access state across all profiles at once, or View by Profiles to see all fields for a specific profile. The three states are *Editable*, *Read-Only*, and *Hidden*.

*Hidden* means the field is completely inaccessible to that profile in the standard UI. The critical security question is whether custom Apex code enforces this restriction or bypasses it by querying *Hidden* fields without using `WITH SECURITY_ENFORCED`, `Security.stripInaccessible()`, or `isAccessible()` checks.

> [!TIP]
> During a pentest, focus on sensitive custom fields (card numbers, identity GUIDs, membership numbers, SSNs, health data fields, financial fields) and check whether they are *Hidden* for the community and guest profiles. Then cross-reference with any Apex methods those profiles can execute to determine whether the code actually enforces the Hidden configuration.
## Apex Classes (under Setup)
**Path:** Setup → quick-find "Apex Classes"

**What it shows:** Every Apex class in the org with its API version and namespace.

**What to look for:**
The Security button on any individual class shows exactly which profiles have been granted execute access to that class. This is the fastest way to answer "who can invoke this method" for any `@AuraEnabled` or `@RestResource` class. This is more efficient than opening every profile and checking its Apex class access list.

For any class you have identified as high-interest during code review, click it and check Security to confirm the reachable audience. If a guest or community profile appears in the enabled list for a sensitive class, that class's methods are reachable by that audience through direct Aura or REST calls.
## Sites and Digital Experiences
**Path:** Setup → quick-find "All Sites" (under Digital Experiences)

**What it shows:** Every Experience Cloud site in the org with its status, URL, and framework (Aura, Lightning Web Runtime, etc.).

**What to look for:**
For each site, click into its Workspaces and navigate to Administration. The key sections are:

**Members:**
Which profiles and permission sets are enabled for this site. This is the population of authenticated external users for the site.

**Preferences:** 
Guest user settings including:
- Whether guest access is enabled at all
- Let guest users see other members of this site (enables user enumeration by guests)
- Give guest users access to public Chatter API requests (widens the guest API surface)
- Let guest users view asset files, library files, and CMS content
- Grant site users access to related cases

**Public Access Settings:** 
Opens the guest user's profile directly. This is the most important screen for external-facing site security. Review Apex Class Access, object permissions, and field permissions for the guest profile specifically.
## Health Check
**Path:** Setup → quick-find "Health Check"

**What it shows:** A scored assessment of the org's security configuration against Salesforce's baseline standard. Flags high-risk deviations across session security, password policies, network access, and sharing settings.

**What to look for:**
Things like overly permissive session timeouts, missing IP restrictions on connected apps, guest user sharing rule issues, and clickjack protection settings. Not exhaustive but useful as a first pass before diving into manual review.

> [!TIP]
> Run this early in an assessment for a fast overview of obvious configuration gaps.
## Connected Apps
**Path:** Setup → quick-find "Connected Apps"

**What it shows:** Every OAuth connected app registered in the org, including external integrations, mobile apps, and the org's own apps.

**What to look for:**
OAuth Policies per App:
Specifically whether refresh tokens are allowed and how long they last, whether admin approval is required before users can authorize, and which scopes the app requests. An app with full access scope and no admin approval requirement is a broad OAuth surface.

IP Relaxation Settings:
Whether the connected app bypasses login IP range restrictions. An integration user with a narrow IP allowlist on their profile but an IP-relaxed connected app can authenticate from any IP via that app.
## Named Credentials and External Credentials
**Path:** Setup → quick-find "Named Credentials"

**What it shows:** Stored endpoint definitions and authentication configuration for external callouts from Apex. The actual credential values are not visible, but the endpoint URLs, authentication type, and which Apex code uses them can be mapped.

**What to look for:**
Endpoint URLs reveal what external systems the org integrates with. Authentication type (Basic, OAuth, JWT, Certificate) indicates what kind of credential is stored. If Apex code making outbound callouts to sensitive systems can be influenced by user input, the named credential becomes a pivoting point.
## Object Manager
**Path:** Setup → quick-find "Object Manager" or the Object Manager tab at the top of Setup

**What it shows:** Every standard and custom object in the org with its fields, page layouts, validation rules, triggers, and relationships.

**What to look for:**
For any object of interest, the Fields and Relationships section lists every field with its API name, data type, and a link to its field-level security settings. This is the authoritative source for field API names, which you need for SOQL queries and for matching what you see in code to what exists in the schema.

> [!TIP]
> Custom fields end in `__c`. Fields from managed packages will have a namespace prefix like `namespace__FieldName__c`. If a field name you see in Apex code is not queryable via its apparent name, check Object Manager for the namespaced version.
## Installed Packages
**Path:** Setup → quick-find "Installed Packages"

**What it shows:** Every managed and unmanaged package installed in the org, including the package name, namespace prefix, version, and publisher.

**What to look for:**
The namespace prefix column is critical for resolving field and class naming issues. If you encounter a field or class that does not resolve under its apparent name, cross-reference the namespace list here to find the prefix to prepend.

> [!NOTE]
> Managed package code is the vendor's responsibility rather than the tenant's for security purposes. If a class you are reviewing turns out to belong to an installed managed package, findings should be routed to the vendor's disclosure process rather than treated as tenant custom code.

---
# Developer Console
The Developer Console is an in-browser IDE and query tool accessible via the gear icon → Developer Console, or by navigating to `/_ui/common/apex/debug/ApexCSIPage`. It requires an internal user account with sufficient permissions. It is the primary tool for code review, SOQL execution, and Anonymous Apex during a Salesforce pentest.

![[Salesforce_DevConsole_Location.png]]
## Query Editor
**Path:** Developer Console → Query Editor tab (bottom panel)

Used for running SOQL queries against the org's data and metadata. The "Use Tooling API" checkbox switches between the standard data API (for querying records) and the Tooling API (for querying metadata like Apex class definitions, field definitions, and compilation status).
## File Browser and Source Code Review
**Path:** Developer Console → File → Open

Opens any Apex class, trigger, Visualforce page, Lightning component, or static resource in the org. Use this to review custom code identified during enumeration. The Edit → Search in Files options allow text search within an open file or across all files in the org respectively.

> [!NOTE]
> Search in Files is the primary tool for enumerating `@AuraEnabled` methods, `@RestResource` annotations, `without sharing` declarations, and FLS enforcement gaps across the full codebase without needing the CLI.
## Execute Anonymous Window
**Path:** Developer Console → Debug → Open Execute Anonymous Window (or Ctrl+E)

Runs arbitrary Apex code in the context of the currently authenticated user. Check "Open Log" before executing so the debug log opens automatically. Use "Debug Only" filter in the log view to isolate `System.debug()` output from the noise.

Useful for schema inspection, data queries that are easier to express in Apex than SOQL, and behavioral confirmation of method behavior where Anonymous Apex invocation is possible.

> [!NOTE]
> Methods in managed package classes cannot be invoked from Anonymous Apex.

---
# Salesforce Object Query Language (SOQL)
Salesforce's read-only query language for its database. It is similar to SQL SELECT statements but with Salesforce-specific extensions for relationship traversal and security enforcement. It does not support INSERT, UPDATE, DELETE, or JOIN, which are handled differently in Apex (DML statements) and the Tooling API.

SOQL runs in the Query Editor in the Developer Console. Switch to Tooling API mode for metadata queries.

**General syntax:**
```sql
SELECT field1, field2 FROM ObjectApiName WHERE condition ORDER BY field LIMIT n
```

**Relationship traversal (parent to child):**
```sql
SELECT Id, Name, (SELECT Id, Subject FROM Cases) FROM Account
```

**Relationship traversal (child to parent):**
```sql
SELECT Id, Subject, Account.Name FROM Case
```
## Useful SOQL Queries for Enumeration
**Find all custom Apex classes and their validity:**
```sql
SELECT Id, Name, IsValid, ApiVersion, NamespacePrefix
FROM ApexClass
WHERE NamespacePrefix = null
ORDER BY Name
```

> [!NOTE]
> Requires Tooling API. `NamespacePrefix = null` filters to org-owned code only, excluding managed packages. `IsValid = false` means the class has a compile error and its methods will not execute.

**Find all Apex classes containing a specific annotation or pattern:**
Use *Search in Files* in the Developer Console for text patterns like `@AuraEnabled`, `@RestResource`, `without sharing`, `WITH SECURITY_ENFORCED`, `stripInaccessible`. SOQL cannot search inside the `Body` field of `ApexClass`.

**Find all custom REST resources:**
```sql
SELECT Id, Name, Body
FROM ApexClass
WHERE Name LIKE '%Resource%'
```

Then cross-reference with *Search in Files* for `@RestResource` to get the actual URL mappings.

**Get all fields on an object including namespace-prefixed ones:**
```sql
SELECT QualifiedApiName, Label, DataType
FROM FieldDefinition
WHERE EntityDefinition.QualifiedApiName = 'ObjectApiName'
ORDER BY Label
```

> [!NOTE]
> Requires Tooling API. Use `QualifiedApiName` (not `Label`) as the field name in subsequent SOQL queries. If a field is not queryable under its apparent name, run this to find the namespaced version.

**Schema describe via Anonymous Apex:**
> [!NOTE]
> This is more reliable than FieldDefinition for namespace issues.

```java
Map<String, Schema.SObjectField> fields = 
    Schema.SObjectType.ObjectApiName.fields.getMap();
for (String fieldName : fields.keySet()) {
    System.debug(fieldName + ' => ' + 
        fields.get(fieldName).getDescribe().getLabel());
}
```

**Check Apex class compile status:**
```sql
SELECT Id, Name, IsValid
FROM ApexClass
WHERE Name = 'ClassName'
```

> [!NOTE]
> Requires Tooling API.

**Find all profiles and their license types:**
```sql
SELECT Id, Name, UserLicense.Name, UserType
FROM Profile
ORDER BY Name
```

**Find which users are assigned to a specific profile:**
```sql
SELECT Id, Name, Username, IsActive, Profile.Name
FROM User
WHERE Profile.Name = 'ProfileNameHere'
AND IsActive = true
```

**Find permission set assignments for a user:**
```sql
SELECT PermissionSet.Name, PermissionSet.IsCustom
FROM PermissionSetAssignment
WHERE AssigneeId = 'userId'
```

**Find all active Experience Cloud sites:**
```sql
SELECT Id, Name, Status, UrlPathPrefix, GuestUserId
FROM Network
```

**Get the guest user ID for a site:**
```sql
SELECT Id, Name, Username, ProfileId
FROM User
WHERE Id = 'GuestUserIdFromNetworkQuery'
```

**Find all AccountContactRelation records for a user**
> [!NOTE]
> This is useful for understanding portal user scope.

```sql
SELECT AccountId, Account.Name, Account.RecordType.DeveloperName, ContactId
FROM AccountContactRelation
WHERE ContactId = 'contactIdHere'
```

**Find all ContentDocumentLink records for a Case (File Attachment Enumeration):**
```sql
SELECT ContentDocumentId, ContentDocument.Title, ContentDocument.OwnerId,
       ShareType, Visibility
FROM ContentDocumentLink
WHERE LinkedEntityId = 'caseIdHere'
```

**Find UJET Session records:**
```sql
SELECT Id, UJET__Contact__c, UJET__Case__c, UJET__Account__c
FROM UJET__UJET_Session__c
LIMIT 10
```

**Find Custom Metadata records:**
> [!NOTE]
> This is useful for understanding app configuration.

```sql
SELECT Id, DeveloperName, MasterLabel
FROM ObjectApiName__mdt
LIMIT 20
```

**Find Custom Labels:**
```sql
SELECT Id, Name, Value
FROM ExternalString
WHERE Name LIKE '%keyword%'
```

> [!NOTE]
> Requires Tooling API.

**Find all sharing rules for an object:**
```sql
SELECT Id, Name, SharingCriteriaRuleType, AccessLevel
FROM CaseSharingRule
```

> [!TIP]
> Replace `Case` with the object name.

**Find Apex triggers on an object:**
```sql
SELECT Id, Name, TableEnumOrId, Status, IsActive
FROM ApexTrigger
WHERE TableEnumOrId = 'Case'
```

> [!NOTE]
> Requires Tooling API.

**Find all installed packages:**
```sql
SELECT Id, SubscriberPackage.Name, SubscriberPackage.NamespacePrefix,
       SubscriberPackageVersion.MajorVersion
FROM InstalledSubscriberPackage
```

> [!NOTE]
> Requires Tooling API.

---
# Aura Endpoint Enumeration
The Aura framework handles all Lightning Experience and Experience Cloud front-end-to-server communication. All Aura action calls go to a single endpoint:
```
POST /s/sfsites/aura?r=<n>&aura.ApexAction.execute=1
```

The path prefix varies by site. For an Experience Cloud site at `/customerservice/s/` the endpoint is `/customerservice/s/sfsites/aura`. For the internal Lightning Experience it is `/aura`.

**Request Structure:**
The following has been URL-decoded for readability:
```json
message={
  "actions":[{
    "id":"1;a",
    "descriptor":"aura://ApexActionController/ACTION$execute",
    "callingDescriptor":"UNKNOWN",
    "params":{
      "namespace":"",
      "classname":"YourClassName",
      "method":"yourMethodName",
      "params":{
        "paramName":"paramValue"
      },
      "cacheable":false,
      "isContinuation":false
    }
  }]
}&aura.context={...}&aura.pageURI=/site/s/&aura.token=null
```

`aura.token=null` indicates a guest/unauthenticated request. An authenticated request carries a non-null token value copied from a captured authenticated session request.

**To Invoke a Specific Method:** Swap `classname` and `method` while keeping the rest of the structure identical to a captured working request. Copy `fwuid` and `loaded` values from any previously captured Aura response context
> [!NOTE]
> `fwuid` and `loaded` are framework version identifiers that must match the org's current deployment).

**To Test Guest Reachability:** Send the request with `aura.token=null` and no session cookies. A response of `"Guest user access is not allowed"` confirms the method is blocked for unauthenticated users. Any other response including an application-level error means the method executed.

**Aura Action Descriptors to Look for in Captured Traffic:**
- `apex://ClassName/ACTION$methodName`: Your custom `@AuraEnabled` method
- `aura://ApexActionController/ACTION$execute`: The standard wrapper for all custom Apex calls
- `serviceComponent://...`: Platform-owned components, generally out of scope

---
# File Handling and the Shepherd Servlet
Salesforce stores all files as `ContentDocument` records with associated `ContentVersion` records for each version. Files are served through the Shepherd servlet.

**Direct File Download:**
```
GET /sfc/servlet.shepherd/document/download/<ContentDocumentId>
```

**Specific Version Download:**
```
GET /sfc/servlet.shepherd/version/download/<ContentVersionId>
```

**Rendition Download (preview/thumbnail):**
```
GET /sfc/servlet.shepherd/document/renditionDownload?docId=<ContentDocumentId>&operationContext=CHATTER&contentId=<ContentVersionId>
```

> [!NOTE]
> The `operationContext` parameter influences how the file is served. Values include `CHATTER`, `SALESFORCE1`, `THUMB720BY480`, and others. Different contexts may return different `Content-Type` and `Content-Disposition` headers, which matters for XSS assessment of uploaded files.

**What to Test:**
Check `Content-Disposition` header on each serving path. A response of `Content-Disposition: attachment` forces download and prevents inline rendering. A response with `Content-Type: image/svg+xml` and no `Content-Disposition: attachment` means the browser will render the SVG inline, which is an XSS vector if the file content is attacker-controlled.

Test with a foreign `ContentDocumentId` (a file uploaded by a different user) to confirm whether the platform's retrieval ACL blocks cross-user file access. A denial response confirms the ACL is enforced. A successful response returning file content indicates an access control gap.

---
# URL Manipulation and Classic UI Access
The vforcesite host used by Experience Cloud exposes Salesforce's underlying infrastructure. Path manipulation can sometimes surface unintended interfaces.

**Classic UI via Extra Slash:**
Inserting an extra forward slash before the `lightning` path segment on the vforcesite host:
```
/vendorportalvforcesite//lightning
```

Can cause the Salesforce Classic UI to render within the authenticated portal session rather than the expected community interface. The rendered frame runs as the current user's identity.

**Standard Object URL Patterns (Classic):**
```
/500/o  - Cases list view
/003/o  - Contacts list view
/001/o  - Accounts list view
/<keyprefix>/o  - any object list view
/<keyprefix>/<recordId>/e  - edit a specific record
```

> [!NOTE]
> Key prefixes are the first three characters of any Salesforce record ID. Testing these patterns after surfacing Classic UI chrome can reveal whether the user's profile blocks access to those object views or allows enumeration.

**Record ID Structure:**
Salesforce record IDs are either 15-character (case-sensitive) or 18-character (case-insensitive) strings. The first three characters are the key prefix identifying the object type. For example Cases start with `500`, Contacts with `003`, Accounts with `001`.

---
# Upload Configuration Endpoint
The file upload configuration for Experience Cloud and LWR sites is served from:
```
GET /webruntime/api/services/data/vXX.X/connect/file/upload/config?language=en-US&asGuest=false&htmlEncode=false
```

The response JSON reveals:
- `allowedFileExtensions`: The server-side extension allowlist. **An empty array means no restriction.**
- `maxFileSize`: The configured file size limit in bytes. **Zero means no explicit limit.**
- `supportedFileTypes`: MIME types the platform will accept.

This endpoint is one of the first things to capture when testing a file upload flow. An empty `allowedFileExtensions` array is a hardening gap regardless of other mitigating controls.

The upload itself posts to the following on the vforcesite host, with the file content in a multipart form body:
```
POST /chatter/handlers/file/body
```

---
# Nebula Logger Enumeration
If the org uses Nebula Logger, log records are stored as queryable Salesforce data rather than transient debug logs.

**Check if Logging is Active:**
```sql
SELECT Id, StartTime__c, LoggedBy__c, TransactionId__c
FROM Log__c
ORDER BY StartTime__c DESC
LIMIT 10
```

**Read Log Entry Messages:**
```sql
SELECT Id, Message__c, ExceptionMessage__c, LoggingLevel__c,
       OriginLocation__c, Timestamp__c
FROM LogEntry__c
ORDER BY Timestamp__c DESC
LIMIT 20
```

**Search for Sensitive Content in Logs:**
```sql
SELECT Id, Message__c, Timestamp__c
FROM LogEntry__c
WHERE Message__c LIKE '%card%'
OR Message__c LIKE '%GUID%'
OR Message__c LIKE '%membershipNumber%'
LIMIT 20
```

> [!NOTE]
> If `LogEntry__c` records are readable by the current user's profile and the code writes sensitive field values to Logger messages, the log records become a secondary data store that bypasses FLS restrictions on the source objects.

---
# Anonymous Apex for Schema Inspection
When field or class names do not resolve as expected due to namespace prefixes, Anonymous Apex schema describe is the most reliable enumeration method.

**Enumerate all Fields on an Object:**
```java
Map<String, Schema.SObjectField> fields = 
    Schema.SObjectType.UJET__UJET_Session__c.fields.getMap();
for (String fieldName : fields.keySet()) {
    System.debug(fieldName + ' => ' + 
        fields.get(fieldName).getDescribe().getLabel());
}
```

**Filter by Keyword:**
```java
Map<String, Schema.SObjectField> fields = 
    Schema.SObjectType.Contact.fields.getMap();
for (String fieldName : fields.keySet()) {
    if (fieldName.containsIgnoreCase('guid') || 
        fieldName.containsIgnoreCase('member')) {
        System.debug(fieldName + ' => ' + 
            fields.get(fieldName).getDescribe().getLabel());
    }
}
```

**Get all Objects in the Org:**
```java
Map<String, Schema.SObjectType> globalDescribe = Schema.getGlobalDescribe();
for (String objectName : globalDescribe.keySet()) {
    System.debug(objectName);
}
```

**Check if a Class is Resolvable (namespace detection):**
```java
Type t = Type.forName('ClassName');
System.debug(t);
```

> [!NOTE]
> Returns null if the class is not resolvable under that name, indicating it may require a namespace prefix or does not exist.

**Two-argument `forName` with Explicit Namespace:**
```java
Type t = Type.forName('namespacePrefixHere', 'ClassName');
System.debug(t);
```

---
# Visualforce-Specific Notes
Visualforce is Salesforce's older server-side page rendering framework, distinct from Lightning Web Components and Aura. Pages are served from:
```
/apex/PageName
```
or via the vforcesite host on Experience Cloud sites.

**Expression Language:** `{! }` delimiters evaluate server-side expressions before the page is sent to the browser. `{!propertyName}` calls the getter on the controller property. Default encoding is HTML encoding, which is appropriate for HTML context but not JavaScript context.

**Encoding Functions:**
- `{!HTMLENCODE(value)}`: HTML encoding, safe for HTML element content
- `{!JSENCODE(value)}`: JavaScript encoding, safe for JS string literals inside `<script>` blocks
- `{!JSINHTMLENCODE(value)}`: Both HTML and JS encoding, for values in HTML event handlers
- `{!URLENCODE(value)}`: URL encoding, for values in query string parameters
- `{!URLFOR(...)}`: Constructs safe URLs

> [!TIP]
> The most common Visualforce XSS pattern is using `{!value}` inside a `<script>` block where `{!JSENCODE(value)}` is required. Static analysis tools will flag this pattern correctly at the sink level but cannot always determine whether the value is user-controlled. Always trace the data flow back to the source before confirming exploitability.

---
### Key Object API Names Reference

|Object|API Name|Notes|
|---|---|---|
|Case|`Case`|Standard|
|Account|`Account`|Standard|
|Contact|`Contact`|Standard|
|User|`User`|Standard|
|Group|`Group`|Queues and public groups|
|GroupMember|`GroupMember`|Queue/group membership|
|AccountContactRelation|`AccountContactRelation`|Many-to-many account-contact|
|ContentDocument|`ContentDocument`|File records|
|ContentVersion|`ContentVersion`|File versions|
|ContentDocumentLink|`ContentDocumentLink`|File-to-record associations|
|Network|`Network`|Experience Cloud sites|
|ApexClass|`ApexClass`|Apex source (Tooling API)|
|ApexTrigger|`ApexTrigger`|Triggers (Tooling API)|
|FieldDefinition|`FieldDefinition`|Field metadata (Tooling API)|
|Log__c|`Log__c`|Nebula Logger parent|
|LogEntry__c|`LogEntry__c`|Nebula Logger entries|
|UJET Session|`UJET__UJET_Session__c`|UJET managed package|