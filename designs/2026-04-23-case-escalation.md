# Solution Design: Case Escalation on Inactivity

**Date:** 2026-04-23
**Story File:** stories/2026-04-23-case-escalation.md
**Architect:** CSAgent Design Agent

---

## Executive Summary

This solution surfaces stalled open Cases to support managers using Salesforce's native Case Escalation Rules, a Classic Email Template, a Case List View, and a standard Report and Dashboard — all declarative, zero custom code. The Escalation Rule fires once per stalled Case after 48 hours of inactivity (measured by Last Modified Date), notifies the Case Owner's manager via email with full Case context, and optionally reassigns the Case. A "Stalled Cases" List View (deployable as metadata) and a Report-backed Dashboard provide proactive, real-time manager visibility alongside the escalation email.

---

## Golden Rule Assessment

| Requirement | Standard Feature? | Config Sufficient? | Code Required? |
|---|---|---|---|
| Auto-escalate Case after 48 h of inactivity | Yes — Case Escalation Rules (Setup > Service > Escalation Rules) | Yes — rule entry criteria + escalation action, no Flow or Apex needed | No |
| Notify manager with Case context email | Yes — Classic Email Template + Escalation Rule notification action | Yes — merge fields cover all required Case fields | No |
| "Stalled Cases" List View for manager visibility | Yes — Standard Case List View with relative date filter | Yes — List View metadata XML can be deployed via SFDX | No |
| Dashboard showing stalled Case count by owner | Yes — Standard Report + Dashboard component | Yes — Reports and Dashboards, no custom code | No |

**No custom code is required for any story in this epic.**

---

## Data Model

### Standard Objects Used

| Object | Why Used | Key Fields Used |
|---|---|---|
| Case | Core support record being monitored and escalated | CaseNumber, Subject, Status, LastModifiedDate, OwnerId |
| User | Represents Case Owners and their managers via Role Hierarchy | Id, Name, Email, ManagerId (resolved via Role Hierarchy) |
| Account | Parent of Case, included in escalation email context | Name |
| Contact | Requester on the Case, included in escalation email context | Name |

### New Custom Objects (CS Prefix)

None. All requirements are met by standard objects.

### Field Definitions

No custom fields are required. All data points referenced in the stories (Case Number, Subject, Account Name, Contact Name, Status, Last Modified Date, Case link) are standard Case fields or standard relationship fields already available as merge fields in Email Templates and as columns in List Views.

### Relationships

All relationships used are standard, pre-existing relationships:

```
Account (1) ──────────< (M) Case
Contact  (1) ──────────< (M) Case
User     (1) Owner ────< (M) Case
User     (M) Role Hierarchy ──> (1) Manager User   [resolved at runtime by Escalation Rule]
```

---

## Automation Strategy

### Recommended Approach

Declarative only — Escalation Rule + Classic Email Template. No Flow, no Apex trigger.

The native Escalation Rule engine is the correct tool: it was built specifically for time-based Case inactivity escalation, runs outside the 24-hour governor constraints that affect Scheduled Flows, fires exactly once per escalation entry (satisfying the "escalate only once" acceptance criterion), and natively walks the Role Hierarchy to identify the Case Owner's manager without any custom logic.

### Escalation Rule Design

- **Rule Name:** 48-Hour Inactivity Escalation
- **Active:** Yes
- **Rule Entry Criteria:** Case Status EQUALS Open (extend to all non-closed statuses as agreed with the business — e.g., also "New", "Waiting on Customer" if those are considered active-inactivity states)
- **Sort Order:** 1 (only one entry needed unless multi-tier escalation is added later)
- **Escalation Action:**
  - Age Over: 48 hours
  - Business Hours: Configure a Business Hours record and attach here if the business excludes nights and weekends; leave as "Ignore Business Hours" for calendar-time SLAs
  - Notify: Case Owner's manager (resolved via Role Hierarchy)
  - Email Template: CS_CaseEscalationNotification (see Email Template section below)
  - Reassign To: Manager's queue or manager's User record (confirm with business; leave blank to notify without reassignment)

