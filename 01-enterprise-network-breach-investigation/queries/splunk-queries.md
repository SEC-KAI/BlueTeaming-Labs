# Splunk Investigation Queries

The lab states that the network logs were pre-ingested into:

```spl
index="network_logs"
```

The source material available for this portfolio does **not** document the exact Splunk field extractions. To avoid inventing field names, the searches below use `_raw`, `rex`, and text matching where possible.

Replace `[REDACTED]` only with values verified from your own original lab evidence.

---

## 1. Review Available Network Logs

```spl
index="network_logs"
| head 50
```

Purpose: inspect the ingested raw format before building more specific searches.

---

## 2. Find Blocked Firewall Events

```spl
index="network_logs" "BLOCK" "TCP"
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

Purpose: identify the source responsible for the largest number of blocked requests.

---

## 4. Pivot on the Suspicious External Source

```spl
index="network_logs" "[REDACTED]"
| sort _time
| table _time _raw
```

Purpose: review all available activity involving the suspicious source across the indexed data.

---

## 5. Check for Allowed Firewall Traffic Involving the Suspicious Source

```spl
index="network_logs" "ALLOW" "[REDACTED]"
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

Purpose: identify unusually high authentication-failure sources.

---

## 8. Follow the Suspicious VPN Source from Failure to Success

```spl
index="network_logs" "[REDACTED]" ("FAIL" OR "SUCCESS")
| sort _time
| table _time _raw
```

Purpose: identify a failed-to-successful authentication sequence and the assigned internal VPN address.

---

## 9. Review Internal Activity from the Compromised VPN Context

```spl
index="network_logs" "[REDACTED]" "ALLOW"
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

Purpose: isolate IDS events related to SMB lateral movement.

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

## 14. Count C2 Beaconing Events

```spl
index="network_logs" "Possible C2 Beaconing"
| stats count
```

To focus on a verified host or destination:

```spl
index="network_logs" "Possible C2 Beaconing" "[REDACTED]"
| stats count
```

---

## 15. Hunt for TCP/4444 Activity

```spl
index="network_logs" ":4444"
| sort _time
| table _time _raw
```

Purpose: review activity associated with the port used by the C2 alerts in this scenario.

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

## 18. Review Large Upload Activity on Ports 80 and 8080

```spl
index="network_logs" "Possible HTTP POST Large Upload" (":80" OR ":8080")
| sort _time
| table _time _raw
```

---

## 19. Reconstruct the Full Timeline for a Verified Compromised Host

```spl
index="network_logs" "[REDACTED]"
| sort _time
| table _time _raw
```

Then review the sequence for:

1. authentication/access
2. internal scanning
3. SMB/RDP/SSH activity
4. C2 beaconing
5. large outbound POST events

---

## Query Design Note

These searches are intentionally conservative. The available lab source confirms the Splunk index name but does not expose the exact field extractions configured in that Splunk instance. Using `_raw` prevents this portfolio from pretending fields such as `src_ip`, `dest_ip`, or `action` were already available when that was not documented.
