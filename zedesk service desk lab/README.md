# Zendesk Service Desk Lab

**Screenshot publication status:** The supporting images are prepared but **not yet uploaded**; the screenshot links in this document will work after those files are added. The textual documentation and case study have been published.


## Overview

This project documents a hands-on **Zendesk Support / Knowledge** lab that I built to understand an internal IT service desk from both perspectives: the employee submitting a request and the technician receiving, triaging, and taking ownership of a ticket.

I used a Zendesk trial, a separate end-user account named **Emily**, and a Chrome Incognito session to simulate the two roles. The main verified scenario was a basic **login issue** submitted from the customer portal as **ticket #4**.

> **Environment:** Personal practice tenant using a student-themed display name. It was **not** a real Fanshawe College or employer helpdesk, and no production employee accounts or systems were involved.

**Scope / evidence boundary:** This project documents the configuration and ticket intake steps I actually performed. The screenshots establish the end-user request and the Zendesk interface; I also located the unassigned ticket and assigned it to myself. They do **not** establish that I solved a real authentication failure, configured live SLAs or automations, or completed an entire customer-to-agent resolution thread. The training prompts and proposed resolutions are marked as examples, not completed actions.

## Lab Objectives

- Distinguish the **agent workspace**, **Admin Center**, and **customer-facing Help Center**.
- Inspect ticket fields: requester, assignee, group, status, type, priority, tags, and conversation.
- Understand the differences between a **public reply** and an **internal note**.
- Activate a customer portal for a two-account test.
- Create a ticket as an end user and find it from the agent side.
- Understand why **unassigned tickets may not appear in a personal work queue**.
- Practice ticket triage, assignment, documentation, and appropriate next steps.

## Environment

| Component | Role in the lab |
|---|---|
| Zendesk Support | Agent workspace for viewing, assigning, and working tickets |
| Zendesk Knowledge / Help Center | Employee-facing page to submit and review requests |
| Admin Center | Administrative settings and access controls |
| Agent account | Technician/admin session |
| Emily account | Separate simulated employee/requester |
| Chrome regular / Incognito sessions | Kept the technician and employee sessions separate |
| Sample ticket #1 | Preloaded ticket used to explore agent fields and statuses |
| Ticket #3 | Early self-submitted ticket used to understand requester identity |
| Ticket #4 | End-user **Emily** submitted a `no login` request |

## What I Did

### 1. Explored the agent workspace

I opened Zendesk Support and reviewed the preloaded **"Unable to log in after password reset"** sample ticket. I identified the **requester**, **assignee**, **priority**, **type**, **status**, **tags**, **public reply** composer, and customer **interaction history**. The ticket screen also showed **Submit as Open**, illustrating that saving a ticket can set its status.

![Agent ticket workspace](screenshots/02-ticket-fields-and-composer.webp)

*Screenshot 1 — Sample ticket #1 in the agent workspace. The form separates ticket metadata, the conversation, and requester history.*

### 2. Checked the customer-facing Help Center

I opened Zendesk Knowledge. The Help Center was visible to me as an administrator, but initially **not activated for end users**. Opening the request form earlier produced an unavailable page. I later enabled the Help Center and saw confirmation that its settings had been updated.

![Help Center settings confirmation](screenshots/06-activation-confirmed.webp)

*Screenshot 2 — Knowledge settings after updating the Help Center configuration.*

### 3. Separated the user and technician accounts

