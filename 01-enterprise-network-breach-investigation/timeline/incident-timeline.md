# Incident Timeline

This timeline contains only timestamps and activity visible in the supplied scenario material. Redacted source/destination values remain redacted.

| Date / Time | Data Source | Observed Activity | Interpretation |
|---|---|---|---|
| 2025-08-26 12:12:47 | Firewall | Blocked TCP probe toward port 21 | External reconnaissance sample |
| 2025-08-27 11:56:20 | Firewall | Blocked TCP probe toward `10.0.0.50:22` | SSH probing |
| 2025-08-27 22:52:00 | Firewall | Blocked TCP probe toward port 445 | SMB probing |
| 2025-08-28 10:00:00 | Firewall | Blocked TCP probe toward port 4444 | Suspicious service probing |
| 2025-08-28 10:02:30 | Firewall | Blocked TCP probe toward port 22 | Continued reconnaissance |
| 2025-09-03 02:19:00 | VPN Auth | Failed VPN login against redacted service account | Credential attack begins in visible sequence |
| 2025-09-03 02:19:10 | VPN Auth | Failed VPN login | Repeated authentication failure |
| 2025-09-03 02:19:20 | VPN Auth | Failed VPN login | Repeated authentication failure |
| 2025-09-03 02:19:30 | VPN Auth | Failed VPN login | Repeated authentication failure |
| 2025-09-03 02:19:40 | VPN Auth | Successful VPN login; internal IP assigned | Successful authenticated access |
| 2025-09-03 02:19:50 | VPN Auth | Additional successful login event | Continued authenticated activity |
| 2025-09-05 06:00:00 | Firewall / IDS | Internal connection to SSH/22; IDS possible SSH scan | Internal reconnaissance |
| 2025-09-05 06:10:00 | Firewall / IDS | Internal connection to SMB/445; IDS possible MS-SMB lateral movement | Lateral-movement activity |
| 2025-09-05 06:20:00 | Firewall / IDS | Internal connection to SSH/22; IDS possible SSH scan | Continued internal reconnaissance |
| 2025-09-05 06:30:00 | IDS | Possible RDP brute force to 3389 | Internal credential/access attempt |
| 2025-09-05 06:40:00 | Firewall | Internal RDP/3389 connection | Continued internal activity |
| 2025-09-05 07:20:00 | IDS | Possible RDP brute force | Continued internal access attempts |
| 2025-09-05 07:30:00 | Firewall / IDS | SSH activity / possible SSH scan | Continued internal reconnaissance |
| 2025-09-11 01:00:00 | IDS | `ET TROJAN Possible C2 Beaconing` to TCP/4444 | C2 activity identified |
| 2025-09-11 07:00:00 | IDS | Repeated C2 beaconing to TCP/4444 | Recurring C2 pattern |
| 2025-09-11 13:00:00 | IDS | Repeated C2 beaconing to TCP/4444 | Recurring C2 pattern |
| 2025-09-12 13:00:00 | IDS | Repeated C2 beaconing | Persistence of C2 activity |
| 2025-09-12 19:00:00 | IDS | Repeated C2 beaconing | Persistence of C2 activity |
| 2025-09-13 01:00:00 | IDS | Repeated C2 beaconing | Persistence of C2 activity |
| 2025-09-13 07:00:00 | IDS | Repeated C2 beaconing | Persistence of C2 activity |
| 2025-09-27 07:00:00 | IDS | Possible HTTP POST Large Upload to 8080 | Potential data exfiltration |
| 2025-09-27 11:00:00 | IDS | Possible HTTP POST Large Upload to 8080 | Potential data exfiltration |
| 2025-09-27 15:00:00 | IDS | Possible HTTP POST Large Upload to 8080 | Potential data exfiltration |
| 2025-09-28 07:00:00 | IDS | Possible HTTP POST Large Upload to 80 | Potential data exfiltration |
| 2025-09-28 11:00:00 | IDS | Possible HTTP POST Large Upload to 8080 | Potential data exfiltration |
| 2025-09-28 15:00:00 | IDS | Possible HTTP POST Large Upload to 80 | Potential data exfiltration |
| 2025-09-28 19:00:00 | IDS | Possible HTTP POST Large Upload to 8080 | Potential data exfiltration |

## Phase Summary

```text
Aug 26–28      External reconnaissance visible in firewall logs
Sep 3          VPN failures → successful authentication
Sep 5          Internal SSH / SMB / RDP activity and lateral movement
Sep 11 onward  Recurring C2 beaconing over TCP/4444
Sep 27–28      Large outbound HTTP POST alerts / potential exfiltration
```

## Timeline Limitations

- The scenario contains additional events outside the rows above; this file highlights the visible timestamps that support the main attack chain.
- Several IP addresses and usernames are redacted in the available portfolio source.
- The timeline does not invent missing values.
