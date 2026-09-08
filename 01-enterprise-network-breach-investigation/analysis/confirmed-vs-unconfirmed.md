# Confirmed vs. Unconfirmed Findings

This file exists to keep the project evidence-driven and prevent the portfolio from presenting inference as fact.

## Confirmed / Supported by the Supplied Scenario

### Reconnaissance
- Multiple blocked connections probed internal services across ports including `21`, `22`, `23`, `445`, and `3389`.
- One redacted external source generated **279 blocked firewall events**, the largest count shown in the investigation.
- The suspicious source also had allowed connections in the firewall logs.

### VPN Authentication
- The suspicious source generated **118 failed VPN authentication attempts**.
- The failures were followed by successful VPN authentication.
- A successful VPN login resulted in an internal VPN address being assigned.

### Internal Activity
- The compromised internal context communicated with multiple internal systems.
- Internal activity included `SSH/22`, `SMB/445`, and `RDP/3389`.
- IDS alerts included possible SSH scanning, possible MS-SMB lateral movement, and possible RDP brute force.
- The supplied lab analysis explicitly concluded that SMB activity resulted in lateral movement.

### Command and Control
- IDS telemetry repeatedly generated `ET TROJAN Possible C2 Beaconing` alerts.
- C2-related traffic used **TCP/4444**.
- The alerts repeated across multiple days.
- An aggregate alert view showed **80 C2 beaconing alerts** associated with the investigated infected host.

### Potential Data Exfiltration
- Compromised systems generated repeated outbound connections to external destinations over ports `80` and `8080`.
- IDS telemetry generated repeated `ET INFO Possible HTTP POST Large Upload` alerts.
- Those alerts were classified as `Potential Data Exfiltration`.
- Visible events occurred on **September 27–28, 2025**.
- The supplied analysis concluded that there was evidence of **data-exfiltration attempts**.

---

## Not Confirmed in the Available Portfolio Source

The following values are redacted or not established by the supplied material and should **not** be guessed:

- exact external IP responsible for the largest reconnaissance volume
- exact internal host identified as the primary scan target
- exact VPN username targeted in the main credential attack
- exact internal VPN IP assigned immediately after the successful malicious login
- exact internal host that beaconed to the C2 server
- exact external C2 IP
- exact host responsible for the exfiltration attempts
- exact data that may have been exfiltrated
- exact data volume successfully transferred
- whether the external destination received usable stolen data
- whether containment/remediation was completed after the investigation

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
- exact IPs/usernames that are redacted in the available source
