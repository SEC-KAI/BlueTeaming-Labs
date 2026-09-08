# Incident Timeline

This timeline uses only timestamps and activity supported by the supplied lab scenario and screenshots.

| Date / Time | Data Source | Observed Activity | Interpretation |
|---|---|---|---|
| 2025-08-26 12:12:47 | Firewall | Blocked TCP probe toward port 21 | External reconnaissance sample |
| 2025-08-27 11:56:20 | Firewall | Blocked TCP probe toward `10.0.0.50:22` | SSH probing |
| 2025-08-27 22:52:00 | Firewall | Blocked TCP probe toward port 445 | SMB probing |
| 2025-08-28 10:00:00 | Firewall | Blocked TCP probe toward port 4444 | Suspicious service probing |
| 2025-08-28 10:02:30 | Firewall | Blocked TCP probe toward port 22 | Continued reconnaissance |
| 2025-09-03 02:19:00 | VPN Auth | `203.0.113.45` failed VPN login against `svc_backup` | Credential attack begins in visible sequence |
| 2025-09-03 02:19:10 | VPN Auth | Repeated failed login from `203.0.113.45` | Continued authentication attack |
| 2025-09-03 02:19:20 | VPN Auth | Repeated failed login from `203.0.113.45` | Continued authentication attack |
| 2025-09-03 02:19:30 | VPN Auth | Repeated failed login from `203.0.113.45` | Continued authentication attack |
| 2025-09-03 02:19:40 | VPN Auth | Successful login; `10.8.0.23` assigned | Successful authenticated access |
| 2025-09-03 02:19:50 | VPN Auth | Additional successful login event | Continued authenticated activity |
| 2025-09-05 06:00:00 | Firewall / IDS | Activity from `10.8.0.23` to SSH/22; IDS possible SSH scan | Internal reconnaissance |
| 2025-09-05 06:10:00 | Firewall / IDS | Activity from `10.8.0.23` to SMB/445; IDS possible MS-SMB lateral movement | Lateral-movement activity |
| 2025-09-05 06:20:00 | Firewall / IDS | Activity from `10.8.0.23` to SSH/22; IDS possible SSH scan | Continued internal reconnaissance |
| 2025-09-05 06:30:00 | IDS | Possible RDP brute force to 3389 | Internal credential/access attempt |
| 2025-09-05 06:40:00 | Firewall | Internal RDP/3389 connection | Continued internal activity |
| 2025-09-05 07:20:00 | IDS | Possible RDP brute force | Continued internal access attempts |
| 2025-09-05 07:30:00 | Firewall / IDS | SSH activity / possible SSH scan | Continued internal reconnaissance |
| 2025-09-11 01:00:00 | IDS | `10.0.0.60:30000 -> 198.51.100.77:4444` — `ET TROJAN Possible C2 Beaconing` | C2 activity identified |
| 2025-09-11 07:00:00 | IDS | `10.0.0.60:30001 -> 198.51.100.77:4444` | Recurring C2 pattern |
| 2025-09-11 13:00:00 | IDS | `10.0.0.60:30002 -> 198.51.100.77:4444` | Recurring C2 pattern |
| 2025-09-11 19:00:00 | IDS | `10.0.0.60:30003 -> 198.51.100.77:4444` | Recurring C2 pattern |
| 2025-09-12 01:00:00 | IDS | `10.0.0.60:30004 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-12 07:00:00 | IDS | `10.0.0.60:30005 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-12 13:00:00 | IDS | `10.0.0.60:30006 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-12 19:00:00 | IDS | `10.0.0.60:30007 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-13 01:00:00 | IDS | `10.0.0.60:30008 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-13 07:00:00 | IDS | `10.0.0.60:30009 -> 198.51.100.77:4444` | Persistent C2 activity |
| 2025-09-27 07:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` to external infrastructure over 8080 | Potential data exfiltration |
| 2025-09-27 11:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 8080 | Potential data exfiltration |
| 2025-09-27 15:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 8080 | Potential data exfiltration |
| 2025-09-28 07:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 80 | Potential data exfiltration |
| 2025-09-28 11:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 8080 | Potential data exfiltration |
| 2025-09-28 15:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 80 | Potential data exfiltration |
| 2025-09-28 19:00:00 | IDS | Large HTTP POST upload associated with `10.0.0.51` over 8080 | Potential data exfiltration |

## Verified Incident Indicators

- Reconnaissance source: `203.0.113.45`
- Internal scan target: `10.0.0.20`
- Targeted VPN username: `svc_backup`
- Assigned VPN IP: `10.8.0.23`
- Lateral SMB port: `445`
- C2 source host: `10.0.0.60`
- C2 destination: `198.51.100.77:4444`
- Exfiltration-attempt host: `10.0.0.51`

## Phase Summary

```text
Aug 26–28      External reconnaissance; 203.0.113.45 identified as top source
Sep 3          VPN failures against svc_backup → success → 10.8.0.23
Sep 5          Internal SSH / SMB / RDP activity and lateral movement
Sep 11 onward  10.0.0.60 → 198.51.100.77:4444 recurring C2 beaconing
Sep 27–28      10.0.0.51 associated with large outbound HTTP POST alerts
```

## Timeline Limitations

- The scenario contains additional events outside the rows above; this file highlights the timestamps that support the main attack chain.
- The screenshots verify the incident identities and hosts used in the project.
- The evidence does not establish the exact contents or successful receipt of data associated with the potential-exfiltration stage.
