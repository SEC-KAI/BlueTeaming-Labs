# Network Traffic & C2 Analysis

## Overview

This project documents hands-on network traffic analysis performed with Wireshark, Nmap, Scapy, and network-security logs. The goal was to move beyond identifying protocols and instead determine what packet patterns can reveal about reconnaissance, ARP manipulation, tunneling, cleartext services, encrypted traffic, and possible command-and-control (C2) behavior.

The investigation set is organized as several focused cases rather than one artificial attack chain. Some cases are offensive traffic observed from the defensive side, while others are protocol-analysis exercises used to learn what normal and suspicious behavior look like on the wire.

## What I Investigated

- Nmap reconnaissance and how scanning appears in packet captures
- TCP SYN behavior, reset patterns, and fragmented scan traffic
- ARP discovery, ARP scanning, and ARP spoofing/poisoning concepts
- DNS query analysis and indicators associated with DNS tunneling
- ICMP tunneling and the difference between the carrier protocol and tunneled data
- Cleartext FTP behavior and authentication exposure
- TLS Client Hello, SNI, encrypted application data, and lab decryption with key logs
- HTTP/2 traffic after TLS decryption
- C2 beaconing based on repeated destination, timing, size, and frequency patterns

## Investigation Principle

A single unusual packet is rarely enough to prove compromise. I treated packet contents, connection frequency, source/destination relationships, timing, protocol behavior, and follow-on activity as a combined evidence set.

For example, a large outbound transfer can be suspicious, but size alone does not prove exfiltration. Likewise, a URI containing a word such as `beacon` does not prove C2. The surrounding traffic pattern has to support the conclusion.

## Cases

| Case | Focus |
|---|---|
| [Reconnaissance](cases/reconnaissance/README.md) | Nmap scans, SYN behavior, fragmentation, and scan recognition |
| [ARP Analysis](cases/arp-analysis/README.md) | ARP discovery, broadcast scanning, poisoning concepts, and verification |
| [DNS Tunneling](cases/dns-tunneling/README.md) | Long/changing DNS labels, repetition, volume, and tunneling indicators |
| [ICMP Tunneling](cases/icmp-tunneling/README.md) | Repeated ICMP traffic carrying data beyond normal ping behavior |
| [Cleartext FTP](cases/cleartext-ftp/README.md) | FTP authentication visibility and login analysis |
| [TLS Analysis](cases/tls-analysis/README.md) | Client Hello, SNI, encrypted traffic, key-log decryption, and HTTP/2 |
| [C2 Beaconing](cases/c2-beaconing/README.md) | Periodic outbound communication and C2-hunting methodology |

## Evidence

The `screenshots/` folder contains screenshots recovered from my earlier labs and investigation conversations. They include Nmap output, Wireshark scan traffic, ARP activity, DNS filtering, FTP analysis, TLS handshakes, TLS decryption, HTTP/2, and traffic-conversation statistics.

See [screenshots/README.md](screenshots/README.md) for the evidence index and captions.

## Packet Captures

The original raw PCAP/PCAPNG files were not recovered as reusable conversation files. I did not create fake captures to fill the folder. See [packet-captures/README.md](packet-captures/README.md).

## Supporting Material

- [Wireshark Filters](wireshark-filters.md)
- [MITRE ATT&CK Mapping](attack-mapping/mitre-attck.md)

## Key Takeaways

This project improved my ability to read traffic as a sequence of behavior instead of isolated packets. The most important lesson was to avoid jumping from one indicator to a conclusion: reconnaissance is identified by patterns across targets or ports, tunneling requires more than an unusual protocol, and C2 is stronger when periodicity and endpoint context line up with the traffic.
