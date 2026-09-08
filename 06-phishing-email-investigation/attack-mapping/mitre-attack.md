# MITRE ATT&CK Mapping

The mappings below are limited to behaviors supported by the investigation evidence. They describe the observed phishing behavior; they do not imply that every technique occurred in every individual sample.

| Technique | ID | Evidence in this directory | Why it applies |
|---|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | `kennaroads.buzz`, `m1crosoftsupport.co/login`, delivery-themed malicious URL | The emails attempted to move recipients to attacker-controlled or suspicious web destinations. |
| User Execution: Malicious Link | T1204.001 | Firewall-confirmed access to the lookalike login URL; separate case report documenting a clicked blacklisted URL | User interaction with a malicious link was observed in the SOC examples. |
| Masquerading | T1036 | Microsoft-lookalike domain and Microsoft impersonation | The phishing infrastructure imitated a trusted brand/service to make the lure appear legitimate. |
| Input Capture: Web Portal Capture | T1056.003 | Microsoft-style phishing page and phishing-kit evidence associated with credential collection | The phishing infrastructure was designed to collect credentials through a web login experience. |

## Scope notes

- SPF failure and DMARC status are detection/authentication evidence, not ATT&CK techniques by themselves.
- The `mutawamarine.com` message supports a sender-identity deception finding, but the selected evidence does not prove a specific payload or post-compromise technique.
- The legitimate HR onboarding message is intentionally **not** mapped to an adversary technique because it was closed as a false positive.
- The SMTP Wireshark capture is a separate mail-transport example and is not used to extend the ATT&CK mapping of the other incidents.
