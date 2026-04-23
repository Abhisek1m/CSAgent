# Customer Satisfaction Score Tracking

**Date:** 2026-04-22
**Original Requirement:** Track customer satisfaction scores after support cases are closed. Each score should be linked to a Case, have a 1–5 rating, a date, and notes from the customer.
**Epic Summary:** Enable service teams to capture and review customer satisfaction scores tied to resolved support cases.

---

> ⚠️ STANDARD FEATURE: **Salesforce Surveys** (Enterprise/Unlimited editions) can send automated post-case surveys and capture satisfaction ratings natively, with built-in reporting. No custom development needed if your org has the Surveys feature enabled.
>
> Steps to evaluate: Setup → Surveys → Enable Surveys. If Surveys is available and meets the need, use it instead of this custom solution.
>
> **Proceed with custom stories below only if:** Surveys is not available in your edition, or the team needs CSAT scores as a first-class record linked directly to the Case object (not via survey response).

---

## Story 1: Record a Customer Satisfaction Score on a Closed Case

**As a** Service Agent
**I want** to create a satisfaction score record linked to a closed Case
**So that** the team has a structured, searchable record of how the customer rated their support experience

### Acceptance Criteria
- [ ] A `CS_SatisfactionScore__c` record can be created from the Case record page
- [ ] The score record requires a rating (1–5), a date, and a lookup to the Case
- [ ] Customer notes field accepts free-form text (up to 32,000 characters)
- [ ] Only one score record per Case is allowed (validation rule or unique constraint)
- [ ] The score record is visible in a related list on the Case record

### Standard Feature Check
Salesforce Surveys covers this natively on Enterprise+. If Surveys is unavailable or the team needs CSAT as a standalone related record (not a survey response), a custom object is required.

### Metadata Impact
- Objects affected: Case (standard), CS_SatisfactionScore__c (new custom)
- New CS_ objects needed: CS_SatisfactionScore__c
- Fields needed:
  - CS_Rating__c — Number(1,0), required
  - CS_Date__c — Date, required, default today
  - CS_CustomerNotes__c — Long Text Area(32768)
  - CS_Case__c — Lookup(Case), required
- Automation type needed: Validation Rule (enforce rating range 1–5), Flow (optional: auto-create record on case closure)

**Priority:** High
**Story Points:** 3
**Notes:** Consider a Validation Rule on CS_Rating__c to enforce value between 1 and 5. Master-detail vs lookup: use Lookup so scores survive if the Case is deleted.

---

## Story 2: Auto-Create a Satisfaction Score Record When a Case Is Closed

**As a** Service Manager
**I want** a satisfaction score record to be automatically created when a Case status changes to "Closed"
**So that** agents are prompted to collect feedback without a manual step

### Acceptance Criteria
- [ ] When a Case Status changes to "Closed", a CS_SatisfactionScore__c record is created with CS_Date__c set to today
- [ ] The auto-created record has CS_Rating__c blank (agent or customer fills it in)
- [ ] If a CS_SatisfactionScore__c record already exists for the Case, no duplicate is created
- [ ] Agent can edit the auto-created record to add rating and customer notes

### Standard Feature Check
No standard Salesforce feature auto-creates a related record on case closure. This requires a Record-Triggered Flow — no Apex needed.

### Metadata Impact
- Objects affected: Case (trigger), CS_SatisfactionScore__c (created)
- New CS_ objects needed: None (CS_SatisfactionScore__c created in Story 1)
- Automation type needed: Record-Triggered Flow on Case (after update, when Status changes to Closed)

**Priority:** Medium
**Story Points:** 2
**Notes:** Flow should use a Get Records element to check for an existing score before creating. This prevents duplicate records if the Case is re-closed.

---

## Story 3: View CSAT Summary on Case Record and Reports

**As a** Service Manager
**I want** to see satisfaction scores on a dashboard and filter by date range, agent, or case category
**So that** I can identify trends in customer satisfaction and coach the team accordingly

### Acceptance Criteria
- [ ] A related list on the Case record shows all associated CS_SatisfactionScore__c records
- [ ] A standard Salesforce Report shows average CS_Rating__c grouped by Case Owner (agent)
- [ ] A standard Dashboard chart displays average CSAT score over time (monthly trend)
- [ ] Report can be filtered by CS_Date__c range

### Standard Feature Check
✅ Standard Salesforce Reports and Dashboards fully cover this need. No custom code required. Build a custom report type on Case with CS_SatisfactionScore__c, then create the report and add it to a dashboard.

### Metadata Impact
- Objects affected: CS_SatisfactionScore__c (report source), Case (parent)
- New CS_ objects needed: None
- Automation type needed: None — standard Reports and Dashboards

**Priority:** Medium
**Story Points:** 2
**Notes:** Create a custom Report Type: "Cases with Satisfaction Scores" to enable reporting on both objects in one report. This is a 5-minute admin task.
