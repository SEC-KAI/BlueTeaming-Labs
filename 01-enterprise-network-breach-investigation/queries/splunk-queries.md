# Splunk Investigation Queries

The lab states that the perimeter logs are pre-ingested into:

```spl
index="network_logs"
```

These searches deliberately use `_raw` or simple literal matching unless the field extraction is demonstrated by the log format. This avoids pretending that custom fields already existed in Splunk.

## Verified Indicators Used

| Indicator | Value |
|---|---|
| Reconnaissance source | `203.0.113.45` |
| Scan target | `10.0.0.20` |
| VPN username | `svc_backup` |
| Assigned VPN IP | `10.8.0.23` |
| C2 source host | `10.0.0.60` |
| C2 destination | `198.51.100.77` |
| Exfiltration-attempt host | `10.0.0.51` |

---

## 1. Review the Network Log Index

```spl
index="network_logs"
| sort _time
| table _time _raw
```

Purpose: understand the available log formats and event types before narrowing the investigation.

---

## 2. Find Blocked Firewall Traffic

```spl
index="network_logs" "BLOCK"
| sort _time
| table _time _raw
```

Purpose: isolate blocked perimeter traffic associated with reconnaissance.

---

## 3. Count Blocked Firewall Events by Source IP

The firewall format shown in the lab places the source as the token before `->`.

```spl
index="network_logs" "BLOCK" "TCP"
| rex field=_raw "BLOCK\s+TCP\s+(?<src_ip>[^: ]+):\d+\s+->"
| stats count by src_ip
| sort - count
```

Purpose: identify the source responsible for the largest number of blocked requests. The completed lab identifies `203.0.113.45` as the top reconnaissance source.

---

## 4. Pivot on the Suspicious External Source

```spl
index="network_logs" "203.0.113.45"
| sort _time
| table _time _raw
```

Purpose: review all available activity involving the suspicious source across the indexed data.

---

## 5. Check for Allowed Firewall Traffic Involving the Suspicious Source

```spl
index="network_logs" "ALLOW" "203.0.113.45"
| sort _time
| table _time _raw
```

Purpose: determine whether the suspicious source was ever permitted by the firewall.

---

## 6. Find VPN Authentication Failures

```spl
index="network_logs" "FAIL"
| table _time _raw
```

Purpose: isolate failed VPN authentication events.

---

## 7. Count VPN Failures by Source IP

The VPN format shown in the lab is:

```text
TIMESTAMP SOURCE_IP USERNAME FAIL
```

A raw-field extraction can therefore be performed without assuming a predefined Splunk field:

```spl
index="network_logs" "FAIL"
| rex field=_raw "^\S+\s+\S+\s+(?<src_ip>\S+)\s+(?<username>\S+)\s+FAIL"
| stats count by src_ip
| sort - count
```

Purpose: identify unusually high authentication-failure sources. The investigation identified `203.0.113.45` with 118 failures.

---

## 8. Follow the Suspicious VPN Source from Failure to Success

```spl
index="network_logs" "203.0.113.45" ("FAIL" OR "SUCCESS")
| sort _time
| table _time _raw
```

Purpose: identify the failed-to-successful authentication sequence involving `svc_backup` and the assignment of `10.8.0.23`.

---

## 9. Review Internal Activity from the Compromised VPN Context

```spl
index="network_logs" "10.8.0.23" "ALLOW"
| sort _time
| table _time _raw
```

Purpose: follow the internal VPN context after successful authentication.

Look for activity involving:

- `:22` — SSH
- `:445` — SMB
- `:3389` — RDP

---

## 10. Find SMB Lateral-Movement Alerts

```spl
index="network_logs" "Possible MS-SMB Lateral Movement"
| sort _time
| table _time _raw
```

Purpose: isolate IDS events related to SMB lateral movement over TCP/445.

---

## 11. Find RDP Brute-Force Alerts

```spl
index="network_logs" "Possible RDP Brute Force"
| sort _time
| table _time _raw
```

---

## 12. Find SSH Scan Alerts

```spl
index="network_logs" "Possible SSH Scan"
| sort _time
| table _time _raw
```

---

## 13. Hunt for C2 Beaconing

```spl
index="network_logs" "Possible C2 Beaconing"
| sort _time
| table _time _raw
```

Purpose: identify recurring IDS C2 alerts.

---

## 14. Count C2 Beaconing Events for the Verified Host

```spl
index="network_logs" "Possible C2 Beaconing" "10.0.0.60" "198.51.100.77"
| stats count
```

Purpose: focus on the verified beaconing relationship `10.0.0.60 -> 198.51.100.77:4444`.

---

## 15. Hunt for TCP/4444 Activity

```spl
index="network_logs" ":4444" ("10.0.0.60" OR "198.51.100.77")
| sort _time
| table _time _raw
```

Purpose: review activity associated with the C2 port used in this scenario.

---

## 16. Find Potential Data-Exfiltration Alerts

```spl
index="network_logs" "Possible HTTP POST Large Upload"
| sort _time
| table _time _raw
```

---

## 17. Focus on IDS Classification for Potential Exfiltration

```spl
index="network_logs" "Potential Data Exfiltration"
| sort _time
| table _time _raw
```

---

## 18. Review Large Upload Activity Associated with the Verified Exfiltration Host

```spl
index="network_logs" "10.0.0.51" "Possible HTTP POST Large Upload" (":80" OR ":8080")
| sort _time
| table _time _raw
```

Purpose: focus on the host the completed lab identifies as showing exfiltration attempts.

---

## 19. Reconstruct the Incident Across Verified Indicators

```spl
index="network_logs" ("203.0.113.45" OR "svc_backup" OR "10.8.0.23" OR "10.0.0.60" OR "10.0.0.51" OR "198.51.100.77")
| sort _time
| table _time _raw
```

Then review the sequence for:

1. reconnaissance
2. VPN failures and successful access
3. internal scanning
4. SMB/RDP/SSH activity
5. C2 beaconing
6. large outbound POST events

---

## Query Design Note

These searches are intentionally conservative. The lab confirms the Splunk index name but does not document every field extraction configured in that instance. Using `_raw` and explicit `rex` only where the raw log format is known avoids inventing undocumented fields.