**Queue Owner edge case:** If the Case Owner is a Queue rather than a User, the Role Hierarchy lookup has no manager to resolve. The business must specify a fallback recipient — options are: (a) send to the Queue email address, or (b) designate a named manager user as the escalation recipient for queue-owned Cases. This must be captured in a business decision before go-live.

### Email Template Design

- **Template Name:** CS_CaseEscalationNotification
- **Template Type:** Classic Text or Classic HTML (Lightning Email Templates are not supported as Escalation Rule notification templates as of API v62.0)
- **Subject:** Case Escalation Alert: Case {!Case.CaseNumber} — No Update for 48 Hours
- **Body merge fields to include:**

| Field | Merge Field Syntax |
|---|---|
| Case Number | `{!Case.CaseNumber}` |
| Case Subject | `{!Case.Subject}` |
| Account Name | `{!Case.Account.Name}` |
| Contact Name | `{!Case.Contact.Name}` |
| Current Status | `{!Case.Status}` |
| Last Modified Date | `{!Case.LastModifiedDate}` |
| Direct Case Link | `{!Case.Link}` |

- **Additional recipient option:** A CC distribution list (e.g., support-ops alias) can be added as a secondary recipient in the Escalation Action — confirm with business.

### List View Design (Deployable Metadata)

- **List View Name:** Stalled Cases (48h+)
- **API Name:** Stalled_Cases_48h (on the Case object)
- **Filter Logic:** Status EQUALS Open AND LastModifiedDate LESS THAN LAST_N_DAYS:2
- **Columns:** CaseNumber, Subject, Account.Name, Status, Owner.Name, LastModifiedDate
- **Visibility:** Shared with All Internal Users (or scoped to Service Agent and Support Manager profiles as agreed)
- **Deployable:** Yes — as `force-app/main/default/objects/Case/listViews/Stalled_Cases_48h.listView-meta.xml`

**Precision caveat:** The LAST_N_DAYS:2 relative filter counts calendar days (midnight-to-midnight), not rolling 48 hours. This is a known List View platform limitation. The escalation rule (which can be set to exact hours) is the authoritative escalation mechanism; the list view is a supplementary visibility tool and need not be perfectly precise to the hour.

### Dashboard Design (Setup UI Only — Not Deployable as Metadata via SFDX)

- **Report Name:** CS_StalledCasesReport
- **Report Type:** Cases
- **Filters:** Status EQUALS Open; Last Modified Date LESS THAN LAST N DAYS 2
- **Grouping:** Case Owner (for bar/table component showing count per agent)
- **Dashboard Component:** Bar chart or summary table — "Stalled Cases by Owner"
- **Dashboard Name:** CS_SupportManagerDashboard (or add component to existing manager dashboard if one already exists)
- **Dashboard Visibility:** Shared with Support Manager role

---

## Why Code? — Not Applicable

No Apex or LWC is required for any requirement in this epic.

All three stories are satisfied entirely by standard Salesforce features:
- Story 1: Case Escalation Rules — a standard Service Cloud feature purpose-built for this exact use case.
- Story 2: Classic Email Templates + Escalation Rule notification action — natively supports all required Case merge fields.
- Story 3: Standard List View with relative date filters + standard Reports and Dashboards.

Custom code would add deployment complexity, governor limit exposure, and maintenance overhead with zero functional benefit over the native tools available.

---

## Component Breakdown

| Story | Requirement | Solution Type | Component Name | Deployable via SFDX? | Notes |
|---|---|---|---|---|---|
| 1 | 48-hour inactivity escalation rule | Setup UI — Escalation Rule | 48-Hour Inactivity Escalation | No — Setup UI only | Configure under Setup > Service > Escalation Rules |
| 1 | Business Hours record (if calendar exclusion needed) | Setup UI — Business Hours | Support Business Hours | No — Setup UI only | Optional; attach to Escalation Action if non-calendar SLA |
| 2 | Escalation notification email with Case context | Setup UI — Classic Email Template | CS_CaseEscalationNotification | No — Classic Templates are Setup UI only | Must be Classic type; Lightning Templates not supported by Escalation Rules |
| 3 | Stalled Cases list view | Metadata XML — List View | Stalled_Cases_48h (on Case) | Yes — listView-meta.xml | Path: force-app/main/default/objects/Case/listViews/ |
| 3 | Stalled Cases report | Setup UI — Report | CS_StalledCasesReport | No — Reports are Setup UI only | Cases report type; same filters as list view |
| 3 | Stalled Cases dashboard component | Setup UI — Dashboard | CS_SupportManagerDashboard | No — Dashboards are Setup UI only | Add to existing manager dashboard or create new |

