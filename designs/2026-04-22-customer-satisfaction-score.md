# Solution Design: Customer Satisfaction Score Tracking

**Date:** 2026-04-22
**Story File:** stories/2026-04-22-customer-satisfaction-score.md
**Architect:** CSAgent Design Agent

> **Org Cache:** Empty (org not connected at design time). No naming conflict check was possible. Run `/sf-org-inspect` before deploying to verify no conflicts with existing metadata.

---

## Executive Summary

This solution adds a lightweight CSAT tracking capability by creating one custom object (`CS_SatisfactionScore__c`) linked to the standard Case object. A Record-Triggered Flow auto-creates a score record when a Case is closed, and standard Reports + Dashboards surface the trend data. No Apex or LWC is required — the entire solution is declarative.

---

## Golden Rule Assessment

| Requirement | Standard Feature? | Config Sufficient? | Code Required? |
|---|---|---|---|
| Store a 1–5 rating linked to a Case | Salesforce Surveys (Enterprise+) — proceed with custom if unavailable | Custom object + fields + Validation Rule | No |
| Auto-create score record on Case closure | No standard feature | Record-Triggered Flow (after update, Status → Closed) | No |
| Prevent duplicate scores per Case | No standard feature | Flow Get Records + Decision element | No |
| Enforce rating range 1–5 | No standard feature | Validation Rule on CS_Rating__c | No |
| View scores on Case record | No standard feature | Related List on Case Page Layout (admin config) | No |
| Reports by agent, date range | Standard Reports + Custom Report Type | Standard feature — admin setup only | No |
| CSAT trend dashboard | Standard Dashboards | Standard feature — admin setup only | No |

**Verdict: Zero custom code required. Full declarative solution.**

---

## Data Model

### Standard Objects Used

| Object | Why Used | Key Fields Used |
|---|---|---|
| Case | Parent object for all CSAT scores — one score per closed Case | Id, Status, OwnerId, ContactId |
| Contact | Referenced from Case for context; no fields added | Id, Name |

### New Custom Objects (CS Prefix)

| API Name | Label | Description | Key Fields |
|---|---|---|---|
| CS_SatisfactionScore__c | Satisfaction Score | Stores a customer's 1–5 satisfaction rating for a resolved support case | CS_Rating__c, CS_Date__c, CS_CustomerNotes__c, CS_Case__c |

### Field Definitions

#### CS_SatisfactionScore__c

| API Name | Label | Type | Required | Default | Description |
|---|---|---|---|---|---|
| CS_Rating__c | Rating (1–5) | Number(1, 0) | Yes | — | Customer satisfaction score. Enforced 1–5 by Validation Rule. |
| CS_Date__c | Score Date | Date | Yes | Set by Flow (today) | Date the score was captured or assigned. |
| CS_CustomerNotes__c | Customer Notes | Long Text Area(32768) | No | — | Free-form feedback from the customer. |
| CS_Case__c | Case | Lookup(Case) | Yes | — | The support Case this score relates to. Lookup (not master-detail) so scores are preserved if the Case is deleted. |

> **Why Lookup, not Master-Detail?** A master-detail relationship would cascade-delete all satisfaction scores if the parent Case is deleted. Since CSAT data has long-term reporting value (even after cases are archived), a Lookup is the correct relationship type.

### Relationships

```
Case (1) ──────────< (0..1) CS_SatisfactionScore__c
                              CS_Case__c = Lookup(Case)

Case >────── (1) Contact
  (CS_SatisfactionScore__c inherits Contact context via Case.ContactId)
```

---

## Automation Strategy

### Recommended Approach

**Declarative only:** Validation Rule + Record-Triggered Flow. No Apex.

---

### Automation 1: Validation Rule — Enforce Rating Range

**Object:** CS_SatisfactionScore__c
**Rule Name:** CS_Rating_Range_1_to_5
**Formula:**
```
OR(CS_Rating__c < 1, CS_Rating__c > 5)
```
**Error Message:** "Rating must be between 1 and 5."
**Error Location:** CS_Rating__c field

---

### Automation 2: Flow — Auto-Create Score on Case Closure

**Flow Name:** CS_CreateSatisfactionScoreOnCaseClosure
**Flow Type:** Record-Triggered Flow
**Trigger Object:** Case
**Trigger Condition:** After Update — when `Status` changes to `"Closed"` AND prior value was not `"Closed"`

