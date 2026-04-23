# Case Escalation on Inactivity

**Date:** 2026-04-23
**Original Requirement:** Automatically escalate a Case to a manager if it stays open for more than 48 hours with no update
**Epic Summary:** Ensure that open Cases with no agent activity within 48 hours are automatically surfaced to a manager so nothing falls through the cracks.

---

## Story 1: Configure Case Escalation Rule for 48-Hour Inactivity

**As a** Service Agent
**I want** the system to automatically escalate my open Cases to my manager when there has been no update for 48 hours
**So that** no customer issue is silently stalled and management can intervene before SLAs are breached

### Acceptance Criteria
- [ ] An Escalation Rule is active on the Case object targeting Cases where Status is Open (or any non-closed status defined by the business)
- [ ] The escalation criteria uses the "Hours since last update" or "Hours since case opened" condition set to 48 hours
- [ ] When the 48-hour threshold is crossed, the Case Owner's manager (Queue manager or direct manager via Role Hierarchy) receives an email notification
- [ ] The escalated Case is reassigned to the manager's queue or the manager's user record, as agreed with the business
- [ ] The escalation fires only once per inactivity window — it does not fire repeatedly for the same stalled Case
- [ ] Testers can validate by shortening the escalation window (e.g., to 1 hour) in a sandbox, creating an Open Case, leaving it untouched, and confirming the notification email arrives and the Case owner changes

### Standard Feature Check
> WARNING STANDARD FEATURE: Use Salesforce Case Escalation Rules (Service Cloud). Navigate to Setup > Service > Escalation Rules. Create a rule with criteria "Case: Status equals Open" and set the escalation action at 48 hours based on "Hours since case was opened" or "Hours since last modification" (i.e., the Last Modified Date field). No custom development needed.
>
> Configuration steps:
> 1. Setup > Service > Escalation Rules > New
> 2. Rule name: "48-Hour Inactivity Escalation" — mark as Active
> 3. Rule Entry Criteria: Case Status EQUALS Open (add additional non-closed statuses as needed)
> 4. Escalation Action: Age Over = 48 hours; select "Ignore business hours" or configure business hours as required
> 5. Notification: choose Email Template and notify the Case Owner's manager (set via the user's Role Hierarchy)
> 6. Optionally reassign the Case to a Manager Queue

### Metadata Impact
- Objects affected: Case (standard)
- New CS_ objects needed: None
- Automation type needed: Declarative — Escalation Rule (Setup UI configuration, no Flow or Apex required)

**Priority:** High
**Story Points:** 2
**Notes:** The "last update" criterion maps to the standard Case Last Modified Date field — this is what Escalation Rules evaluate when using the "hours since last modification" option. Confirm with the business whether business hours should be excluded (e.g., do not count nights and weekends toward the 48 hours). If business hours must be excluded, configure a Business Hours record and attach it to the Escalation Rule. If the org uses Entitlements and Milestones for SLA tracking, consider whether this escalation rule duplicates or complements existing Milestone Actions — align with the support operations team before activating.

---

## Story 2: Notify the Manager with Case Context via Email Alert

**As a** Support Manager
**I want** to receive an email notification that includes the Case details (Case number, subject, customer, current status, and last activity date) when a Case is escalated to me
**So that** I have enough context to act immediately without having to open Salesforce first

### Acceptance Criteria
- [ ] The escalation notification email includes: Case Number, Case Subject, Account Name, Contact Name, Current Status, Date/Time of Last Update, and a direct link to the Case record
- [ ] The email is sent to the manager of the Case Owner as defined by the Salesforce Role Hierarchy
- [ ] If the Case Owner is a Queue (not a user), the notification is sent to the Queue email address or a designated manager user — business must specify this fallback
- [ ] Testers can confirm the email content by inspecting a received escalation email in a sandbox and verifying all fields are populated correctly

### Standard Feature Check
> WARNING STANDARD FEATURE: Use a standard Email Template (Setup > Email > Classic Email Templates or Lightning Email Templates) combined with the Escalation Rule notification action configured in Story 1. No custom development needed. The email template supports merge fields from the Case object, allowing all required Case details to be included.
>
> Configuration steps:
> 1. Setup > Email > Classic Email Templates > New Template (type: Text or HTML)
> 2. Add merge fields: {!Case.CaseNumber}, {!Case.Subject}, {!Case.Account.Name}, {!Case.Contact.Name}, {!Case.Status}, {!Case.LastModifiedDate}, {!Case.Link}
> 3. Attach this template to the Escalation Action in the Escalation Rule created in Story 1

### Metadata Impact
- Objects affected: Case (standard), User (standard — Role Hierarchy for manager lookup)
- New CS_ objects needed: None
- Automation type needed: Declarative — Email Template + Escalation Rule notification action

**Priority:** High
**Story Points:** 1
**Notes:** Classic Email Templates are required for Escalation Rule notification actions — Lightning Email Templates are not supported in this context as of API v62.0. Confirm with the business whether the email must also CC a distribution list (e.g., a support-ops alias) — this can be added as a secondary recipient in the Escalation Action. If the org requires a chatter post or an in-app notification in addition to email, a separate scheduled Flow can be layered on top without replacing the Escalation Rule.

---

## Story 3: Create a "Stalled Cases" List View and Dashboard for Manager Visibility

**As a** Support Manager
**I want** a list view and a dashboard component showing all Open Cases with no update in the last 48 hours
**So that** I have a proactive, real-time view of at-risk Cases without waiting for escalation emails

### Acceptance Criteria
- [ ] A Case list view named "Stalled Cases (48h+)" is visible to all users with the Service Agent and Support Manager profiles
- [ ] The list view filters Cases where: Status is Open AND Last Modified Date is less than or equal to 48 hours ago (i.e., NOW minus 2 days)
- [ ] The list view columns include: Case Number, Subject, Account Name, Status, Owner, Last Modified Date, Age (in hours)
- [ ] A dashboard component (report-based) displays the count of stalled Cases, grouped by owner or queue, visible on the Support Manager home page
- [ ] Testers verify the list view by creating an Open Case, backdating Last Modified Date in a sandbox data load, and confirming the Case appears in the view

### Standard Feature Check
> WARNING STANDARD FEATURE: Use standard Salesforce List Views (Case object > New List View) and standard Reports and Dashboards. No custom development needed.
>
> Configuration steps for List View:
> 1. Cases tab > List Views > New > Name: "Stalled Cases (48h+)"
> 2. Filter: Status EQUALS Open; AND Last Modified Date LESS THAN LAST N DAYS:2
> 3. Share with: All Internal Users (or specific roles)
>
> Configuration steps for Dashboard:
> 1. Reports > New Report > Case object
> 2. Filter: same criteria as list view
> 3. Group by: Case Owner
> 4. Add to a Dashboard as a bar or table component

### Metadata Impact
- Objects affected: Case (standard)
- New CS_ objects needed: None
- Automation type needed: None — declarative List View and Report/Dashboard only

**Priority:** Medium
**Story Points:** 2
**Notes:** The "Last Modified Date less than 2 days ago" filter in a List View uses the relative date filter "LAST N DAYS:2" — note this counts calendar days, not business hours. If the business defines 48 hours as 2 business days, the list view cannot perfectly replicate that logic; the escalation rule (with business hours configured) remains the authoritative escalation mechanism. The list view is a supplementary visibility tool and need not be perfectly precise.
