# SOC Incident Report

## Incident Title
Enterprise Network Breach — VPN Compromise, Lateral Movement, C2 and Potential Exfiltration

## Environment
Initech Corp — simulated financial-services network used in an authorized training environment.

## Data Reviewed
- Firewall telemetry
- IDS/WAF telemetry
- VPN authentication logs
- Splunk-accessible copy of the same network logs

## Incident Summary

Review of approximately one month of perimeter telemetry identified a connected attack sequence beginning with external reconnaissance and progressing into authenticated VPN access, internal reconnaissance, SMB-related lateral movement, recurring command-and-control traffic, and potential outbound data exfiltration.

The investigation began with blocked firewall traffic. One redacted external source generated 279 blocked connection attempts, the highest count shown in the supplied investigation. The source probed multiple service ports and was also associated with allowed traffic.

VPN authentication logs then showed 118 failures associated with the suspicious source, followed by successful authentication and assignment of an internal VPN address. Activity from the compromised internal context later reached multiple internal systems over SSH, SMB, and RDP.

IDS telemetry generated repeated alerts for SSH scanning, SMB lateral movement, and RDP brute force. The supplied lab analysis concluded that SMB activity resulted in lateral movement.

Later, IDS telemetry repeatedly identified possible C2 beaconing over TCP/4444 from a compromised internal host to an external destination. The final stage of the investigation identified repeated large outbound HTTP POST uploads over ports 80 and 8080, which the IDS classified as Potential Data Exfiltration.

## Incident Classification

**True Positive — Confirmed breach in the training scenario**

Supported by:
- successful VPN authentication after a high-volume failure pattern
- internal activity following the successful login
- lateral-movement evidence identified in IDS telemetry
- recurring C2 beaconing alerts
- repeated potential-exfiltration alerts

## Scope

### Known
- Perimeter/VPN access was successfully obtained.
- Internal systems were probed after access.
- The supplied analysis concluded lateral movement occurred.
- At least one internal host generated recurring C2 alerts.
- At least one compromised host generated potential-exfiltration alerts.

### Redacted / Not Recoverable from the Supplied Portfolio Source
- primary malicious external IP
- main targeted VPN username
- exact assigned VPN IP after compromise
- exact C2 source host
- exact C2 destination IP
- exact exfiltration source host

## Attack Progression

1. External reconnaissance against exposed/internal-facing services.
2. High-volume VPN authentication failures.
3. Successful VPN login.
4. Internal address assignment.
5. Internal reconnaissance over SSH/SMB/RDP.
6. SMB-related lateral movement.
7. Recurring C2 beaconing over TCP/4444.
8. Large outbound HTTP POST activity classified as potential data exfiltration.

## Potential Impact

The observed activity created risk to:
- user/service account credentials
- internal server/workstation access
- sensitive finance/file-server resources
- continued attacker access through C2
- confidentiality of data potentially transferred externally

The exact data accessed or transferred is not documented in the available source and is therefore not claimed.

## Recommended Response

1. Revoke the compromised VPN session and reset affected credentials.
2. Isolate systems associated with lateral movement and C2 activity.
3. Block confirmed malicious external infrastructure.
4. Restrict unauthorized outbound TCP/4444 and review egress controls.
5. Investigate SMB/RDP/SSH activity across all internal assets during the incident window.
6. Hunt across the environment for the same C2 and HTTP POST patterns.
7. Preserve relevant logs and endpoint evidence.
8. Determine whether sensitive files were accessed before the outbound upload activity.
9. Rotate credentials exposed to affected systems where appropriate.
10. Verify containment before returning systems to normal operation.

## Analyst Conclusion

The event should not be treated as a collection of independent alerts. The firewall, VPN, IDS, and outbound-network evidence forms a coherent sequence consistent with a successful perimeter breach that progressed into internal activity, lateral movement, C2 communication, and attempted data exfiltration.

The portfolio intentionally avoids reconstructing redacted values or claiming confirmed data theft where the evidence only supports potential exfiltration attempts.
