Re-run the org inspection and overwrite the existing org context cache with fresh data.

This is identical to /sf-org-inspect but always overwrites the cache even if it already has content.
Use this command when:
- You deployed new custom objects or fields to the org
- You installed a new managed package
- You added new Apex classes or triggers manually

Follow the exact same steps as /sf-org-inspect:
1. Run `sf org display --json`
2. Run `sf sobject list --sobject-type custom --json`
3. Run `sf data query --query "SELECT Id, Name, Status FROM ApexClass ORDER BY Name" --json`
4. Run `sf data query --query "SELECT Id, Name, Status FROM ApexTrigger ORDER BY Name" --json`
5. Run `sf data query --query "SELECT Id, SubscriberPackage.Name FROM InstalledSubscriberPackage" --json`

Overwrite `cache/org-context.json` and `cache/org-context-meta.json` with fresh results.

Confirm: "Cache refreshed at [timestamp]. Found [N] custom objects, [N] Apex classes."
