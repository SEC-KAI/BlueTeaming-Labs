# MITRE ATT&CK Mapping

This mapping is intentionally conservative. Techniques are assigned only where the documented behavior provides a reasonable direct match. Where the logs establish a tactic but not a specific ATT&CK technique, the technique is left unassigned instead of guessed.

| Observed Behavior | ATT&CK Tactic | Technique | Confidence / Note |
|---|---|---|---|
| `203.0.113.45` probing multiple exposed service ports and targeting `10.0.0.20` | Reconnaissance | **T1595 — Active Scanning** | High. The firewall shows repeated external probing before access. |
| Repeated VPN authentication failures from `203.0.113.45` against `svc_backup` | Credential Access | **T1110 — Brute Force** | High. The source explicitly treats the pattern as VPN brute-force / credential access. Sub-technique not assigned because the exact guessing method is not documented. |
| Successful VPN authentication followed by assignment of `10.8.0.23` | Initial Access | **T1078 — Valid Accounts** | High. Access occurred using a successfully authenticated account. |
| Internal probing over SSH, SMB, and RDP | Discovery | **T1046 — Network Service Discovery** | High. The compromised internal context probed services across internal systems. |
| SMB-related exploitation used for movement between internal systems | Lateral Movement | **T1210 — Exploitation of Remote Services** | Medium. The supplied lab analysis states that the SMB service was exploited and lateral movement occurred; the exact exploit/mechanism is not exposed. |
| `10.0.0.60` repeatedly beaconing to `198.51.100.77:4444` | Command and Control | **Technique not assigned** | The tactic is supported, but the logs do not reveal the application protocol or implementation required to confidently select a specific ATT&CK technique. |
| `10.0.0.51` communicating with `198.51.100.77` over 80/8080 while IDS alerts classify large HTTP POST uploads as Potential Data Exfiltration | Exfiltration | **Technique not assigned** | The tactic is supported as an exfiltration attempt, but the available evidence does not establish whether the transfer was over the C2 channel, another web service, or another exact ATT&CK technique. |

## Attack Chain View

```text
Reconnaissance
T1595 Active Scanning
        ↓
Credential Access
T1110 Brute Force
        ↓
Initial Access
T1078 Valid Accounts
        ↓
Discovery
T1046 Network Service Discovery
        ↓
Lateral Movement
T1210 Exploitation of Remote Services
        ↓
Command and Control
Technique intentionally not over-specified
        ↓
Exfiltration
Technique intentionally not over-specified
```

## Why Some Techniques Are Not Assigned

The IDS tells us that C2 beaconing occurred over TCP/4444, but a TCP port alone does not establish the application-layer protocol or exact C2 implementation.

Likewise, the IDS identified repeated large HTTP POST uploads classified as Potential Data Exfiltration, but the available source does not prove whether those uploads used the same C2 channel or which exact ATT&CK exfiltration mechanism applied.

Leaving those technique fields unassigned is more accurate than forcing a mapping unsupported by the evidence.
