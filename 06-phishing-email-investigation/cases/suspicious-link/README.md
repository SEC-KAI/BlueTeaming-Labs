# Case: Suspicious Link

## Problem

Several phishing exercises involved external links that attempted to imitate trusted services. I focused on the actual destination, the surrounding email context, and any downstream network activity rather than judging the email only by its wording.

## Example 1 — Redirected Microsoft impersonation

The investigation identified:

- Adversary sender: `Accounts.Payable@groupmarketingonline.icu`
- Redirect/root domain: `kennaroads.buzz`
- Impersonated brand: Microsoft
- Phishing-kit archive: `Update365.zip`
- SHA-256: `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686`
- Threat category shown in the lab analysis: Trojan
- Files in the archive: 49

The browser evidence also showed the suspicious site being served over HTTP from an Apache server on `kennaroads.buzz`.

## Example 2 — Microsoft lookalike domain with network correlation

A separate SOC event contained:

- Sender: `no-reply@m1crosoftsupport.co`
- Subject: `Unusual Sign-In Activity on Your Microsoft Account`
- URL: `https://m1crosoftsupport.co/login`
- Recipient: `c.allen@thetrydaily.thm`

The domain uses the numeral `1` in `m1crosoft`, which imitates the Microsoft name. The email also used an account-security warning to push the recipient toward a login page.

Firewall telemetry then showed:

- Action: `allowed`
- Source IP: `10.20.2.25`
- Destination IP: `45.148.10.131`
- Destination port: `443/TCP`
- URL: `https://m1crosoftsupport.co/login`

That network event demonstrates why I correlate email data with firewall logs: it shows whether the suspicious URL was actually reached.

## Additional true-positive example

Another SOC report documented a delivery-themed email from `urgents@amazon.biz`. The user interacted with the URL, but the firewall blocked it because the destination was already blacklisted. The alert was classified true positive and escalated for additional review.

This example is kept separate from the `m1crosoftsupport.co` indicators; the two events are not treated as one incident.

## Disposition

The malicious-link examples were treated as **true-positive phishing activity** because the domains, impersonation behavior, phishing infrastructure, and/or user interaction were corroborated by additional evidence.

## Evidence

- [Sender, redirect, and impersonation findings](../../screenshots/07-phishing-email-redirection-and-impersonation.png)
- [Phishing URL](../../screenshots/08-phishing-url-kennaroads-buzz.png)
- [Phishing-kit analysis](../../screenshots/09-phishing-kit-update365-analysis.png)
- [Lookalike-domain email event](../../screenshots/10-microsoft-lookalike-phishing-email.png)
- [Firewall connection](../../screenshots/11-microsoft-lookalike-firewall-connection.png)
- [True-positive SOC report](../../screenshots/12-malicious-link-true-positive-report.png)
