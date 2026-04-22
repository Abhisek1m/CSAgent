Generate step-by-step Salesforce Setup UI instructions for complex admin activities found in a solution design.

**Input:** $ARGUMENTS — design file path only
Example: designs/2026-04-22-satisfaction-score.md

## Instructions

Use the dev-agent to scan the design and generate setup instructions:

1. Read the design file at the path provided
2. Scan the design for any mention of these activities: Approval Process, Page Layout, Custom Report Type, Email Alert, Custom Tab, Custom App, Assignment Rule, Escalation Rule, Custom Label, Custom Setting, Custom Metadata Type, Email Template
3. For each activity found, generate a structured instruction block using the Admin Setup Instructions Standards templates in dev-agent
4. Order the instruction blocks by implementation sequence — objects and fields first, then automation, then UI, then reporting
5. If no covered activities are found in the design, output: "No complex admin activities found in this design. Use /sf-object for metadata and /sf-permission-set for permissions."
6. Save the complete output to `designs/setup-instructions/YYYY-MM-DD-<feature-name>-setup.md` using today's date and deriving the feature name from the design file name by stripping the date prefix and `.md` extension (e.g., `designs/2026-04-22-customer-satisfaction-score.md` → feature name is `customer-satisfaction-score`)

After saving, output:
"Setup instructions saved to designs/setup-instructions/[filename].

Complete these manual Setup steps before or alongside deploying the metadata files from /sf-object and /sf-permission-set."
