# Investigation Methodology

## Objective

Determine whether abnormal perimeter activity represented a successful breach and reconstruct the incident using the available firewall, IDS, and VPN authentication telemetry.

## Data Sources

| Source | Purpose |
|---|---|
| `firewall.log` | Identify blocked/allowed network activity and pivot between external and internal communication |
| `ids_alerts.log` | Validate suspicious behavior using IDS signatures and classifications |
| `vpn_auth.log` | Analyze failed/successful authentication and VPN address assignment |
| Splunk `index="network_logs"` | Search and correlate the same log data at scale |

## Method

### 1. Establish Perimeter Baseline
- Inspect the first records in each log.
- Understand timestamp format, source/destination formatting, event status, and alert wording.
- Review the provided asset inventory before interpreting internal IP activity.

### 2. Identify Reconnaissance
- Filter firewall events for `BLOCK`.
- Count source IPs associated with blocked requests.
- Identify the source producing the highest volume.
- Examine the destination ports and internal targets contacted by that source.

### 3. Check Whether the Suspicious Source Was Ever Allowed
- Pivot on the suspicious source in `firewall.log`.
- Filter for `ALLOW`.
- Determine whether perimeter controls permitted any connections associated with the source.

### 4. Investigate VPN Authentication
- Count failed VPN authentication attempts by source.
- Pivot on the suspicious source.
- Look for repeated failures followed by a successful login.
- Track the internal VPN address assigned after successful authentication.

### 5. Follow the Compromised Internal Context
- Search firewall activity involving the assigned internal context.
- Identify internal destinations and service ports.
- Focus on patterns involving SSH, SMB, and RDP.

### 6. Validate Internal Movement with IDS Telemetry
- Pivot on the compromised internal source in `ids_alerts.log`.
- Review alert messages, priorities, destinations, and service ports.
- Separate scanning alerts from evidence the lab associates with successful lateral movement.

### 7. Hunt for C2 Activity
- Search IDS telemetry for `C2` / beaconing alerts.
- Identify repeated source/destination patterns.
- Review the destination port and recurrence over time.
- Aggregate alert counts for the infected host.

### 8. Investigate Possible Exfiltration
- Review internal-to-external connections from compromised systems.
- Inspect IDS telemetry for large outbound upload alerts.
- Correlate destination, port, timing, and alert classification.
- Avoid calling an event confirmed data theft solely because the transfer is large.

### 9. Reconstruct the Attack Chain
Correlate the evidence chronologically:

```text
Reconnaissance
  → VPN credential attack
  → Successful login
  → Internal reconnaissance
  → Lateral movement
  → C2 beaconing
  → Potential data exfiltration
```

### 10. Document Confidence and Limitations
- State what the logs directly support.
- Keep IDS signatures separate from confirmed successful actions.
- Use verified IPs/usernames only when they are supported by the completed lab or supplied screenshots.
- Use "potential exfiltration" when the evidence shows large outbound upload alerts but not the transferred data contents.
