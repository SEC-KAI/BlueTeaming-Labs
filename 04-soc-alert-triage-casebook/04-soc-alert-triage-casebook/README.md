# SOC Alert Triage Casebook

This directory documents six SOC alert-triage investigations completed in a lab/training environment. The focus is not just on the alert that fired, but on the evidence used to decide whether the activity was malicious, legitimate, contained, or required escalation.

## What this casebook demonstrates

- Alert validation using endpoint, firewall, DNS, email, process, and user-context evidence
- Correlation of multiple events instead of judging a single alert in isolation
- True-positive versus false-positive classification
- Escalation decisions based on observed impact and follow-on activity
- Clear documentation of evidence, reasoning, and recommended next steps

## Cases

| Case | Investigation | Outcome |
|---|---|---|
| 01 | Phishing URL / Microsoft-impersonation domain | True positive; escalated for follow-up |
| 02 | Legitimate internal onboarding email | False positive; no escalation |
| 03 | Atomic Red Team activity by IT staff | False positive; authorized/security-testing context |
| 04 | Administrative backup activity using `wbadmin.exe` | False positive; legitimate administrative behavior |
| 05 | Macro-enabled Word document alerts | False positive after process/network context review |
| 06 | DNS + firewall phishing-site block | Blocked suspicious request; no compromise observed and no escalation required from the available evidence |

## Investigation approach

For each alert, the investigation asks:

1. What exactly triggered the alert?
2. Which user, host, process, URL, domain, or IP was involved?
3. Is the behavior expected for that user or system?
4. Did the alert correlate with endpoint, firewall, DNS, email, or network evidence?
5. Was there successful execution, access, persistence, credential theft, or outbound communication?
6. Is the event a true positive, false positive, or a correctly blocked security event?
7. Does the evidence justify escalation?

## Evidence

The `evidence/` folders contain screenshots recovered from the original lab investigations and renamed so their purpose is clear.

Case 01 is intentionally documentation-only in this archive. The earlier conversation preserved the investigation result, but the original case screenshot was not available as a retrievable file when this archive was rebuilt. No unrelated or fabricated screenshot was substituted.

## Scope note

These investigations were performed in lab/training environments and are presented as portfolio evidence of SOC analysis methodology, not as production incident-response experience.
