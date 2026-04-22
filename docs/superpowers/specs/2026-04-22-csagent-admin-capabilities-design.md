# CSAgent Admin Capabilities — Design Spec

**Date:** 2026-04-22
**Author:** CSAgent Brainstorming Session
**Status:** Approved for implementation

---

## Goal

Extend CSAgent with complete Salesforce admin metadata generation capability. Fill the gap between solution design and deployable org configuration by adding three new slash commands and extending dev-agent with XML templates and Setup instruction formats for all standard Salesforce admin activities.

---

## Architecture

Three new commands, all delegating to the extended dev-agent. No new agents. No changes to story-agent or design-agent. No changes to settings.json.

```
/sf-object          ──► dev-agent (Object/Field XML section)      ──► force-app/main/default/objects/
/sf-permission-set  ──► dev-agent (Permission Set XML section)    ──► force-app/main/default/permissionsets/
/sf-admin-setup     ──► dev-agent (Setup Instructions section)    ──► designs/setup-instructions/
```

### New Output Locations

| Path | Contents |
|---|---|
| `force-app/main/default/objects/CS_Object__c/` | Object XML, fields, record types, validation rules, compact layouts, list views |
| `force-app/main/default/permissionsets/` | Permission set XMLs |
| `force-app/main/default/permissionsetgroups/` | Permission set group XMLs (when design has multiple roles) |
| `force-app/main/default/sharingRules/` | Sharing rule XMLs (when design specifies sharing strategy) |
| `designs/setup-instructions/` | Human-readable Setup UI instructions for complex admin activities |

---

## Command 1: `/sf-object`

### Invocation
```
/sf-object designs/YYYY-MM-DD-<name>.md CS_ObjectName__c
```

### Arguments
- Arg 1: Design file path
- Arg 2: Object API name — must start with `CS_` and end with `__c`

### CS Prefix Enforcement
If object name does not start with `CS_`, hard stop:
> "⚠️ Object name must start with CS_ prefix. Example: CS_ServiceRequest__c"

All generated fields must start with `CS_`. Dev-agent rejects any field definition from the design that does not use the CS_ prefix and outputs a warning listing the non-compliant fields.

### Generated File Structure

```
force-app/main/default/objects/CS_ObjectName__c/
├── CS_ObjectName__c.object-meta.xml
├── fields/
│   └── CS_FieldName__c.field-meta.xml        (one file per field)
├── recordTypes/
│   └── CS_RecordTypeName.recordType-meta.xml  (if design has record types)
├── validationRules/
│   └── CS_RuleName.validationRule-meta.xml    (one file per validation rule)
├── compactLayouts/
│   └── CS_CompactLayoutName.compactLayout-meta.xml  (if specified in design)
└── listViews/
    └── CS_ListViewName.listView-meta.xml      (if specified in design)
```

### Supported Field Types

Dev-agent generates correct `field-meta.xml` for all 18 standard field types:

| Type | Notes |
|---|---|
| Text | Length required |
| Number | Precision + scale required |
| Currency | Precision + scale required |
| Percent | Precision + scale required |
| Date | No extra config |
| DateTime | No extra config |
| Checkbox | Default value required |
| Picklist | Values list required |
| Multi-Select Picklist | Values list required |
| Long Text Area | Length required (max 131072) |
| Rich Text Area | Length required |
| Formula | Return type + formula required |
| Lookup | Referenced object required |
| Master-Detail | Referenced object required; warns cascade-delete risk |
| Email | No extra config |
| Phone | No extra config |
| URL | No extra config |
| Auto Number | Display format required |

### What Generates vs What Doesn't

**Generates:** All metadata listed above in Salesforce DX source format.

**Does NOT generate:** Page layouts (too complex — covered by `/sf-admin-setup`). OWD settings (Setup UI only).

---

## Command 2: `/sf-permission-set`

### Invocation
```
/sf-permission-set designs/YYYY-MM-DD-<name>.md CS_PermSetName
```

### Arguments
- Arg 1: Design file path
- Arg 2: Permission set name — must start with `CS_`

### CS Prefix Enforcement
If permission set name does not start with `CS_`, hard stop:
> "⚠️ Permission set name must start with CS_ prefix. Example: CS_ServiceAgentPermissions"

### Generated Files

**Single role in design:**
```
force-app/main/default/permissionsets/
└── CS_PermSetName.permissionset-meta.xml
```

**Multiple roles in design** (e.g., Service Agent + Service Manager):
```
force-app/main/default/permissionsets/
├── CS_ServiceAgentPermissions.permissionset-meta.xml
└── CS_ServiceManagerPermissions.permissionset-meta.xml
force-app/main/default/permissionsetgroups/
└── CS_ServicePermissions.permissionsetgroup-meta.xml
```

### Permission Set XML Contents

For each CS_ object in the design:
- Object permissions: Read ✅, Create ✅, Edit ✅, Delete ❌, ViewAll ❌, ModifyAll ❌ (default — dev-agent comments where to adjust)
- Field-level security: Read ✅, Edit ✅ for all CS_ fields (default)
- Apex class access for any CS classes referenced in the design

### Sharing Rules

