# ICMP Tunneling Analysis

## Problem

Identify when ICMP Echo traffic is being used as a carrier for data instead of simply testing reachability.

## What I Investigated

The lab demonstrated that the outer protocol visible in Wireshark can be ICMP while the payload transports another form of communication. This required separating the carrier protocol from the data placed inside it.

## Evidence

- `../../screenshots/00-wireshark-traffic-analysis-room-scope.png` confirms completion of the DNS/ICMP tunneling section of the traffic-analysis lab.
- `../../screenshots/03-recon-fragmented-scan-traffic.png` includes ICMP traffic alongside packet-level analysis and reinforces inspection of more than the protocol label alone.

The original dedicated ICMP-tunnel packet screenshot was not recoverable from my saved conversation files, so I did not substitute an unrelated image.

## Analysis Method

Indicators I would correlate include:

- Repeated Echo Request/Reply traffic for an extended period.
- Payloads much larger than expected for normal ping use.
- Similar or structured payload lengths.
- Unusual sequencing or timing.
- Payload bytes that decode or reassemble into another protocol or command stream.

Starting filter:

```text
icmp
```

To reduce normal small pings during hunting:

```text
icmp && data.len > 64
```

## Defensive Recommendations

- Baseline ICMP use in the environment.
- Alert on long-lived bidirectional ICMP sessions with large payloads.
- Inspect payload entropy and repetition when ICMP volume is unusual.
- Correlate suspicious ICMP with endpoint processes and other network indicators.

## What I Learned

Wireshark's protocol column identifies the carrier seen on the wire. It does not automatically tell me the purpose of the bytes inside that carrier.
