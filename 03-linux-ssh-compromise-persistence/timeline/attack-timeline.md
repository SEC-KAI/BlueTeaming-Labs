# Attack Timeline

The recovered case information confirms the **order of events**, but not reliable exact timestamps for every step. This timeline therefore uses sequence numbers rather than invented times.

| Sequence | Event | Confirmed Detail | Analyst Interpretation |
|---:|---|---|---|
| 1 | Repeated SSH authentication failures | Source `10.14.94.82` | Credential-access activity against SSH |
| 2 | Successful SSH login | Account `jack-brown` | Existing account was successfully accessed after the failures |
| 3 | Privilege activity | `sudo` used by/after the compromised account | Post-login privilege escalation activity |
| 4 | New account created | `ssh-remote` | New account provided an additional access/persistence path |
| 5 | Crontab modified | Cron persistence identified | Scheduled persistence established |
| 6 | Outbound communication | `10.10.33.31:7654` | Network communication associated with the post-compromise sequence |
| 7 | Incident decision | True Positive / Escalate | Correlated sequence supported confirmed compromise in the lab scenario |

## Investigation Flow

```text
10.14.94.82
   │
   ├── repeated failed SSH logins
   │
   └── successful login → jack-brown
                         │
                         ├── sudo activity
                         │
                         ├── create ssh-remote
                         │
                         └── modify crontab
                                  │
                                  └── outbound → 10.10.33.31:7654
```

## What Is Not Claimed

- Exact timestamps are not supplied because they were not recovered with sufficient confidence.
- The initial credential-acquisition method is not claimed beyond the observed repeated failures and later successful login.
- The protocol used to communicate with `10.10.33.31:7654` is not identified in the recovered case information.
- The repository does not claim the outbound connection successfully transferred data or established a particular C2 framework.