---

## Implementation Order

1. **Configure Business Hours** (if required) — Setup > Company Settings > Business Hours. Create a "Support Business Hours" record defining operational hours. This must exist before the Escalation Rule is created if business-hours-based counting is needed.
2. **Create the Classic Email Template** — Setup > Email > Classic Email Templates > New. Build the CS_CaseEscalationNotification template with all required merge fields. Test the merge field rendering before attaching to the rule.
3. **Create and activate the Escalation Rule** — Setup > Service > Escalation Rules > New. Configure rule entry criteria (Status = Open), escalation action (48 hours, email template, manager notification, optional reassignment), and mark the rule Active. Confirm the Queue Owner fallback with the business before activating.
4. **Deploy the List View metadata** — Run `sf project deploy start` to push the Stalled_Cases_48h listView-meta.xml to the org. Verify the list view appears on the Cases tab and is shared correctly.
5. **Create the Stalled Cases Report** — Reports tab > New Report > Cases object. Apply the same filters as the list view. Save as CS_StalledCasesReport.
6. **Create or update the Dashboard** — Dashboards tab > New (or edit existing manager dashboard). Add a bar or table component from CS_StalledCasesReport. Share with Support Manager role.
7. **Sandbox validation** — Shorten the escalation window to 1 hour (or use a test Case with an old LastModifiedDate via data load), confirm the escalation email arrives at the manager's inbox, verify Case owner changes if reassignment is configured, and confirm the List View surfaces the Case correctly.
8. **Go-live and rollback plan** — Mark the Escalation Rule inactive to disable escalation instantly if issues are found post-launch; no code rollback or deployment reversal required.

---

## Risks and Considerations

- **Queue Owner escalation gap** — If a Case is owned by a Queue, Salesforce cannot walk the Role Hierarchy to find a manager. The business must define a fallback recipient before go-live. Without this decision, queue-owned Cases will not trigger escalation notifications.
- **Business hours vs. calendar hours alignment** — The List View uses calendar days (LAST_N_DAYS:2) while the Escalation Rule can be configured to use business hours. These two may surface different Cases. Communicate this distinction to managers so they do not treat the list view as the definitive escalation record.
- **Escalation fires once** — The Escalation Rule's "once per entry" behavior is by design and satisfies the acceptance criterion. However, if a Case is updated (resetting LastModifiedDate) and then stalls again, the clock resets and the rule will fire again. This is correct behavior but should be communicated to the support team.
- **Role Hierarchy completeness** — The manager notification depends on all User records having their Role set correctly. If any agent's Role is blank or the Role Hierarchy is incomplete, those Cases will not generate a manager notification. A one-time audit of User Role assignments is recommended before go-live.
- **Classic Email Template limitation** — Lightning Email Templates cannot be used with Escalation Rule notification actions as of API v62.0. If the org plans to migrate fully to Lightning Email Templates, this constraint must be re-evaluated at that time.
- **Entitlement and Milestone overlap** — If the org uses Entitlements and Milestones for SLA tracking, confirm with the support operations team that this Escalation Rule complements (rather than duplicates) existing Milestone Actions. Running both can result in duplicate manager notifications for the same stalled Case.
- **Report and Dashboard are not source-trackable** — Reports and Dashboards cannot be deployed via `sf project deploy start` in most orgs without enabling Reports and Dashboards as metadata. If the org requires source-tracked deployments for all components, evaluate whether Reports metadata deployment is feasible; otherwise treat these as manual Setup UI steps.
