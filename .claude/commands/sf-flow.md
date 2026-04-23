Generate a CS-prefixed Flow description and metadata XML from a solution design.

**Input:** $ARGUMENTS — design file path followed by the Flow name
Example: designs/2026-04-22-account-hierarchy.md CS_AccountValidation

## Instructions

Parse $ARGUMENTS to extract:
- Design file path (first token)
- Flow name (second token — must start with CS_)

Use the dev-agent to generate the Flow:

1. Read the design file at the path provided
2. If the Flow name does not start with CS_, stop and output: "⚠️ Flow name must start with CS_ prefix. Example: CS_AccountValidation"
3. First output a plain English step-by-step description of the flow logic based on the design
4. Then generate the Flow metadata XML:
   - apiVersion 62.0
   - Correct processType (AutoLaunchedFlow for record-triggered, Flow for screen flows)
   - Start element with trigger object and conditions
   - Decision, Assignment, and Action elements as needed
   - Status: Active
5. Save to `force-app/main/default/flows/[CS_FlowName].flow-meta.xml`

After saving, output exactly this block (replacing placeholders with actuals):

---
## Generated: Flow metadata

- `force-app/main/default/flows/[CS_FlowName].flow-meta.xml`

> Review the Flow XML in VS Code before deploying. Complex logic may need adjustments in the visual Flow Builder (Setup → Flows → [Flow name]).

---
## Full deployment order

Deploy in this sequence — each component depends on the previous:

```bash
# 1. Object and fields (always first)
sf project deploy start --source-dir force-app/main/default/objects/[CS_ObjectName__c] --target-org <alias>

# 2. Permission set
sf project deploy start --source-dir force-app/main/default/permissionsets --target-org <alias>

# 3. Flow (depends on object existing in org)
sf project deploy start --source-dir force-app/main/default/flows --target-org <alias>
```

---
## After deployment — manual steps

Complete the manual Setup UI steps documented in:
`designs/setup-instructions/[date]-[feature-name]-setup.md`

These cover activities that cannot be deployed via metadata (page layouts, report types, OWD sharing settings, etc.).

---
