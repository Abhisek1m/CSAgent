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

After the file is saved, output:
"Design saved to designs/[filename]. Review it, then run one of:
- /sf-apex designs/[filename] CSClassName
- /sf-lwc designs/[filename] csComponentName
- /sf-flow designs/[filename] CS_FlowName"
