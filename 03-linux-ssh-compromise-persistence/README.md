# Linux SSH Compromise & Persistence Investigation

## Overview

This project documents a Linux incident investigation in which repeated SSH authentication failures were followed by a successful login to an existing user account, privilege-escalation activity, creation of a new account, cron-based persistence, and outbound network communication.

The purpose of the investigation was to connect separate authentication, privilege, persistence, and network events into one incident sequence rather than treating each alert in isolation.

> **Lab context:** This was completed in an authorized training/lab environment. The exact TryHackMe room name for this incident has not been confirmed from the recovered material, so it is intentionally not listed here.

## Confirmed Incident Sequence

```text
Repeated SSH failures from 10.14.94.82
        ↓
Successful SSH login to jack-brown
        ↓
sudo / privilege-escalation activity
        ↓
New account created: ssh-remote
        ↓
Crontab modified for persistence
        ↓
Outbound communication to 10.10.33.31:7654
        ↓
True Positive / Escalation
```

## Confirmed Indicators

| Type | Value | What it supports |
|---|---|---|
| Source IP | `10.14.94.82` | Repeated failed SSH authentication attempts |
| Compromised account | `jack-brown` | Successful SSH login after the failed attempts |
| Privilege activity | `sudo` | Post-login privilege-escalation activity |
| Newly created account | `ssh-remote` | Persistence / continued access |
| Persistence mechanism | Crontab modification | Scheduled persistence activity |
| Outbound destination | `10.10.33.31:7654` | Network activity associated with the persistence sequence |
| Final classification | True Positive | Combined behavior supported compromise and escalation |

## Investigation Approach

### 1. Authentication Analysis

The investigation began with repeated failed SSH logins from `10.14.94.82`. A later successful SSH login to `jack-brown` was then correlated with those failures.

The successful login was important because it changed the event from isolated failed authentication attempts into a likely account-compromise sequence requiring follow-on investigation.

### 2. Privilege Activity

After the successful login, `sudo` activity was identified under the compromised user context. The investigation therefore continued beyond authentication and into post-login actions.

### 3. Account Creation

A new account named `ssh-remote` was created after the compromise. In the context of the preceding authentication and privilege activity, this was treated as persistence-related behavior rather than normal account administration.

### 4. Cron Persistence

The investigation identified a crontab modification after the new account activity. This provided another persistence indicator and connected the compromise to a scheduled mechanism capable of recurring execution.

### 5. Network Activity

The persistence sequence was associated with outbound communication to `10.10.33.31:7654`. The exact application protocol is not confirmed in the recovered case information, so this repository does not assign one.

## Finding

The activity was classified as a **True Positive** because the events formed a coherent compromise sequence:

- repeated failed SSH authentication;
- later successful access to an existing account;
- privilege-related activity;
- creation of a new account;
- cron-based persistence; and
- outbound communication following the persistence activity.

No single event is presented as proof by itself. The conclusion is based on correlation across the full sequence.

## Recommended Response Actions

For an equivalent real-world incident, appropriate actions would include:

1. Isolate or otherwise contain the affected Linux host.
2. Disable or reset credentials for `jack-brown` and investigate the source of credential compromise.
3. Disable and review the newly created `ssh-remote` account.
4. Review authorized keys, SSH configuration, shell history, and recent authentication activity.
5. Remove unauthorized cron entries only after preserving relevant evidence.
6. Block and investigate communication with the suspicious destination where appropriate.
7. Review the system for additional persistence mechanisms, altered services, new binaries, and privilege changes.
8. Determine whether the same credentials or indicators appear on other systems.

These are response recommendations; they are not presented as actions confirmed to have been performed in the original lab.

## Evidence in This Repository

The exact original screenshots for `10.14.94.82`, `jack-brown`, `ssh-remote`, and `10.10.33.31:7654` were not recoverable from the available image library.

To avoid misrepresenting evidence, the recovered screenshots in `evidence/` are explicitly treated as **related Linux investigation evidence**. They demonstrate hands-on work with Linux authentication logs, Auditd process relationships, systemd persistence, cron execution, and reverse-shell/network behavior, but they are not claimed to be screenshots from the exact `jack-brown` incident.

See [`screenshots/README.md`](screenshots/README.md) for the evidence index and captions.

## Repository Structure

```text
03-linux-ssh-compromise-persistence/
├── README.md
├── evidence/
│   ├── authentication/
│   ├── user-activity/
│   ├── cron/
│   └── network/
├── queries/
│   └── log-analysis-commands.md
├── timeline/
│   └── attack-timeline.md
├── attack-mapping/
│   └── mitre-attck.md
└── screenshots/
    └── README.md
```

## Skills Demonstrated

- Linux security-log investigation
- SSH authentication analysis
- Alert correlation
- Privilege-escalation investigation
- Linux account-creation analysis
- Cron persistence analysis
- Auditd process analysis
- Network-event correlation
- Incident timeline reconstruction
- True-positive classification and escalation reasoning
- MITRE ATT&CK mapping with evidence-based confidence levels
