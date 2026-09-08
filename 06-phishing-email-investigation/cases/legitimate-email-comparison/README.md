# Case: Legitimate Email Comparison

## Problem

An HR onboarding message triggered an `Inbound Email Containing Suspicious External Link` alert. The goal was to determine whether the alert represented phishing or a legitimate business workflow.

## Evidence reviewed

The alert showed:

- Sender: `onboarding@hrconnex.thm`
- Recipient: `j.garcia@thetrydaily.thm`
- Subject: `Action Required: Finalize Your Onboarding Profile`
- Attachment: `None`
- Link: an `hrconnex.thm/onboarding/...` URL
- Direction: inbound

I searched the email events for the HR domain and reviewed the message context. The final case report states that `hrconnex.thm` was a trusted source associated with the company's HR system and that no suspicious link activity was identified.

## Contrast with malicious examples

| Indicator | Legitimate HR email | Malicious-link examples |
|---|---|---|
| Domain context | Approved HR service | Lookalike or unrelated domains |
| Message purpose | Expected onboarding workflow | Credential/login or delivery lure |
| Attachment | None | Phishing-kit/archive activity in separate sample |
| User/network activity | No suspicious link activity identified | Outbound connection or blocked malicious URL observed |
| Final disposition | False positive | True positive |

## Disposition

**False positive.** The alert was useful because it prompted validation, but the evidence supported legitimate business activity rather than phishing.

## Evidence

- [Alert details](../../screenshots/13-legitimate-hr-onboarding-alert.png)
- [SIEM email event](../../screenshots/14-legitimate-hr-email-siem-event.png)
- [False-positive closure](../../screenshots/15-legitimate-email-false-positive-closure.png)
