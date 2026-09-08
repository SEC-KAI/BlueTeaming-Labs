# Reconnaissance Analysis

## Problem

Determine how network reconnaissance appears from a defender's point of view and distinguish scanning behavior from ordinary connection attempts.

## What I Investigated

I compared Nmap output with packet-level evidence in Wireshark. The lab included service enumeration, SYN-based scanning, other scan styles, and fragmented traffic intended to change how probes appear on the wire.

## Evidence

- `../../screenshots/01-recon-nmap-service-enumeration.png` shows Nmap reconnaissance against `10.64.159.173` and the services exposed by the target.
- `../../screenshots/02-recon-wireshark-syn-scan.png` shows a high concentration of TCP SYN probes from `10.10.60.7` toward `10.10.47.123` across many destination ports.
- `../../screenshots/03-recon-fragmented-scan-traffic.png` shows fragmented IPv4 traffic mixed with TCP/ICMP traffic during scan analysis.
- `../../screenshots/04-arp-scan-broadcast-pattern.png` shows rapid ARP requests from one host for many local IPv4 addresses.

## Findings

A scan is identified by the pattern, not by one SYN packet. In the TCP capture, one source probes many destination ports in a short period. That concentration is much more meaningful than seeing a normal SYN by itself.

The ARP capture demonstrates the Layer 2 equivalent: one source repeatedly asks who owns many local addresses. A single ARP request is expected behavior; a burst across a large address range is consistent with local host discovery.

Fragmentation also matters during analysis because a probe may be split across multiple IP fragments. Simple detections that assume every relevant header or payload appears in one packet can miss or misinterpret this traffic.

## Defensive Recommendations

- Alert on unusually high unique destination-port counts from one source.
- Correlate SYN attempts with completed handshakes and resets.
- Monitor bursts of ARP requests across many local IP addresses.
- Reassemble fragmented traffic before applying content-dependent detections.
- Baseline known vulnerability scanners so authorized scanning can be separated from unexpected reconnaissance.

## What I Learned

The strongest indicator of reconnaissance is repeated behavior across ports, hosts, or addresses. Packet-level context makes the difference between an ordinary connection and a scan pattern.
