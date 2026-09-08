# Case: Spoofed Sender

## Problem

A suspicious message claimed to originate from `info@mutawamarine.com`. I needed to determine whether the transport path and email-authentication results supported that claimed identity.

## Investigation

I reviewed the raw source instead of trusting the display name. The useful fields were:

- `Return-Path: <info@mutawamarine.com>`
- `From: "Mr. James Jackson" <info@mutawamarine.com>`
- Earliest useful external `Received` hop: `192.119.71.157`
- Hostname shown in the hop: `hwsrv-737338.hostwindsdns.com`
- SMTP HELO identity: `mutawamarine.com`
- `Received-SPF: fail`
- `Authentication-Results: spf=fail`
- `dmarc=unknown`

The exercise also emphasized that an `X-Originating-IP` field should not override the actual `Received` chain. The earliest useful `Received` header was used to identify the source of the visible mail path.

I then checked the sender domain's SPF policy:

```text
v=spf1 include:spf.protection.outlook.com -all
```

The observed sending source did not match the infrastructure authorized by that policy, which agrees with the recorded SPF failure.

## Findings

The message's claimed domain was not authenticated by SPF at the receiving system. The mail path also showed infrastructure inconsistent with the sender domain's published SPF policy. Together, these fields support treating the claimed sender identity as untrusted.

I did **not** record a DKIM failure because the selected header evidence does not show a DKIM result. I also do not treat `dmarc=unknown` as the same thing as a DMARC failure.

## Disposition

**Suspicious / phishing-related sender identity.** The authentication and routing evidence did not support the claimed sender.

## Evidence

- [Raw headers](../../screenshots/01-spoofed-sender-raw-headers.png)
- [Originating Received hop](../../screenshots/02-spoofed-sender-originating-hop.png)
- [Authentication results](../../screenshots/03-spf-dmarc-authentication-results.png)
- [SPF policy lookup](../../screenshots/04-mutawamarine-spf-policy.png)
- [Origin-IP analysis workflow](../../screenshots/05-spoofed-sender-origin-ip-analysis.png)
