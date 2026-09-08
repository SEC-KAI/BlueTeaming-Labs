# TLS / HTTPS Analysis

## Problem

Investigate encrypted web traffic when application contents are not immediately readable in a packet capture.

## What I Investigated

I followed a TLS session from handshake metadata through encrypted Application Data and then configured a lab TLS key log so Wireshark could decrypt the session and expose HTTP/2 messages.

## Evidence

- `../../screenshots/10-tls-clienthello-sni-analysis.png` shows TLS 1.3 Client Hello packets and the Server Name field `clientservices.googleapis.com`.
- `../../screenshots/11-tls-encrypted-application-data.png` shows a TCP/TLS stream with Client Hello, handshake messages, and encrypted Application Data.
- `../../screenshots/12-tls-keylog-decryption-setup.png` shows Wireshark's TLS preferences configured with a key-log file.
- `../../screenshots/13-decrypted-http2-traffic.png` shows HTTP/2 HEADERS, DATA, SETTINGS, and PING frames after decryption.

## Findings

Before decryption, useful metadata can still include endpoints, ports, timing, packet sizes, handshake messages, and—depending on the session—server-name information. The actual HTTP content remains protected.

After loading the appropriate lab-generated TLS session keys, Wireshark was able to dissect the protected traffic as HTTP/2, exposing the application-layer conversation for analysis.

## Useful Filters

```text
tls
tls.handshake.type == 1
tls.handshake.type == 2
tls.handshake.extensions_server_name
http2
```

## Defensive Recommendations

- Correlate TLS destinations and timing with endpoint telemetry.
- Use approved enterprise decryption/inspection only where policy and architecture permit it.
- Preserve endpoint and proxy metadata because packet payloads may remain encrypted.
- Hunt on connection patterns rather than assuming encrypted traffic is benign.

## What I Learned

Encryption changes what is visible, not whether the connection can be investigated. Even without plaintext, flow behavior and handshake metadata still provide useful evidence.
