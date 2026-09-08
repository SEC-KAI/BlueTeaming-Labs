# MITRE ATT&CK Mapping

This mapping distinguishes behaviors demonstrated in the labs from techniques that would only apply if malicious intent were confirmed.

| Observed / Analyzed Behavior | ATT&CK Technique | Why It Fits | Confidence |
|---|---|---|---|
| Network/port scanning | **T1046 - Network Service Discovery** | Probing ports/services can identify reachable services after access to a network. | High for the lab behavior |
| External active scanning concept | **T1595 - Active Scanning** | Reconnaissance conducted against target infrastructure before access can fall under Active Scanning. | Context dependent |
| ARP cache poisoning / MITM | **T1557.002 - ARP Cache Poisoning** | Crafted ARP information can redirect local traffic through an adversary-controlled host. | High for the authorized ARP lab concept |
| DNS used for C2-style communication | **T1071.004 - DNS** | Adversaries can encode C2 traffic within DNS queries/responses. | Technique studied; not every DNS anomaly is C2 |
| ICMP used as a command channel | **T1095 - Non-Application Layer Protocol** | ICMP can carry command-and-control data outside normal application protocols. | Technique studied |
| FTP used by an adversary | **T1071.002 - File Transfer Protocols** | FTP can be abused for command/control or data movement. | Context dependent; FTP screenshot mainly shows auth failures |
| HTTP/HTTPS C2 | **T1071.001 - Web Protocols** | Web protocols are frequently used for C2 because they blend with normal traffic. | Applicable to C2 hypothesis, not proven by one HTTP request |
| Encrypted C2 channel | **T1573 - Encrypted Channel** | An adversary may encrypt command traffic to conceal contents. | Context dependent; TLS lab itself is benign protocol analysis |

## Mapping Notes

- A protocol is not malicious simply because ATT&CK contains a technique that uses it.
- The mapping is strongest when the observed behavior, timing, endpoint context, and intent align.
- TLS traffic in this project demonstrates analysis of encrypted communications; it is not presented as proof that the captured TLS session was malicious.
- The HTTP screenshot containing `beacon.dll` is not mapped as confirmed C2 on filename alone.
