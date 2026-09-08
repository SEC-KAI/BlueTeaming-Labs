# SOC Alert Triage Template

## Alert Summary

- **Alert name:**
- **Severity:**
- **Category:**
- **Time of activity:**
- **User:**
- **Host:**
- **Source IP:**
- **Destination IP / domain / URL:**
- **Process / command line:**

## Initial Question

What behavior caused the alert, and what would make the activity malicious versus legitimate?

## Evidence Reviewed

- Alert details
- Endpoint / Sysmon events
- Process and parent-process context
- User and asset role
- Authentication activity
- Firewall / proxy activity
- DNS activity
- Email activity
- File hash / reputation evidence
- Related alerts in the same session or time window

## Findings

Document only what the evidence confirms. Separate confirmed facts from assumptions.

## Classification

- **Decision:** True Positive / False Positive / Benign-But-True / Inconclusive
- **Rationale:**

## Escalation

- **Escalate:** Yes / No
- **Reason:**

## Indicators / Entities

- Users:
- Hosts:
- IP addresses:
- Domains / URLs:
- Files / hashes:
- Processes / commands:

## Recommended Actions

- Containment:
- Validation:
- Remediation:
- Detection improvement:

## Conclusion

State the final analyst decision in 2-4 sentences and explain which evidence was decisive.
