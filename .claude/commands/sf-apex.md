Generate a CS-prefixed Apex class and companion test class from a solution design.

**Input:** $ARGUMENTS — design file path followed by the class name
Example: designs/2026-04-22-account-hierarchy.md CSAccountService

## Instructions

Parse $ARGUMENTS to extract:
- Design file path (first token)
- Apex class name (second token — must start with CS)

Use the dev-agent to generate the Apex artifacts:

1. Read the design file at the path provided
2. Read `cache/org-context.json` to verify no naming conflict with existing classes
3. If the class name does not start with CS, stop and output: "⚠️ Class name must start with CS prefix. Example: CSAccountService"
4. Generate the Apex class following all dev-agent standards:
   - `with sharing` modifier
   - No SOQL/DML in loops
   - Bulkified methods accepting List<SObject>
   - Header comment block with date and standard/config alternative considered
5. Generate the companion test class targeting 75%+ coverage:
   - @TestSetup for shared data
   - Happy path test method
   - Bulk/edge case test method (empty list, 200 records)
6. Generate .cls-meta.xml for both files
7. Save all files to `force-app/main/default/classes/`

After saving, output:
"Generated:
- force-app/main/default/classes/[CSClassName].cls
- force-app/main/default/classes/[CSClassName].cls-meta.xml
- force-app/main/default/classes/[CSClassNameTest].cls
- force-app/main/default/classes/[CSClassNameTest].cls-meta.xml"
