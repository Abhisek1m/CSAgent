Convert a user story file into a Salesforce solution design.

**Input:** $ARGUMENTS — path to the story file (e.g., stories/2026-04-22-account-hierarchy.md)

## Instructions

Use the design-agent to process the story file at: $ARGUMENTS

The design-agent will:
1. Read the story file at the path provided
2. Read `cache/org-context.json` to check existing org metadata
3. Apply the Golden Rule (Standard → Config → Code) to every requirement in the stories
4. Produce a complete solution design covering:
   - Data model (standard objects reused + new CS_ custom objects with full field definitions)
   - Relationship diagram
   - Automation strategy (Flow preferred over Apex)
   - Component breakdown
   - Implementation order
5. Include a **Why Code?** justification for any Apex or LWC that is recommended
6. Save to `designs/YYYY-MM-DD-<feature-name>.md` using today's date

After the file is saved, output exactly this block (replacing [filename] and component names with actuals from the design):

---
## Design saved: designs/[filename]

Review the design file, then generate metadata in this order:

### Step 1 — Object & Fields
```
/sf-object designs/[filename] CS_ObjectName__c
```
Run once per custom object in the design.

### Step 2 — Permissions
```
/sf-permission-set designs/[filename] CS_PermSetName
```

### Step 3 — Automation
```
/sf-flow designs/[filename] CS_FlowName
```
Use `/sf-apex designs/[filename] CSClassName` or `/sf-lwc designs/[filename] csComponentName` if the design requires Apex or LWC.

### Step 4 — Manual Admin Setup
```
/sf-admin-setup designs/[filename]
```
Generates step-by-step Setup UI instructions for page layouts, report types, and other admin-only activities.

---
