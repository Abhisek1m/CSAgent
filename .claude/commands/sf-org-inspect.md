Query the connected Salesforce org and populate the org context cache.

## Steps

1. Run `sf org display --json` to get org info (alias, username, org ID)
2. Run `sf sobject list --sobject-type custom --json` to get custom object API names
3. Run `sf data query --query "SELECT Id, Name, Status FROM ApexClass ORDER BY Name" --json` to get Apex classes
4. Run `sf data query --query "SELECT Id, Name, Status FROM ApexTrigger ORDER BY Name" --json` to get triggers
5. Run `sf data query --query "SELECT Id, SubscriberPackage.Name FROM InstalledSubscriberPackage" --json` to get installed packages

Parse the results and write to `cache/org-context.json` in this exact structure:
```json
{
  "orgAlias": "<alias from sf org display>",
  "orgId": "<orgId from sf org display>",
  "username": "<username from sf org display>",
  "apiVersion": "62.0",
  "lastRefreshed": "<ISO timestamp now>",
  "customObjects": ["<list of custom object API names>"],
  "customFields": {},
  "apexClasses": ["<list of Apex class names>"],
  "apexTriggers": ["<list of trigger names>"],
  "lwcComponents": [],
  "installedPackages": ["<list of package names>"]
}
```

Also write to `cache/org-context-meta.json`:
```json
{
  "lastRefreshed": "<ISO timestamp now>",
  "orgAlias": "<alias>",
  "refreshedBy": "sf-org-inspect",
  "cacheVersion": "1.0"
}
```

After writing, confirm: "Org context cache populated. Found [N] custom objects, [N] Apex classes, [N] triggers."

If any sf command fails (org not connected), output:
"⚠️ Could not connect to org. Run `sf org login web` to authenticate, then retry /sf-org-inspect."
