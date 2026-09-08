# Header Evidence

## Fields reviewed

### Claimed identity

```text
Return-Path: <info@mutawamarine.com>
From: "Mr. James Jackson" <info@mutawamarine.com>
```

### Earliest useful Received hop

The captured header chain shows an external handoff containing:

```text
hwsrv-737338.hostwindsdns.com
192.119.71.157
helo=mutawamarine.com
```

This is more useful for reconstructing the visible transport path than relying on the `X-Originating-IP` field alone.

## Interpretation

`Received` headers are read from the oldest useful hop upward. I use them to reconstruct server handoffs, then compare the observed connecting infrastructure to authentication results and the claimed sender identity.

The header evidence alone is not treated as proof of the attacker's physical machine. It shows the infrastructure visible in the mail path.

## Screenshots

- [Full raw header view](../../screenshots/01-spoofed-sender-raw-headers.png)
- [Received-hop close-up](../../screenshots/02-spoofed-sender-originating-hop.png)
