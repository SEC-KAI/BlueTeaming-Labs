# Confirmed vs. Unconfirmed Findings

This file keeps the project evidence-driven and prevents alerts or interpretation from being presented as stronger facts than the available evidence supports.

## Confirmed / Supported by the Supplied Scenario and Screenshots

### Reconnaissance
- Multiple blocked connections probed internal services across ports including `21`, `22`, `23`, `445`, and `3389`.
- `203.0.113.45` generated **279 blocked firewall events**, the largest count shown in the investigation.
- `10.0.0.20` (`FINANCE-SRV1`) was identified as the internal host targeted by the scans.
- The suspicious source also had allowed connections in the firewall logs.

### VPN Authentication
- `203.0.113.45` generated **118 failed VPN authentication attempts** in the lab investigation.
- The targeted VPN username was `svc_backup`.
- The failures were followed by successful VPN authentication.
- The successful VPN login resulted in `10.8.0.23` being assigned as the internal VPN address.

### Internal Activity
- Activity from the compromised VPN context communicated with multiple internal systems.
- Internal activity included `SSH/22`, `SMB/445`, and `RDP/3389`.
- The port used for lateral SMB attempts was `445`.
- IDS alerts included possible SSH scanning, possible MS-SMB lateral movement, and possible RDP brute force.
- The supplied lab analysis concluded that SMB activity resulted in lateral movement.

### Command and Control
- `10.0.0.60` (`WORKSTATION-60`) repeatedly generated `ET TROJAN Possible C2 Beaconing` alerts.
- The C2 destination was `198.51.100.77:4444`.
- The alerts repeated across multiple days.
- The lab material includes an aggregated view showing **80 C2 beaconing alerts** for the investigated host.

### Potential Data Exfiltration
- `10.0.0.51` (`APP-WEB-01`) generated repeated outbound connections to `198.51.100.77` over ports `80` and `8080`.
- IDS telemetry generated repeated `ET INFO Possible HTTP POST Large Upload` alerts.
- Those alerts were classified as `Potential Data Exfiltration`.
- Visible events occurred on **September 27–28, 2025**.
- The supplied analysis concluded that there was evidence of **data-exfiltration attempts**.

---

## Not Confirmed by the Available Evidence

The following points are not established and should not be presented as facts:

- the exact data that may have been exfiltrated
- the exact data volume successfully transferred
- whether `198.51.100.77` received usable stolen data
- whether containment/remediation was completed after the investigation
- one single definitive destination host for the successful SMB lateral-movement step

---

## Alerts That Should Not Be Presented as Confirmed Attack Steps

The IDS data also contained signatures such as:

- `Possible SQL Injection`
- `Suspicious HTTP`
- `Possible Benign Scan`

These are alerts/signatures, not proof that the corresponding technique succeeded. The supplied material does not establish successful SQL injection as part of the confirmed incident chain, so this portfolio does not claim it.

---

## Wording Standard Used in This Project

Use:

- **supports**
- **indicates**
- **observed**
- **the IDS classified**
- **the supplied lab analysis concluded**
- **potential data exfiltration / exfiltration attempts**

Avoid unsupported claims such as:

- "all company data was stolen"
- "SQL injection compromised the server"
- "the attacker exfiltrated X GB"
- "the external server definitely received the stolen data"