**Entry condition formula:**
```
AND(
  ISPICKVAL(Status, "Closed"),
  NOT(ISPICKVAL(PRIORVALUE(Status), "Closed"))
)
```

**Flow Logic (step by step):**

1. **Get Records** — Query `CS_SatisfactionScore__c` where `CS_Case__c = {Case.Id}`, limit 1
2. **Decision** — "Score Already Exists?"
   - **Yes path:** `{Get_Existing_Score.Id}` is not null → Exit (do nothing)
   - **No path:** Continue to Create Record
3. **Create Record** — New `CS_SatisfactionScore__c`:
   - `CS_Case__c` = `{Case.Id}`
   - `CS_Date__c` = `{!$Flow.CurrentDate}`
   - `CS_Rating__c` = left blank (agent/customer fills in)
   - `CS_CustomerNotes__c` = left blank

**Why Flow, not Apex?**
This is a straightforward create-on-condition automation with a simple duplicate check. Record-Triggered Flows handle this natively without governor limit concerns. Apex would add unnecessary complexity and test coverage overhead.

---

## Component Breakdown

| Requirement | Solution Type | Component Name | Notes |
|---|---|---|---|
| Store CSAT data | Custom Object | CS_SatisfactionScore__c | 4 fields + Lookup to Case |
| Enforce 1–5 range | Validation Rule | CS_Rating_Range_1_to_5 | On CS_SatisfactionScore__c |
| Auto-create on closure | Record-Triggered Flow | CS_CreateSatisfactionScoreOnCaseClosure | After Update on Case |
| Prevent duplicates | Flow Decision element | (part of above Flow) | Get Records → Decision |
| View on Case record | Page Layout config | Case Page Layout (standard) | Add CS_SatisfactionScore__c related list |
| Reporting by agent/date | Custom Report Type + Report | "Cases with Satisfaction Scores" | Admin setup — no code |
| CSAT trend dashboard | Standard Dashboard | CSAT Trend Dashboard | Uses above Report |

---

## Implementation Order

1. **Create `CS_SatisfactionScore__c` custom object** (Label: Satisfaction Score, API: CS_SatisfactionScore__c)
2. **Add all 4 custom fields** (CS_Rating__c, CS_Date__c, CS_CustomerNotes__c, CS_Case__c)
3. **Create Validation Rule** `CS_Rating_Range_1_to_5` on CS_SatisfactionScore__c
4. **Add CS_SatisfactionScore__c related list** to Case Page Layout (admin: Setup → Object Manager → Case → Page Layouts)
5. **Build Flow** `CS_CreateSatisfactionScoreOnCaseClosure` (Record-Triggered, After Update on Case)
6. **Activate Flow** — test by updating a Case Status to Closed in a sandbox
7. **Create Custom Report Type** "Cases with Satisfaction Scores" (primary: Cases, related: Satisfaction Scores)
8. **Build Report + Dashboard** using the new report type

---

## Risks & Considerations

- **Blank rating after auto-create:** The Flow creates the score record with CS_Rating__c blank intentionally. CS_Rating__c is marked required, so agents must fill it in before saving. Consider a Page Layout rule or a notification to the Case Owner reminding them to complete the score.
- **Cases re-opened and re-closed:** The Flow's duplicate check prevents creating a second score if a Case is re-closed. The original score record remains — agents can manually update it.
- **Validation Rule blocks blank ratings on auto-create:** The Flow sets CS_Rating__c to blank, but if the Validation Rule fires on insert (before the agent enters a rating), it will block the Flow. **Fix:** Set CS_Rating__c default value to 3 in the Flow, or make the Validation Rule only fire when Rating is explicitly non-null. Recommended: change the Validation Rule to `AND(NOT(ISNULL(CS_Rating__c)), OR(CS_Rating__c < 1, CS_Rating__c > 5))` so blank is allowed on creation.
- **Org edition:** If the org upgrades to Enterprise/Unlimited, consider migrating to Salesforce Surveys for richer feedback capabilities.
- **Sharing model:** CS_SatisfactionScore__c will inherit OWD from its setup. Default to "Public Read/Write" or "Controlled by Parent" (if using master-detail — but we're using Lookup, so set explicitly). Recommended: set OWD to "Public Read/Write" so all service agents can view all scores.
