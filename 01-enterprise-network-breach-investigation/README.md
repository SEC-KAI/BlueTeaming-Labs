# Enterprise Network Breach Investigation

## Overview

This project documents a simulated SOC investigation of a perimeter breach affecting **Initech Corp**, a mid-sized financial-services environment. The investigation covered approximately one month of perimeter activity and used three primary log sources:

- `firewall.log` — allowed and blocked network connections
- `ids_alerts.log` — IDS/WAF security alerts
- `vpn_auth.log` — VPN authentication activity

The investigation was performed through **manual Linux log analysis** and could also be reproduced in **Splunk**, where the logs were pre-ingested into `index="network_logs"`.

The objective was to determine whether an external adversary successfully breached the perimeter and, if so, reconstruct the progression of the intrusion.

> **Environment note:** This was an authorized TryHackMe training scenario. No production systems were involved.

---

## Executive Summary

The investigation identified a connected sequence of malicious activity rather than isolated alerts:

```text
External Reconnaissance
        ↓
VPN Authentication Attack
        ↓
Successful VPN Login
        ↓
Internal Access
        ↓
Internal Reconnaissance
        ↓
SMB / RDP / SSH Activity
        ↓
Lateral Movement
        ↓
C2 Beaconing
        ↓
Large HTTP POST Activity
        ↓
Potential Data Exfiltration
```

A suspicious external source generated the largest volume of blocked reconnaissance activity. Pivoting into the VPN authentication logs showed repeated failed authentication attempts followed by successful authentication and assignment of an internal VPN address.

Activity from the compromised internal VPN context was then observed against internal systems over **SSH (22), SMB (445), and RDP (3389)**. IDS telemetry contained repeated SMB lateral-movement alerts, and the supplied lab analysis concluded that lateral movement occurred.

Later IDS activity showed repeated **C2 beaconing over TCP/4444** from a compromised internal host to external infrastructure. The final stage of the investigation identified repeated large outbound HTTP POST events over ports **80 and 8080**, classified by the IDS as **Potential Data Exfiltration**.

The evidence therefore supports a successful breach with internal activity, lateral movement, command-and-control communication, and **data-exfiltration attempts**. The available material does **not** establish the exact data contents transferred or prove successful data theft.

---

## Environment

The scenario provided the following asset inventory:

| IP Address | Hostname | Role | OS | Team | Criticality |
|---|---|---|---|---|---|
| `10.0.0.20` | `FINANCE-SRV1` | File / Finance Server (SMB) | Windows Server | Finance IT | High |
| `10.0.0.50` | `VPN-GW` | VPN Gateway | Linux | NetOps | Critical |
| `10.0.0.51` | `APP-WEB-01` | Internal Web / App | Linux | Apps Team | High |
| `10.0.0.60` | `WORKSTATION-60` | Employee Workstation | Windows 10 | Sales | Medium |
| `10.8.0.23` | `VPN-CLIENT-ATTK` | VPN Assigned Client (Ephemeral) | N/A | N/A | Critical |
| `10.0.1.10` | `DMZ-WEB` | DMZ Web Server | Linux | NetOps | Medium |

**Important:** The source material available for this portfolio copy redacts several solved values. This README does not infer those values from hostnames or surrounding context. See [`analysis/confirmed-vs-unconfirmed.md`](analysis/confirmed-vs-unconfirmed.md).

---

## Log Sources

### Firewall Logs
Used to identify:

- blocked reconnaissance attempts
- high-volume scanning sources
- allowed connections involving suspicious infrastructure
- internal-to-internal connections after compromise
- outbound connections associated with potential exfiltration

### IDS / WAF Logs
Used to identify:

- SSH scanning
- SMB lateral-movement alerts
- RDP brute-force activity
- possible C2 beaconing
- suspicious HTTP activity
- large HTTP POST upload alerts

### VPN Authentication Logs
Used to identify:

- repeated failed VPN authentication
- targeted account activity
- successful authentication after repeated failures
- assignment of an internal VPN address

---

## Investigation Methodology

The investigation followed a pivot-and-correlate approach:

1. **Identify the noisiest suspicious external source** in blocked firewall traffic.
2. **Pivot on the suspicious source** to determine whether any traffic was allowed.
3. **Correlate the source with VPN authentication logs** and identify failed-to-successful authentication sequences.
4. **Track the assigned internal VPN context** after successful authentication.
5. **Search internal firewall activity** for connections from the compromised context toward other assets.
6. **Validate suspicious internal movement against IDS alerts**, especially SMB/RDP/SSH behavior.
7. **Hunt for recurring outbound C2 patterns** in IDS telemetry.
8. **Review outbound activity for potential exfiltration**, focusing on repeated large HTTP POST uploads.
9. **Separate confirmed observations from unconfirmed interpretation** before producing the final incident conclusion.

The exact manual commands are documented in [`queries/linux-log-analysis.txt`](queries/linux-log-analysis.txt). Splunk equivalents that avoid assuming pre-extracted field names are in [`queries/splunk-queries.md`](queries/splunk-queries.md).

---

# Investigation

## Phase 1 — External Reconnaissance

Firewall review showed an external source probing internal systems across multiple service ports, including:

- FTP — `21`
- SSH — `22`
- Telnet — `23`
- SMB — `445`
- RDP — `3389`

Counting blocked requests showed one redacted external source producing **279 blocked events**, substantially more than the other sources shown in the supplied material.

This source became the primary pivot for the rest of the investigation.

### Analyst assessment

The combination of repeated blocked connections and probing across multiple service ports supports **reconnaissance / service discovery activity**.

---

## Phase 2 — VPN Authentication Attack

The VPN logs showed **118 failed authentication attempts** associated with the suspicious source, while the other visible sources had only isolated failures.

Filtering directly on the suspicious source showed repeated authentication failures at short intervals against a service account, followed by successful authentication.

### Observed sequence

```text
Repeated VPN failures
        ↓
Successful VPN authentication
        ↓
Internal VPN address assigned
```

### Analyst assessment

The evidence supports a VPN credential attack that progressed to successful authenticated access. The available portfolio source redacts the exact external IP, targeted username, and assigned internal IP, so those values are intentionally not reconstructed here.

---

## Phase 3 — Internal Reconnaissance

After successful VPN access, firewall activity from the compromised internal context showed connections toward multiple internal machines over:

- SSH — `22`
- SMB — `445`
- RDP — `3389`

The activity occurred across multiple internal systems rather than remaining limited to the VPN gateway.

### Analyst assessment

This supports internal service discovery and reconnaissance following initial access.

---

## Phase 4 — Lateral Movement

IDS alerts associated with the compromised internal context included:

- `ET SCAN Possible SSH Scan`
- `ET EXPLOIT Possible MS-SMB Lateral Movement`
- `ET EXPLOIT Possible RDP Brute Force`

Repeated SMB-related alerts were observed against TCP/445. The supplied lab analysis explicitly concluded that SMB exploitation enabled lateral movement.

### Analyst assessment

The evidence supports **lateral movement through SMB-related activity**. The exact destination host used for the successful movement is redacted in the supplied portfolio source and is therefore not guessed here.

---

## Phase 5 — Command and Control

IDS hunting for C2 activity returned recurring alerts similar to:

```text
ET TROJAN Possible C2 Beaconing
TCP ... -> ...:4444
```

The alerts repeated over multiple days and were associated with a compromised internal host communicating with external infrastructure over **TCP/4444**.

Alert statistics for the infected host showed repeated C2 detections, including a larger count of **80 C2 beaconing alerts** in one aggregated view.

### Analyst assessment

The recurring outbound pattern and IDS classification support **command-and-control communication**. The exact internal beaconing host and external C2 IP are redacted in the available source.

---

## Phase 6 — Potential Data Exfiltration

The final stage examined outbound traffic from compromised systems to external destinations.

Firewall activity showed repeated outbound connections to external services over ports:

- `80`
- `8080`

IDS logs then showed repeated:

```text
ET INFO Possible HTTP POST Large Upload
Classification: Potential Data Exfiltration
```

The visible exfiltration-related events occurred on **September 27–28, 2025** and repeatedly involved large HTTP POST uploads.

### Analyst assessment

The evidence supports **data-exfiltration attempts**. It does not establish:

- the exact content of the outbound data
- whether the external destination successfully received usable data
- the total amount of data transferred

