# Case 01 — Phishing URL

## The Problem

A phishing alert involved a domain impersonating a Microsoft service. Firewall evidence showed that the user's system attempted to access the suspicious URL.

## What I Investigated

I treated the URL itself as only the starting point and looked for evidence that the activity reached the endpoint. The key questions were whether the domain was deceptive, whether a user system actually attempted the connection, and whether the activity justified follow-up beyond simply blocking the URL.

## What I Found

The domain used Microsoft-themed impersonation and the firewall evidence showed an access attempt from the user's system. That combination is stronger than a reputation-only alert because it connects a deceptive destination with actual user-side activity.

## Classification

**True Positive**

The evidence supported a real phishing interaction rather than an isolated detection with no endpoint involvement.

## Escalation

**Yes**

The case was escalated for user and endpoint follow-up because a system attempted to reach the phishing destination.

## Recommended Actions

- Confirm whether the user clicked the link or entered credentials.
- Review browser, proxy, DNS, and endpoint telemetry around the same time.
- Reset credentials if credential submission cannot be ruled out.
- Block the phishing domain and related indicators.
- Search for the same URL/domain across other users and hosts.

## What I Learned

A blocked or detected phishing URL should not be judged by the domain name alone. The important step is correlating it with user and network evidence to determine whether there was real interaction and whether escalation is necessary.

## Evidence

The original case screenshot was referenced in earlier triage work but was not available as a retrievable file when this archive was rebuilt. No substitute screenshot was fabricated.
