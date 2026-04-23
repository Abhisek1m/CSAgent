Convert a plain English business requirement into structured Salesforce user stories.

**Input:** $ARGUMENTS — the plain English business requirement

## Instructions

Use the story-agent to process this requirement: $ARGUMENTS

The story-agent will:
1. Evaluate whether the requirement maps to any standard Salesforce features
2. Flag standard features with ⚠️ STANDARD FEATURE before writing custom stories
3. Write structured user stories in As a / I want / So that format
4. Include acceptance criteria, priority, story points, and metadata impact
5. Save the output to `stories/YYYY-MM-DD-<kebab-name>.md` using today's date

After the file is saved, output exactly this block (replacing [filename] with the actual file name):

---
## Story saved: stories/[filename]

Review the stories file, then kick off the design:

**Next step →** `/sf-design stories/[filename]`

---
