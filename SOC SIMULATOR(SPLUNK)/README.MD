
# TryHackMe SOC Alert Triage Investigation

## Overview

This lab placed me in a simulated **Security Operations Center (SOC)** environment where I was responsible for reviewing security alerts, investigating suspicious activity, and determining whether each alert was a **true positive or false positive**.

The main goal was to practice the alert triage process that a SOC analyst would normally follow:

**Alert → Investigate → Gather Evidence → Classify → Escalate/Close → Document**

---

## Tools Used

* SIEM / Splunk
* Alert Queue
* Firewall Logs
* Email Logs
* Analyst VM
* Case Reporting System
* SOC Playbooks

---

## Investigation Process

### 1. Alert Triage

I started by taking ownership of alerts from the SOC alert queue.

The alerts included activity such as:

* Suspicious external links in inbound emails
* Phishing attempts
* Connections to blacklisted URLs
* Potential user interaction with malicious links

Each alert needed to be investigated before deciding whether it was actually malicious.

---

### 2. Email Investigation

For phishing-related alerts, I reviewed information such as:

* Sender address
* Recipient
* Email subject
* Email content
* URLs contained in the message

For example, one email impersonated Microsoft and attempted to convince the user to visit a suspicious login page.

---

### 3. SIEM Investigation

I used **Splunk** to search logs and find additional evidence related to the alerts.

This allowed me to correlate information between different data sources.

For example:

```text
Email Alert
     ↓
Identify suspicious URL
     ↓
Search firewall/network logs
     ↓
Check if the user accessed the URL
```

This was important because receiving a phishing email does not automatically mean the user interacted with it.

---

## True Positive Investigation

One phishing alert contained a suspicious Microsoft impersonation domain.

After checking the firewall logs, I found that the user's system attempted to connect to the phishing URL over HTTPS.

Because there was evidence that the user interacted with the malicious link, I classified the alert as a:

**True Positive**

The incident was then escalated because additional investigation would be required to determine whether credentials were submitted or further malicious activity occurred.

---

## False Positive Investigation

Another alert was triggered because an email contained multiple external links.

After investigating the sender, I confirmed that the domain belonged to the organization's legitimate HR system.

I also checked for evidence showing whether the user interacted with suspicious URLs and found no malicious activity.

The alert was therefore classified as a:

**False Positive**

This showed me why context is important when investigating alerts. A detection rule being triggered does not always mean an attack occurred.

---

## Case Reporting

After completing each investigation, I documented:

* Time of activity
* Affected users and systems
* Evidence discovered
* Investigation steps
* Incident classification
* Reason for classification
* Escalation decision

This helped me practice not only identifying threats but also clearly documenting why a decision was made.

---

## Results

At the end of the scenario, I achieved:

* **100% True Positive identification rate**
* **100% False Positive identification rate**
* **5 alerts closed**
* **2-minute Mean Time to Resolve (MTTR)**
* Successfully prevented the simulated security breach

---

## What I Learned

The biggest thing I learned from this lab is that SOC alert triage is not simply about deciding whether an alert "looks suspicious."

A SOC analyst needs to gather evidence from multiple sources and correlate the activity before making a decision.

I also gained more hands-on experience with:

* Alert triage
* SIEM investigation
* Splunk searches
* Phishing analysis
* Log correlation
* True positive vs false positive classification
* Incident escalation
* Case documentation

This lab was part of my progress through the **TryHackMe SOC Analyst Level 1 path** and gave me a better understanding of how alerts move from initial detection to investigation and response.
