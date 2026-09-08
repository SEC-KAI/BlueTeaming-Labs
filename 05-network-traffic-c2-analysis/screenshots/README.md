# Screenshot Evidence Index

These images were recovered from my earlier network-analysis lab conversations and renamed so their purpose is clear in the repository.

| File | Case | What it shows |
|---|---|---|
| `00-wireshark-traffic-analysis-room-scope.png` | Overall | Completed Wireshark traffic-analysis room with tasks for Nmap, ARP poisoning/MITM, DNS/ICMP tunneling, FTP, HTTP, HTTPS decryption, and credential hunting. |
| `01-recon-nmap-service-enumeration.png` | Reconnaissance | Nmap service/port reconnaissance against `10.64.159.173`. |
| `02-recon-wireshark-syn-scan.png` | Reconnaissance | Wireshark view of many SYN probes from `10.10.60.7` to `10.10.47.123` across numerous destination ports. |
| `03-recon-fragmented-scan-traffic.png` | Reconnaissance | Fragmented IPv4 packets mixed with TCP and ICMP during scan analysis. |
| `04-arp-scan-broadcast-pattern.png` | Recon / ARP | One host broadcasting ARP requests for many `192.168.1.x` addresses, consistent with local ARP scanning. |
| `05-arp-scapy-host-discovery.png` | ARP Analysis | Scapy ARP result and Windows interface information used to correlate IP and MAC addresses. |
| `06-arp-wireshark-gateway-traffic.png` | ARP Analysis | Wireshark capture containing ARP request/reply activity alongside DNS and ICMP traffic. |
| `07-dns-a-query-filter-analysis.png` | DNS | Type A DNS requests isolated with a valid Wireshark query filter. |
| `08-http-beacon-pattern-analysis.png` | C2 Hunting / HTTP | Cleartext HTTP request analysis including a `/scripts/beacon.dll` URI; used as a reminder to validate patterns rather than infer intent from a name. |
| `09-ftp-login-failure-analysis.png` | Cleartext FTP | Repeated FTP `530 Login incorrect` responses between two hosts. |
| `10-tls-clienthello-sni-analysis.png` | TLS | TLS 1.3 Client Hello traffic and Server Name (`clientservices.googleapis.com`) visible in handshake metadata. |
| `11-tls-encrypted-application-data.png` | TLS | Full TCP/TLS stream showing Client Hello, handshake traffic, and encrypted Application Data. |
| `12-tls-keylog-decryption-setup.png` | TLS | Wireshark TLS preferences configured with a lab key-log file for authorized session decryption. |
| `13-decrypted-http2-traffic.png` | TLS | HTTP/2 HEADERS, DATA, SETTINGS, and PING frames visible after TLS decryption. |
| `14-wireshark-conversations-baseline.png` | C2 Hunting | Wireshark Conversations statistics used to compare endpoints by packet and byte volume. |

## Evidence Handling Note

Not every case had a dedicated standalone screenshot recoverable from the conversation library. In particular, I did not find the original dedicated DNS-tunnel or ICMP-tunnel packet screenshots. I kept the written analysis but did not fabricate replacement evidence.
