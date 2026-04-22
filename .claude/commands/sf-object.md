Generate CS-prefixed Salesforce DX metadata files for a custom object, its fields, record types, validation rules, compact layouts, and list views from a solution design.

**Input:** $ARGUMENTS — design file path followed by the object API name
Example: designs/2026-04-22-satisfaction-score.md CS_SatisfactionScore__c

## Instructions

Parse $ARGUMENTS to extract:
- Design file path (first token)
- Object API name (second token — must start with CS_ and end with __c)

Use the dev-agent to generate the object metadata:

1. Read the design file at the path provided
2. Read `cache/org-context.json` to verify no naming conflict with existing custom objects
3. If the object name does not start with CS_ or does not end with __c, stop and output: "⚠️ Object name must start with CS_ prefix and end with __c. Example: CS_ServiceRequest__c"
4. If any field in the design does not start with CS_, output a warning listing the non-compliant fields and skip those fields — do not generate field files for them
5. Generate `CS_ObjectName__c.object-meta.xml` following the Custom Object & Field Standards in dev-agent
6. Generate one `CS_FieldName__c.field-meta.xml` per field defined in the design, using the correct field type template from dev-agent
7. Generate `CS_RecordTypeName.recordType-meta.xml` for each record type in the design (skip if none)
8. Generate `CS_RuleName.validationRule-meta.xml` for each validation rule in the design (skip if none)
9. Generate `CS_CompactLayoutName.compactLayout-meta.xml` if the design specifies a compact layout (skip if none)
10. Generate `CS_ListViewName.listView-meta.xml` for each list view in the design (skip if none)
11. Create subdirectories as needed and save all files under `force-app/main/default/objects/CS_ObjectName__c/`

After saving, output:
"Generated:
- force-app/main/default/objects/[CS_ObjectName__c]/[CS_ObjectName__c].object-meta.xml
- force-app/main/default/objects/[CS_ObjectName__c]/fields/ ([N] field files)
[list validation rules, record types, compact layouts, list views if generated]

Deploy with: sf project deploy start --source-dir force-app/main/default/objects/[CS_ObjectName__c]"
