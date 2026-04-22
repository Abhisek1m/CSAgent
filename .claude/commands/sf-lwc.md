Generate a CS-prefixed Lightning Web Component from a solution design.

**Input:** $ARGUMENTS — design file path followed by the component name
Example: designs/2026-04-22-account-hierarchy.md csAccountHierarchy

## Instructions

Parse $ARGUMENTS to extract:
- Design file path (first token)
- Component name (second token — must start with cs in camelCase)

Use the dev-agent to generate the LWC:

1. Read the design file at the path provided
2. Read `cache/org-context.json` to verify no naming conflict with existing LWC components
3. If the component name does not start with cs (lowercase), stop and output: "⚠️ LWC component name must start with cs (lowercase). Example: csAccountHierarchy"
4. Create the component folder: `force-app/main/default/lwc/[csComponentName]/`
5. Generate [csComponentName].html with:
   - Loading spinner state
   - Error display state
   - Data display state using lwc:if directives
   - lightning-card wrapper with appropriate icon
6. Generate [csComponentName].js with:
   - @wire adapter for data fetching
   - @api recordId if record context is needed
   - Error and loading state handlers
   - Header comment block
7. Generate [csComponentName].js-meta.xml with apiVersion 62.0 and appropriate targets

After saving, output:
"Generated:
- force-app/main/default/lwc/[csComponentName]/[csComponentName].html
- force-app/main/default/lwc/[csComponentName]/[csComponentName].js
- force-app/main/default/lwc/[csComponentName]/[csComponentName].js-meta.xml"
