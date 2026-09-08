# ARP Analysis

## Problem

Analyze how ARP discovery and ARP manipulation appear on a local network and understand how a defender can recognize abnormal IP-to-MAC behavior.

## What I Investigated

In an authorized lab, I used Scapy on Kali Linux to create ARP traffic while monitoring Windows and Wireshark. The exercises covered ARP discovery, ARP cache behavior, local ARP scanning, and spoofing/poisoning concepts.

## Evidence

- `../../screenshots/04-arp-scan-broadcast-pattern.png` shows one MAC address broadcasting ARP requests for many `192.168.1.x` addresses.
- `../../screenshots/05-arp-scapy-host-discovery.png` shows Scapy ARP activity beside Windows network configuration, including IP and MAC information used during the lab.
- `../../screenshots/06-arp-wireshark-gateway-traffic.png` shows ARP requests/replies alongside DNS and ICMP traffic in Wireshark.

## Findings

ARP normally answers a simple local question: which MAC address owns this IPv4 address? That makes changes in an IP-to-MAC mapping especially important during a poisoning investigation.

The local scan evidence shows repeated broadcast requests from the same source to many addresses. During spoofing exercises, the key verification step was to compare the target's ARP cache before and after crafted ARP traffic and confirm whether the claimed IP mapped to the attacker's MAC.

A spoofed mapping may also be temporary. Legitimate ARP traffic can refresh the correct mapping, so persistence requires repeated poisoning in many practical scenarios.

## Defensive Recommendations

- Use Dynamic ARP Inspection where supported.
- Pair DAI with trusted DHCP snooping bindings.
- Monitor for one MAC unexpectedly claiming multiple important IP addresses.
- Alert on conflicting gateway IP-to-MAC mappings.
- Investigate bursts of unsolicited ARP replies or abnormal ARP volume.

## What I Learned

ARP poisoning is best verified by comparing mappings over time. Seeing ARP traffic is not enough; the important question is whether a trusted IP unexpectedly changes to a different MAC address.
