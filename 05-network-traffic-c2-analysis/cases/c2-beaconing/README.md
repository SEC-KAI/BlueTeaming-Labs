# C2 Beaconing Analysis

## Problem

Identify network behavior that could represent an infected endpoint periodically checking in with attacker-controlled infrastructure.

## What I Investigated

My C2 exercises focused on repeated small outbound connections and the characteristics that make periodic communication suspicious: destination, interval, size consistency, frequency, and follow-on traffic.

I also worked through IDS logs containing repeated `Possible C2 Beaconing` alerts to a recurring destination service, which reinforced the importance of analyzing an event series rather than one alert.

## Evidence

- `../../screenshots/14-wireshark-conversations-baseline.png` shows Wireshark conversation statistics used to compare hosts by packet and byte volume.
- `../../screenshots/08-http-beacon-pattern-analysis.png` shows cleartext HTTP requests including a `/scripts/beacon.dll` URI in the capture.

The string `beacon.dll` is **not treated as proof of malicious C2**. The screenshot is useful because it demonstrates why URI names must be correlated with timing, repetition, endpoints, and other evidence before making a C2 determination.

## Detection Logic

A stronger C2-beaconing hypothesis usually combines several of these characteristics:

1. One internal host repeatedly contacts the same unusual external destination.
2. Connections occur at regular or near-regular intervals.
3. Requests have similar sizes or shapes.
4. The endpoint remains quiet between check-ins.
5. DNS, process, proxy, IDS, or reputation evidence supports the destination being suspicious.
6. Later command execution, downloads, or larger transfers are correlated to the same host/session.

## Important Distinction: Beaconing vs. Exfiltration

A later large outbound transfer may deserve investigation, but size alone does not prove exfiltration. I would need evidence showing what data was sent, which process sent it, where it went, and how it relates to the suspected compromise.

## Useful Hunting Approaches

Wireshark starting points:

```text
ip.addr == X.X.X.X
tcp.stream eq N
tcp.len > 1000
http.request
```

For log data, group by source, destination, destination port, time interval, and bytes transferred. Repeated periodic events become much easier to see when sorted chronologically.

## Defensive Recommendations

- Baseline normal recurring outbound services.
- Detect low-volume periodic connections to rare destinations.
- Correlate network findings with process and DNS telemetry.
- Block confirmed malicious infrastructure and isolate affected endpoints.
- Prefer behavior-based detections in addition to single-IP blocklists.

## What I Learned

Beaconing is a pattern. The strongest investigation starts with “who talks to whom, how often, how much, and what happens next?” rather than treating one connection as a verdict.
