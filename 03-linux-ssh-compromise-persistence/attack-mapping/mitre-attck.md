# MITRE ATT&CK Mapping

This mapping is intentionally conservative. Only behaviors supported by the recovered incident sequence are mapped. Where the exact technique is uncertain, the uncertainty is stated rather than guessed.

| Observed Behavior | ATT&CK Tactic | Technique | Confidence / Basis |
|---|---|---|---|
| Repeated failed SSH authentication followed by successful access | Credential Access | **T1110 – Brute Force** | **Moderate.** The pattern is consistent with brute-force/password-guessing activity, but the exact credential-acquisition method is not independently confirmed. |
| Successful use of existing `jack-brown` account | Initial Access / Persistence / Privilege Escalation / Defense Evasion | **T1078 – Valid Accounts** | **High.** A successful login to an existing account is confirmed. |
| `sudo` activity after access | Privilege Escalation | **T1548.003 – Abuse Elevation Control Mechanism: Sudo and Sudo Caching** | **High.** `sudo` activity is confirmed in the incident sequence. |
| Creation of `ssh-remote` | Persistence | **T1136.001 – Create Account: Local Account** | **High** if `ssh-remote` was created locally, which is consistent with the recovered case description. The repository does not claim a domain/directory account. |
| Crontab modification | Persistence / Privilege Escalation / Execution | **T1053.003 – Scheduled Task/Job: Cron** | **High.** Cron-based persistence is confirmed. |
| Outbound communication to `10.10.33.31:7654` | Command and Control | **Technique not assigned** | **Intentional.** The destination is confirmed, but the application protocol and C2 mechanism are not. |

## Why the Mapping Is Conservative

ATT&CK mapping should describe behavior that the evidence supports, not fill in gaps. For example, an outbound connection alone does not prove a specific application-layer protocol or C2 framework, so no such sub-technique is assigned here.
