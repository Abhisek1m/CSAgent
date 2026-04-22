Generate a CS-prefixed Permission Set (and optionally Permission Set Group and Sharing Rules) from a solution design.

**Input:** $ARGUMENTS — design file path followed by the permission set name
Example: designs/2026-04-22-satisfaction-score.md CS_ServiceAgentPermissions

## Instructions

Parse $ARGUMENTS to extract:
- Design file path (first token)
- Permission set name (second token — must start with CS_)

Use the dev-agent to generate the permission set metadata:

1. Read the design file at the path provided
2. If the permission set name does not start with CS_, stop and output: "⚠️ Permission set name must start with CS_ prefix. Example: CS_ServiceAgentPermissions"
3. Generate `CS_PermSetName.permissionset-meta.xml` with:
   - Object permissions (Read, Create, Edit by default) for all CS_ objects in the design
   - Field-level security (Read, Edit by default) for all CS_ fields in the design
   - Apex class access for any CS classes referenced in the design
   - Dev-agent comments on each block indicating where to restrict permissions
4. If the design describes different permission requirements for different user types or personas (e.g., Service Agent vs. Service Manager), generate one permission set per user type AND a `CS_[Feature]Permissions.permissionsetgroup-meta.xml` bundling them — this is about functional personas in the design, not Salesforce Role Hierarchy objects
5. If the design specifies a sharing strategy, generate `CS_ObjectName__c.sharingRules-meta.xml` using the appropriate template (criteria-based or owner-based) from dev-agent
6. If no sharing strategy is mentioned in the design, output: "⚠️ OWD not specified in design — defaulting to Private. Update OWD manually: Setup → Sharing Settings → [CS_ObjectName__c]."
7. Save to the correct directories per dev-agent output paths

After saving, output:
"Generated:
- force-app/main/default/permissionsets/[CS_PermSetName].permissionset-meta.xml
[list group file and sharing rule file if generated]

Deploy with: sf project deploy start --source-dir force-app/main/default/permissionsets"
