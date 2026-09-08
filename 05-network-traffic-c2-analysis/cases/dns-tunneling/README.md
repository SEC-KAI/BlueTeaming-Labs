# DNS Tunneling Analysis

## Problem

Determine when ordinary DNS queries begin to look like a channel for carrying data rather than normal hostname resolution.

## What I Investigated

I reviewed DNS requests in Wireshark and focused on query direction, record type, query-name length, repetition, and changes in subdomain labels.

## Evidence

- `../../screenshots/07-dns-a-query-filter-analysis.png` shows Type A DNS requests isolated with `dns.flags.response == 0 && dns.qry.type == 1`.
- `../../screenshots/00-wireshark-traffic-analysis-room-scope.png` documents completion of the lab section covering DNS and ICMP tunneling.

## Analysis Method

Normal DNS traffic can still contain long names, so a long query alone is not proof of tunneling. I look for a combination such as:

1. Many queries from the same client.
2. Repeated use of the same parent domain.
3. Long or encoded-looking subdomain labels that change every request.
4. Unusual query volume or regular timing.
5. Query/response behavior that does not match ordinary application use.

A useful starting filter is:

```text
dns.qry.name.len > 15 && !mdns
```

In the training material, examples such as changing labels beneath the same parent domain were used to demonstrate how data can be placed inside DNS names.

## Defensive Recommendations

- Baseline normal DNS volume per endpoint.
- Detect high-entropy or unusually long subdomain labels.
- Track repeated queries to rare parent domains.
- Correlate DNS anomalies with endpoint processes and external reputation.
- Use DNS logging at the resolver so patterns can be analyzed over time.

## What I Learned

DNS tunneling is a behavioral conclusion. The protocol remains DNS; the suspicious part is how the query-name field is being used and how often the behavior repeats.
