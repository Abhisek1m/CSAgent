Generate metadata and/or Setup UI instructions for admin activities found in a solution design.

**Input:** $ARGUMENTS — design file path only
Example: designs/2026-04-22-satisfaction-score.md

## Instructions

Use the dev-agent to scan the design and generate output:

1. Read the design file at the path provided
2. Scan for any mention of these activities and classify each as DEPLOYABLE or MANUAL:

   **DEPLOYABLE via SFDX — generate metadata XML:**
   - Page Layout → `Layout` metadata, `force-app/main/default/layouts/`
   - Custom Report Type → `ReportType` metadata, `force-app/main/default/reportTypes/`
   - Email Template → `EmailTemplate` metadata, `force-app/main/default/email/CS_EmailTemplates/`

   **MANUAL — Setup UI only (cannot be deployed via sf project deploy):**
   - Escalation Rule (not a deployable metadata type)
   - Approval Process (keep manual — too complex and risky to auto-deploy)
   - Assignment Rule (not a deployable metadata type)
   - Email Alert (keep manual — requires workflow context)
   - Custom Tab (keep manual — App Builder is simpler)
   - Custom App (keep manual — App Builder is simpler)
   - Custom Label (keep manual — simple enough in Setup)
   - Custom Setting (keep manual — data values not deployable)
   - Custom Metadata Type records (type is deployable, records are data)
   - Reports / Dashboards (not deployable via sf project deploy)
   - Business Hours (not a deployable metadata type)
   - OWD / Sharing Settings (keep manual — triggers org-wide sharing recalculation; risky to deploy)

3. For DEPLOYABLE activities: generate the metadata XML file using templates from the Deployable UI Metadata Standards section in dev-agent. Save to the correct `force-app/` path.

4. For MANUAL activities: generate a structured step-by-step instruction block using the Admin Setup Instructions Standards templates in dev-agent.

5. Order output by implementation sequence: objects/fields → layouts/templates → automation → reporting

6. If no covered activities are found, output: "No complex admin activities found in this design. Use /sf-object for metadata and /sf-permission-set for permissions."

7. Save the complete setup instructions (manual steps only) to `designs/setup-instructions/YYYY-MM-DD-<feature-name>-setup.md` using today's date, deriving the feature name from the design file name by stripping the date prefix and `.md` extension.

After completing, output exactly this block (replacing placeholders with actuals):

---
## Admin setup complete

### Metadata generated (deploy with SFDX)
[list each generated file with its deploy command, or "None" if all activities were manual]

### Manual Setup UI steps
`designs/setup-instructions/[filename]`

These [N] activities **cannot be deployed** and must be completed manually:
[numbered list of manual activity types found]

**Why OWD stays manual:** OWD changes trigger an org-wide sharing recalculation that can affect all records. Set it manually in Setup → Sharing Settings → [CS_ObjectName__c] to control timing.

---
