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

The investigation began with blocked firewall traffic. `203.0.113.45` generated 279 blocked connection attempts, the highest count shown in the investigation, and `10.0.0.20` was identified as the primary internal host targeted by scans. The source probed multiple service ports and was also associated with allowed traffic.

VPN authentication logs then showed 118 failures associated with `203.0.113.45` against the `svc_backup` account, followed by successful authentication and assignment of `10.8.0.23` as an internal VPN address. Activity from the compromised internal context later reached multiple internal systems over SSH, SMB, and RDP.

IDS telemetry generated repeated alerts for SSH scanning, SMB lateral movement, and RDP brute force. The supplied lab analysis concluded that SMB activity resulted in lateral movement over TCP/445.

Later, IDS telemetry repeatedly identified C2 beaconing from `10.0.0.60` to `198.51.100.77:4444`. The final stage of the investigation identified repeated outbound connections from `10.0.0.51` to `198.51.100.77` over ports 80 and 8080, alongside large HTTP POST upload alerts classified by the IDS as Potential Data Exfiltration.

## Incident Classification

**True Positive — Confirmed breach in the training scenario**

Supported by:
- successful VPN authentication after a high-volume failure pattern
- internal activity following the successful login
- lateral-movement evidence identified in IDS telemetry
- recurring C2 beaconing alerts
- repeated potential-exfiltration alerts

## Scope

### Verified
- Reconnaissance source: `203.0.113.45`
- Primary internal scan target: `10.0.0.20` (`FINANCE-SRV1`)
- Targeted VPN account: `svc_backup`
- Internal VPN address assigned after successful login: `10.8.0.23`
- Lateral SMB attempt port: `445`
- C2 beaconing host: `10.0.0.60` (`WORKSTATION-60`)
- C2 destination: `198.51.100.77:4444`
- Host showing exfiltration attempts: `10.0.0.51` (`APP-WEB-01`)
- Exfiltration-associated destination: `198.51.100.77` over ports `80/8080`

### Not Established
- the exact data accessed or transferred
- the exact volume successfully transferred
- whether the external destination received usable stolen data
- whether containment/remediation was completed
- one definitive destination host for the successful SMB lateral-movement step

## Attack Progression

1. External reconnaissance against exposed/internal-facing services.
2. High-volume VPN authentication failures against `svc_backup`.
3. Successful VPN login from the investigated attack path.
4. `10.8.0.23` assigned as the internal VPN address.
5. Internal reconnaissance over SSH/SMB/RDP.
6. SMB-related lateral movement over TCP/445.
7. Recurring C2 beaconing from `10.0.0.60` to `198.51.100.77:4444`.
8. Large outbound HTTP POST activity associated with `10.0.0.51` and classified as potential data exfiltration.

## Potential Impact

The observed activity created risk to:
- user/service account credentials
- internal server/workstation access
- sensitive finance/file-server resources
- continued attacker access through C2
- confidentiality of data potentially transferred externally

The exact data accessed or transferred is not documented in the available evidence and is therefore not claimed.

## Recommended Response

1. Revoke the compromised VPN session and reset the `svc_backup` credentials.
2. Isolate systems associated with lateral movement and C2 activity, including `10.0.0.60` and `10.0.0.51` pending scoping.
3. Block confirmed malicious external infrastructure, including `198.51.100.77`, where appropriate.
4. Restrict unauthorized outbound TCP/4444 and review egress controls.
5. Investigate SMB/RDP/SSH activity across all internal assets during the incident window.
6. Hunt across the environment for the same C2 and HTTP POST patterns.
7. Preserve relevant logs and endpoint evidence.
8. Determine whether sensitive files were accessed before the outbound upload activity.
9. Rotate credentials exposed to affected systems where appropriate.
10. Verify containment before returning systems to normal operation.

## Analyst Conclusion

The event should not be treated as a collection of independent alerts. The firewall, VPN, IDS, and outbound-network evidence forms a coherent sequence consistent with a successful perimeter breach that progressed into internal activity, lateral movement, C2 communication, and attempted data exfiltration.

The portfolio uses the verified IP addresses and username from the completed lab while still avoiding unsupported claims about the contents or successful receipt of potentially exfiltrated data.