I kept separate browser sessions to represent the agent and employee. This was important because an earlier test submitted while signed in as the agent created a ticket **on my own behalf** (ticket #3), and Zendesk sent the corresponding requester notification email.

### 4. Submitted a request as Emily

After activating the portal, I signed into the end-user side as **Emily** and created ticket **#4**, titled **"no login"**, with the description **"i cant login"**. The requester page displayed **Emily**, **Open** status, and **Normal** priority.

![Ticket 4 in Emily's Help Center](screenshots/07-emily-ticket-04.webp)

*Screenshot 3 — The end-user view confirms a request created by Emily, with ticket ID #4.*

### 5. Investigated the agent ticket queue and assignment

The agent's **Your work → Tickets** screen initially showed only the sample ticket, not Emily's newly created ticket. I found ticket #4 using search; its **assignee was blank**. I then assigned the ticket to myself. This illustrated the distinction between a **personal working queue** and an **unassigned/shared queue**. The screenshot below establishes the initial dashboard behavior; the assignment itself was described during the lab but was **not captured in the supplied screenshot set**.

![Agent dashboard with filtered ticket list](screenshots/08-agent-dashboard.webp)

*Screenshot 4 — The agent dashboard showed a single sample ticket before #4 was found through search.*

## How the Ticket Workflow Works

1. **Employee** submits a request in the Help Center (or through a connected email channel).
2. **Zendesk** records the request and creates a numbered ticket.
3. **Service desk** locates the new/unassigned ticket in an appropriate view or search.
4. **Technician** reviews the problem, chooses a group/assignee, validates priority, and categorizes the ticket using company conventions.
5. **Technician** sends a **public reply** to ask for clarification or share a fix; **internal notes** document checks and handoffs.
6. **Status** changes according to who needs to act: Open for agent action, Pending for requester response, On-hold when available for an external dependency, and Solved after resolution is confirmed.
7. **Service desk** documents the resolution and allows Zendesk's lifecycle to handle eventual closure.

**This is the standard workflow modeled in the lab, not a claim that every step was executed on ticket #4.**

## Key Learnings

- **ITIL / ITSM** describes how service support is organized; **Zendesk** is a tool that implements ticket-handling processes.
- **Assignment** determines who owns the next action; a ticket can exist without an assignee.
- **Priority** is based on impact and urgency, not solely on the requester's wording.
- **SLA** targets and escalation paths are organization-defined; none were configured or measured in this exercise.
- **Tags** help reporting/searching/routing; agents should follow a company tagging convention instead of guessing labels.
- **Macros** are manually applied canned actions; **triggers** evaluate conditions automatically.
- **Past ticket history** and internal notes provide continuity between authorized agents, subject to permissions.
- The customer-facing **Help Center** and the technician **agent workspace** show different information about the same ticket.

## Documentation

- [Setup and reproduction steps](docs/01-setup-and-reproduction.md)
- [Ticket #4: intake and triage record](docs/02-ticket-04-case-study.md)
- [Service desk field guide](docs/03-zendesk-field-guide.md)
- [Optional next exercises (not yet performed)](docs/04-next-exercises.md)
- [Screenshots and what each proves](screenshots/README.md)
- [Portfolio summary](portfolio-card.md)

## Repository Layout

```text
zedesk service desk lab/
├── README.md
├── portfolio-card.md
├── docs/
│   ├── 01-setup-and-reproduction.md
│   ├── 02-ticket-04-case-study.md
│   ├── 03-zendesk-field-guide.md
│   └── 04-next-exercises.md
└── screenshots/
    ├── README.md
    ├── 01-agent-workspace.webp
    ├── 02-ticket-fields-and-composer.webp
    ├── 03-help-center-preview.webp
    ├── 04-request-form-unavailable.webp
    ├── 05-admin-settings-access-denied.webp
    ├── 06-activation-confirmed.webp
    ├── 07-emily-ticket-04.webp
    ├── 08-agent-dashboard.webp
    └── 09-email-notification.webp
```

## Skills Demonstrated

**Zendesk navigation · Help Center activation · End-user simulation · Ticket intake · Initial triage · Ownership and assignment · Ticket field interpretation · ITSM/ITIL fundamentals · Ticket-queue troubleshooting · Evidence-based technical documentation**

## Notes on Reproducibility and Privacy

- The trial UI, permissions, menu names, and available features can change; the instructions describe what I encountered in **October 2026**.
- Sample requester names and support issues are **fictional**. Do not include real employee information, passwords, tokens, or organization-specific private tickets in a public repository.
- Screenshots are provided for evidence of the workflow; the browser chrome was cropped to minimize unnecessary account details.
- No enterprise infrastructure, Entra ID production tenancy, or employer Zendesk instance was accessed.
