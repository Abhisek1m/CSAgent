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

After saving, output:
"Generated:
- force-app/main/default/flows/[CS_FlowName].flow-meta.xml

Note: Review the Flow XML in VS Code Salesforce extension before deploying. 
Complex Flow logic may require adjustments in the visual Flow Builder."
