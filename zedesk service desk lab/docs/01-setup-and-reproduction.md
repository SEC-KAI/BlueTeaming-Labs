# Setup and Reproduction — Zendesk Service Desk Lab

## Purpose

Recreate a small IT support environment that has **one agent** and **one employee**. The employee submits a request, and the agent reviews and takes ownership of the resulting ticket.

## Prerequisites

- Access to a Zendesk Support account or trial that includes a Help Center / Knowledge feature.
- Permission to use **Support** and activate the **Help Center** (if required).
- Two email identities under your control, one for the technician and one for the test requester.
- Two separate browser sessions (for example, a normal Chrome session and Incognito).

Do not use an employer's real Zendesk instance for unapproved practice or submit live requests to a production service desk.

## Procedure Completed in My Lab

### Part A — Explore Support

1. Sign in to Zendesk and open **Support**.
2. Select **Your work → Tickets**.
3. Open the sample login ticket to examine its **Requester**, **Group/Assignee**, **Type**, **Priority**, **Tags**, and **Status**.
4. Identify the message composer and the dropdown that switches between **Public reply** and **Internal note**.
5. Locate the **Submit as [status]** control at the bottom.

Evidence: [Agent workspace](../screenshots/01-agent-workspace.webp) and [ticket detail](../screenshots/02-ticket-fields-and-composer.webp).

### Part B — Check and activate Knowledge / Help Center

1. Open **Knowledge** from the product selector.
2. Open the customer-facing Help Center preview.
3. Check whether the customer portal is already publicly active. Initially, a banner stated that it was not visible to end users.
4. Open **Knowledge admin → Settings → Help center settings**.
5. Activate the Help Center using the available control and save/update the settings.

My initial direct attempt at **Submit a request** resulted in a "This page doesn't exist" page. I also encountered **Access Denied** in an Admin Center end-user settings area. These observations were part of troubleshooting setup—not evidence that all Zendesk instances behave the same way.

Evidence: [preview](../screenshots/03-help-center-preview.webp), [unavailable form](../screenshots/04-request-form-unavailable.webp), [permissions limitation](../screenshots/05-admin-settings-access-denied.webp), and [settings updated](../screenshots/06-activation-confirmed.webp).

### Part C — Sign in as the employee

1. Keep the Zendesk agent session available in one browser.
2. Open the Help Center in a separate session.
3. Sign in with a **different end-user account** (my test account was **Emily**).
4. Make sure the Help Center shows the end user's name rather than the agent's name.

### Part D — Submit the end-user ticket

1. From the Help Center, select **Submit a request**.
2. Enter a short subject and issue description.
3. For my lab I used:
   - **Subject:** `no login`
   - **Description:** `i cant login`
4. Submit the request.
5. Confirm the requester name, ticket number, status, and priority in the end-user portal.

**Observed result:** Emily's request created **ticket #4**, status **Open**, priority **Normal**.

Evidence: [Emily's ticket #4](../screenshots/07-emily-ticket-04.webp).

### Part E — Find and take ownership

1. Return to Zendesk Support in the agent session.
2. Check **Your work → Tickets**.
3. If the request is not shown, use ticket search and look up the ticket ID (for example, `4` or a `type:ticket id:4` query if supported in the search interface).
4. Open the ticket and inspect the **Assignee** field.
5. My ticket was initially **unassigned**, which explains why it was missing from the personal queue.
6. I selected myself as its assignee.

**Important:** A ticket can be created successfully even if it does not appear in an individual's personal view. A team should use an **Unassigned** or **triage** view for this type of intake; available views depend on roles and configuration.

Evidence: [agent dashboard before ticket lookup](../screenshots/08-agent-dashboard.webp). The exact "assign to me" selection was not screenshot-captured.

## Mistakes and Lessons

| Observation | What I learned |
|---|---|
| An earlier request was submitted while signed in as the agent | Zendesk recorded the **agent** as requester, not an independent employee |
| Requester received an email when ticket #3 was created | The customer notification and the agent work queue are different parts of the workflow |
| Initial request form failed | A Help Center visible to an admin may not yet be activated for end users |
| Admin Center settings displayed Access Denied | Administrative permissions vary by page and role |
| Emily's ticket #4 did not appear in 'Your work' | That screen did not include all unassigned tickets |
| Emily's ticket had no assignee | A technician or routing rule needs to establish ownership |

## Outcome

The customer portal and two-role test worked: a test employee successfully created a request, the request was assigned a ticket number, and I later located its unassigned record and took ownership. Public replies, resolution, SLAs, and triggers remain possible **future exercises**.
