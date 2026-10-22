# Ticket #4 — Customer Login Request

## Case overview

| Field | Recorded information |
|---|---|
| Ticket ID | **#4** |
| Requester | **Emily** — fictional lab employee |
| Subject | `no login` |
| Description | `i cant login` |
| Initial status shown | **Open** |
| Priority shown | **Normal** |
| Initial assignee | **Blank / unassigned** (identified when opened in Support) |
| Later assignment | I assigned the ticket to myself (reported in the lab conversation) |
| Resolution | **Not documented** — no supported evidence of a successful authentication fix |
| Date | October 22, 2026 |

## What happened

**1. Customer-side intake:** I activated the Help Center and signed in as Emily using a separate customer session. Emily submitted the short `no login` request. Zendesk returned ticket **#4**, visible in the end-user portal under Emily's name.

![Emily's ticket #4](../screenshots/07-emily-ticket-04.webp)

*Evidence — ticket #4 exists, Emily is the requester, and Zendesk shows it as Open with Normal priority.*

**2. Agent-side discovery:** Returning to Support, the personal Tickets page still showed only the preloaded sample ticket **#1**.

![Agent work list](../screenshots/08-agent-dashboard.webp)

*Evidence — a personal work list is not automatically an all-ticket intake queue.*

**3. Assignment:** I used ticket search to find **#4**, saw that it had no assignee, and assigned it to myself. This last action is based on my recorded account of the lab, not on a separate before/after assignment screenshot.

## Initial triage

The ticket only says **"i cant login"**, so it is not yet possible to identify the application, error, scope, or underlying cause.

### Facts established

- A user reported an inability to sign in.
- The request was recorded in Zendesk as ticket #4.
- The requester was Emily, not the agent account.
- The shown priority was Normal.
- The ticket initially had no assignee and did not appear in the personal ticket list.

### Unknowns

- Whether the affected service is Windows, VPN, Microsoft 365, or something else.
- Exact error message and start time.
- Whether any other users are affected.
- Whether the account is disabled, locked, expired, or blocked by conditional access.
- Whether the incident was actually resolved.

## Appropriate technician response (example, **not sent in this lab**)

**Public reply example:**

> Hi Emily, I can help you with the login issue. Which application or service are you trying to access, and what exact error message do you see? Also, does the problem affect other applications, or only this one? Please don't send your password.

**Internal note example:**

> Requester reports login failure; affected system and exact error unknown. Ticket initially unassigned. Need to identify application, determine scope and impact, then check approved account/sign-in diagnostics before selecting remediation.

**Suggested status:** Pending **only after** a clarification request has actually been sent and the technician is waiting for Emily to respond.

**Ticket type:** Incident would be a plausible choice if an existing service stopped working, but the screenshot does not verify a type change for ticket #4.

**Tags:** Do not invent organization-specific tags. A practice tag such as `login_issue` could be used if the lab has a defined convention. The screenshots do not prove any tag was applied to #4.

## Evidence and limitations

This is a **ticket-intake / assignment** scenario, not a completed technical fix. No screenshot confirms an Entra ID sign-in review, an Active Directory account unlock, a Teams client repair, a resolved public reply, or a Zendesk ticket marked Solved. These would be useful follow-up exercises, not findings from the current case.

The earlier sample ticket #1 was used for understanding interface fields and ticket statuses and should not be mistaken for Emily's ticket #4.