If design specifies a sharing strategy, generates:
```
force-app/main/default/sharingRules/CS_ObjectName__c.sharingRules-meta.xml
```

If design does not specify sharing strategy, dev-agent outputs:
> "⚠️ OWD not specified in design — defaulting to Private. Update OWD manually: Setup → Sharing Settings → CS_ObjectName__c."

---

## Command 3: `/sf-admin-setup`

### Invocation
```
/sf-admin-setup designs/YYYY-MM-DD-<name>.md
```

### Arguments
- Arg 1: Design file path only — no second argument. Dev-agent reads the design and generates instructions for all complex admin activities it finds.

### Generated File
```
designs/setup-instructions/YYYY-MM-DD-<name>-setup.md
```

### Covered Activities

Dev-agent scans the design for any mention of the following and generates the corresponding Setup instructions:

| Activity | Setup Path | Instruction Format |
|---|---|---|
| Approval Process | Setup → Process Automation → Approval Processes | Entry criteria, approver hierarchy, approval/rejection actions, email templates |
| Page Layout | Setup → Object Manager → [Object] → Page Layouts | Field sections, related lists, buttons, field order |
| Custom Report Type | Setup → Report Types | Primary object, related objects, fields to expose, label |
| Email Alert | Setup → Process Automation → Email Alerts | Template, recipients, from address, trigger |
| Custom Tab | Setup → Tabs | Object, tab style, app assignment |
| Custom App | Setup → App Manager | App name, nav items, logo, profiles |
| Assignment Rule | Setup → [Object] Assignment Rules | Criteria, assignee, order of evaluation |
| Escalation Rule | Setup → [Object] Escalation Rules | Time trigger, criteria, escalation action |
| Custom Label | Setup → Custom Labels | Name, value, categories |
| Custom Setting | Setup → Custom Settings | Type (List/Hierarchy), fields |
| Custom Metadata Type | Setup → Custom Metadata Types | Fields, records — plus note to use `/sf-object` for XML |
| Email Template | Setup → Email Templates | Type, subject, body structure |

### Instruction Format (per activity)

Each activity section follows this structure:

```markdown
## [Activity Type]: [CS_Name]

**Setup Path:** Setup → [exact navigation path]

### Steps
1. [Exact UI action with field names and values]
2. [Next step]
...

### Key Values from Design
| Field | Value |
|---|---|
| [field name] | [value from design] |

### Notes
[Any warnings, gotchas, or post-setup verification steps]
```

---

## dev-agent Extensions

Four new sections appended to `.claude/agents/dev-agent.md` after the existing Flow section:

### New Section 1: Custom Object & Field Standards
- XML templates for `object-meta.xml` (deploymentStatus, label, pluralLabel, nameField, sharingModel)
- XML templates for all 18 field types as `field-meta.xml`
- XML template for `recordType-meta.xml`
- XML template for `validationRule-meta.xml` (active, errorConditionFormula, errorMessage, errorDisplayField)
- XML template for `compactLayout-meta.xml`
- XML template for `listView-meta.xml`

### New Section 2: Permission Set Standards
- XML template for `permissionset-meta.xml` with objectPermissions, fieldPermissions, classAccesses blocks
- Default permission values and comments explaining where to tighten
- XML template for `permissionsetgroup-meta.xml`

### New Section 3: Sharing Rule Standards
- XML template for `sharingRules-meta.xml` (criteria-based and owner-based variants)

### New Section 4: Admin Setup Instructions Standards
- Structured template format for each of the 12 complex admin activity types
- How to scan the design file to identify which activities are present
- Output file naming convention: `designs/setup-instructions/YYYY-MM-DD-<name>-setup.md`

---

## CLAUDE.md Updates

### Command Table — 3 rows added

| Command | Usage | Output |
|---|---|---|
| /sf-object | /sf-object designs/\<file\>.md CS_ObjectName__c | force-app/.../objects/ |
| /sf-permission-set | /sf-permission-set designs/\<file\>.md CS_PermSetName | force-app/.../permissionsets/ |
| /sf-admin-setup | /sf-admin-setup designs/\<file\>.md | designs/setup-instructions/ |

### Naming Conventions Table — 1 row added

| Component | Convention | Example |
|---|---|---|
| Permission Set | CS_ + PascalCase | CS_ServiceAgentPermissions |

### Output Locations — 3 entries added

- Object metadata → `force-app/main/default/objects/`
- Permission sets → `force-app/main/default/permissionsets/`
- Admin setup instructions → `designs/setup-instructions/`

---

## Files Changed

| File | Change Type |
|---|---|
| `.claude/commands/sf-object.md` | Create |
| `.claude/commands/sf-permission-set.md` | Create |
| `.claude/commands/sf-admin-setup.md` | Create |
| `.claude/agents/dev-agent.md` | Extend (4 new sections appended) |
| `CLAUDE.md` | Update (command table, naming table, output locations) |

---

## Out of Scope

- Profiles (use permission sets instead — Salesforce best practice)
- Connected Apps (contain sensitive credentials — manual Setup only)
- Letterheads (visual design — manual Setup only)
- Lightning App Builder pages (complex JSON, not reliably source-trackable)
- Deploy command (out of scope for this spec — `sf project deploy start` is a one-liner users can run directly)
