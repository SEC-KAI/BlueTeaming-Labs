# Zendesk Service Desk Field Guide

This is a concise reference to the concepts I studied while using Zendesk. It explains the product/workflow; it does **not** imply every feature was configured in my trial.

## Where work happens

| Area | Purpose |
|---|---|
| **Support / Agent Workspace** | Read requests, troubleshoot, reply, update tickets, assign or escalate |
| **Knowledge / Help Center** | Customer self-service, knowledge articles, submitting and tracking requests |
| **Admin Center** | Configure agents, groups, fields, rules, channels, and permissions |

## Standard ticket fields

| Field | What it means |
|---|---|
| Requester | Person asking for help |
| Group | Team that owns the work |
| Assignee | Agent currently responsible for the ticket |
| Type | Incident, Problem, Question, or Task, where available |
| Priority | Low, Normal, High, Urgent; follow organizational impact/urgency guidelines |
| Tags | Labels useful for workflows, reporting, and search; follow documented conventions |
| Status | Who needs to take action or whether the issue has been solved |
| Public reply | Customer-visible message |
| Internal note | Agent/team-only documentation, subject to permission |

## Status and ownership

- **New:** Submitted, not yet acted upon/assigned in a typical default flow.
- **Open:** Agent attention is needed.
- **Pending:** Waiting for the requester to respond.
- **On-hold:** Waiting on an external party/dependency; may be unavailable if not enabled.
- **Solved:** Agent considers the request addressed and ready for closure workflow.
- **Closed:** Finalized by Zendesk's configured lifecycle, not usually manually set by agents.

Status and assignee are different: a request can be **Open** and still have **no assignee**. This distinction was important when locating ticket #4.

## Common ITSM concepts

**Incident:** A service is unexpectedly disrupted, for example an employee cannot sign in.

**Service request:** A user asks for a standard service, such as access to an approved application.

**Impact:** Scope and business consequence, such as one employee versus an entire office.

**Urgency:** How quickly action is needed to avoid serious interruption.

**SLA:** A target for response or resolution; the numbers and schedules depend on organizational policy. No SLA target was configured or measured in this lab.

**Escalation:** Transfer to a team or individual with greater access, expertise, or authority. Provide reproducible notes so the next technician does not restart the investigation.

## Queue, view, and search

- A **view** is a filtered list of tickets.
- **Your work** can omit requests not assigned to the current agent.
- A dedicated **Unassigned** or shared triage view is useful for new intake.
- Agents may search by ticket ID or other fields when permitted.
- Search visibility and interaction history depend on roles and access permissions.

## Macros, triggers, and AI triage

| Feature | Difference |
|---|---|
| Macro | An **agent** manually applies predefined ticket actions or text |
| Trigger | Zendesk automatically performs actions when an event-based condition is met |
| Automation | Time-based automatic rules, where configured |
| Intelligent triage | AI-assisted classifications (availability/plan dependent), which can be used to drive routing |
| Sentiment | The emotional tone of a request; **not** a technical diagnosis or a substitute for impact/urgency |

## Suggested first-response checklist

1. Read the entire request and identify the impacted service.
2. Check the requester's history **only if authorized**.
3. Assess impact and urgency before changing priority.
4. Search relevant known fixes and knowledge-base articles.
5. Check what has already been tried.
6. Make the smallest safe change appropriate to your access.
7. Add clear internal notes with facts and actions.
8. Reply publicly in plain language and ask the requester to verify.
9. Use a waiting status only when the relevant response/dependency has actually been requested.
10. Mark Solved only when reasonable resolution criteria have been met.

## Credential safety

Never request an employee's password, MFA recovery codes, tokens, or private authentication secrets in a ticket. Use approved identity verification and account-reset procedures.