For that reason, this project intentionally uses **potential data exfiltration / exfiltration attempts** rather than claiming confirmed data theft.

---

# Incident Timeline

A detailed timeline is available at:

[`timeline/incident-timeline.md`](timeline/incident-timeline.md)

High-level progression:

| Phase | Approximate Period | Key Evidence |
|---|---|---|
| Reconnaissance | Aug–Sep 2025 | High-volume blocked probing across multiple ports |
| VPN credential attack | Sep 3, 2025 | Repeated failures followed by successful VPN authentication |
| Internal reconnaissance | Sep 5, 2025 | SSH/SMB/RDP connections toward internal systems |
| Lateral movement | Sep 5, 2025 onward | Repeated SMB lateral-movement IDS alerts |
| C2 | Sep 11, 2025 onward | Repeated TCP/4444 C2 beaconing alerts |
| Exfiltration attempts | Sep 27–28, 2025 | Large outbound HTTP POST alerts on 80/8080 |

---

# Key Findings

1. A suspicious external source performed the largest volume of perimeter reconnaissance in the supplied logs.
2. The same investigation path led to repeated VPN authentication failures followed by successful VPN access.
3. The attacker obtained an internal VPN context and began interacting with internal services.
4. Internal activity targeted SSH, SMB, and RDP across multiple systems.
5. IDS evidence supported SMB-based lateral movement.
6. A compromised internal host generated recurring C2 beaconing alerts over TCP/4444.
7. Later outbound HTTP POST activity was classified by the IDS as potential data exfiltration.
8. The exact identities/IPs hidden by the supplied redactions are not reconstructed or guessed in this portfolio version.

---

# MITRE ATT&CK Mapping

See [`attack-mapping/mitre-attck.md`](attack-mapping/mitre-attck.md).

The mapping intentionally avoids assigning a specific ATT&CK technique when the available logs do not provide enough evidence to do so confidently.

---

# Recommended Response Actions

Based on the observed incident sequence, appropriate response priorities would include:

1. **Terminate and revoke the compromised VPN session.**
2. **Reset the affected account credentials** and investigate other authentication activity involving that account.
3. **Isolate compromised internal hosts** associated with lateral movement and C2 communication.
4. **Block identified malicious external infrastructure** at perimeter controls where appropriate.
5. **Restrict unauthorized outbound TCP/4444 communication** and review egress policy.
6. **Review SMB/RDP/SSH activity across the environment** to determine the full lateral-movement scope.
7. **Hunt for the same C2 and HTTP POST patterns** across other hosts.
8. **Preserve relevant firewall, IDS, VPN, and endpoint logs** for further investigation.
9. **Review the suspected exfiltration period** to determine what data, if any, left the environment.
10. **Validate containment before recovery**, ensuring attacker access and persistence paths are no longer active.

These are response recommendations based on the observed lab evidence; the supplied scenario does not document whether these remediation steps were actually carried out.

---

# Skills Demonstrated

- SOC incident investigation
- Multi-source log correlation
- Firewall log analysis
- IDS alert analysis
- VPN authentication analysis
- Splunk search and investigation
- Linux command-line log analysis
- Reconnaissance detection
- Credential-attack investigation
- Internal network reconnaissance analysis
- Lateral-movement investigation
- C2 beaconing analysis
- Potential data-exfiltration investigation
- Timeline reconstruction
- Evidence-based incident scoping
- MITRE ATT&CK mapping
- Distinguishing confirmed facts from unverified assumptions

---

# Repository Contents

```text
01-enterprise-network-breach-investigation/
├── README.md
├── analysis/
│   ├── asset-inventory.md
│   ├── confirmed-vs-unconfirmed.md
│   ├── incident-report.md
│   └── investigation-methodology.md
├── queries/
│   ├── linux-log-analysis.txt
│   └── splunk-queries.md
├── timeline/
│   └── incident-timeline.md
├── attack-mapping/
│   └── mitre-attck.md
├── evidence/
│   ├── README.md
│   ├── firewall/
│   ├── ids/
│   ├── vpn-authentication/
│   └── splunk/
└── screenshots/
    └── README.md
```

---

## Disclaimer

This project documents an **authorized cybersecurity training scenario**. IP addresses, usernames, and other values that are redacted in the available source material have deliberately been left redacted rather than inferred.
