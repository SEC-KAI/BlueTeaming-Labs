# Wireshark Investigation Filters

These are starting points for investigation, not automatic verdicts.

## Reconnaissance

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
arp
arp.opcode == 1
ip.addr == X.X.X.X
tcp.stream eq N
```

Look for one source touching many ports/hosts, incomplete handshakes, rapid timing, or broad ARP requests.

## ARP Analysis

```text
arp
arp.opcode == 1
arp.opcode == 2
arp.dst.hw_mac == 00:00:00:00:00:00
```

Compare the same IP address over time and check whether its MAC mapping unexpectedly changes.

## DNS / DNS Tunneling

```text
dns
dns && dns.flags.response == 0 && dns.qry.type == 1
dns.qry.name.len > 15 && !mdns
```

Long names alone are not enough; correlate changing labels, repeated parent domains, volume, and timing.

## ICMP / ICMP Tunneling

```text
icmp
icmp && data.len > 64
```

Investigate repeated large payloads, structured data, unusual timing, and long-lived bidirectional activity.

## FTP

```text
ftp
tcp.port == 21
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp.response.code == 530
```

Follow the TCP stream to reconstruct the FTP control conversation.

## HTTP

```text
http
http.request
http.request.method == "GET"
http.request.method == "POST"
http.request.uri contains "login"
```

Use HTTP filters for cleartext traffic and for decrypted web sessions when Wireshark can dissect them.

## TLS / HTTPS

```text
tls
tls.handshake.type == 1
tls.handshake.type == 2
tls.handshake.extensions_server_name
http2
```

Use endpoint, timing, handshake, SNI where visible, packet sizes, and decryption in authorized lab environments.

## C2 / Large Transfer Hunting

```text
ip.src == X.X.X.X
ip.dst == X.X.X.X
ip.addr == X.X.X.X
tcp.stream eq N
tcp.len > 1000
```

For beaconing, Wireshark filtering is only one step. Export or review timestamps and compare the interval between repeated connections.
