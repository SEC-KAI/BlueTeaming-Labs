# Phishing & Email Investigation

This directory documents hands-on email investigations using raw headers, authentication results, SIEM/firewall telemetry, phishing-page analysis, and packet captures. The goal is to show how I separate a suspicious email from a confirmed phishing event and from a legitimate message that merely triggered a detection rule.

## What I investigated

I organized the evidence into three case types:

1. **Spoofed sender** — reconstructed the mail path and compared the claimed sender with SPF and DMARC results.
2. **Suspicious link** — investigated phishing URLs, a Microsoft-lookalike domain, a phishing kit, and related firewall activity.
3. **Legitimate email comparison** — reviewed an HR onboarding alert that initially looked suspicious but was closed as a false positive after contextual validation and activity checks.

## Investigation approach

My workflow was:

- Review the visible sender, Return-Path, Reply-To, subject, and message content.
- Read `Received` headers from the oldest visible hop upward to reconstruct delivery.
- Check SPF, DKIM, and DMARC evidence without assuming that one field alone proves maliciousness.
- Inspect the real destination of links instead of trusting display text.
- Correlate email events with firewall or network evidence when user interaction may have occurred.
- Compare suspicious indicators against known-good business context before deciding true positive or false positive.
- Map only the behaviors supported by the evidence to MITRE ATT&CK.

## Key findings

### Spoofed-sender investigation

The raw message claimed `info@mutawamarine.com` as the sender. The earliest useful `Received` hop showed a connection from `192.119.71.157` with the host name `hwsrv-737338.hostwindsdns.com`, while the SMTP identity claimed `mutawamarine.com`.

The message authentication evidence showed:

- `Received-SPF: fail`
- `Authentication-Results: spf=fail`
- `dmarc=unknown`

A separate SPF lookup for `mutawamarine.com` showed:

```text
v=spf1 include:spf.protection.outlook.com -all
```

That policy authorizes the listed Microsoft 365 SPF infrastructure and ends in a hard fail for unauthorized senders. The observed source therefore did not support the claimed sending identity. No DKIM result was visible in the captured excerpt, so I did not use DKIM as a deciding factor.

Evidence:

- [Raw message headers](screenshots/01-spoofed-sender-raw-headers.png)
- [Earliest useful Received hop](screenshots/02-spoofed-sender-originating-hop.png)
- [SPF/DMARC authentication result](screenshots/03-spf-dmarc-authentication-results.png)
- [Published SPF policy](screenshots/04-mutawamarine-spf-policy.png)
- [Originating-IP investigation](screenshots/05-spoofed-sender-origin-ip-analysis.png)

### Suspicious-link investigation

One phishing exercise identified `Accounts.Payable@groupmarketingonline.icu` as the adversary-controlled sender. The investigation followed a redirect to `kennaroads.buzz`, where the page impersonated Microsoft. The associated `Update365.zip` phishing kit was identified as a Trojan by the analysis workflow and contained 49 files.

A second SOC example used the lookalike sender `no-reply@m1crosoftsupport.co` and the URL:

```text
https://m1crosoftsupport.co/login
```

Firewall telemetry then showed an allowed HTTPS connection from `10.20.2.25` to `45.148.10.131:443` for that URL. This was stronger evidence than email content alone because it showed the endpoint reached the suspicious destination.

A separate true-positive case report also documented a user clicking a blacklisted delivery-themed URL, which was blocked by the firewall and escalated for further review.

Evidence:

- [Redirect, sender, and impersonation findings](screenshots/07-phishing-email-redirection-and-impersonation.png)
- [Observed phishing URL](screenshots/08-phishing-url-kennaroads-buzz.png)
- [Phishing-kit analysis](screenshots/09-phishing-kit-update365-analysis.png)
- [Microsoft-lookalike phishing email](screenshots/10-microsoft-lookalike-phishing-email.png)
- [Firewall correlation](screenshots/11-microsoft-lookalike-firewall-connection.png)
- [True-positive malicious-link report](screenshots/12-malicious-link-true-positive-report.png)

### Legitimate-email comparison

An inbound email from `onboarding@hrconnex.thm` triggered the same general suspicious-external-link type of alert. The message contained an onboarding URL and no attachment. SIEM review showed the inbound HR message, but the case was ultimately closed as a false positive after the sender was validated as an approved HR source and no suspicious link activity was identified.

This comparison is important because an external link by itself is not enough to classify an email as malicious. Context, sender validation, user interaction, and downstream telemetry changed the disposition.

Evidence:

- [HR onboarding alert](screenshots/13-legitimate-hr-onboarding-alert.png)
- [HR email SIEM event](screenshots/14-legitimate-hr-email-siem-event.png)
- [False-positive closure](screenshots/15-legitimate-email-false-positive-closure.png)

## Network evidence note

The SMTP Wireshark screenshot in this directory is a **separate network-analysis example** from the sender-header case. It shows SMTP DATA traffic and a `552-5.7.0` server response blocking the message, followed by TCP connection teardown. I kept it because it demonstrates how packet data can corroborate email-delivery behavior, but I do not treat it as the same email as the `mutawamarine.com` header sample.

- [SMTP packet stream](screenshots/06-smtp-message-blocked-packet-stream.png)

## Skills demonstrated

- Email header analysis and mail-path reconstruction
- SPF and DMARC interpretation
- Suspicious-domain and URL analysis
- Phishing-kit triage
- SIEM and firewall correlation
- Wireshark SMTP analysis
- True-positive vs false-positive classification
- Evidence-based MITRE ATT&CK mapping

## Directory map

```text
06-phishing-email-investigation/
├── README.md
├── cases/
│   ├── spoofed-sender/
│   ├── suspicious-link/
│   └── legitimate-email-comparison/
├── evidence/
│   ├── headers/
│   ├── authentication-results/
│   └── network/
├── analysis/
│   └── email-header-analysis.md
├── attack-mapping/
│   └── mitre-attack.md
└── screenshots/
```
