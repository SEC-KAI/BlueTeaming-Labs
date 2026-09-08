# Portfolio Card — Windows Endpoint Compromise

**Project:** Windows Endpoint Compromise Investigation  
**Role:** SOC Analyst / Incident Investigator  
**Environment:** Authorized security-training lab  
**Primary telemetry:** Windows Sysmon, Windows event logs, SIEM/Splunk  

## Problem

Investigate suspicious PowerShell activity on a Windows endpoint and determine whether it represented legitimate administration or compromise.

## What I Investigated

- process ancestry and PID relationships
- hidden Base64-encoded PowerShell
- decoded command behavior
- outbound PowerShell network communication
- user discovery
- account creation and Administrators membership context
- RDP and command-shell activity
- registry activity
- `vssadmin.exe` shadow-copy enumeration and deletion

## What I Found

PowerShell PID `1405`, launched from a `services.exe` / `UpdateCheck` context, initiated an outbound TCP connection from `10.0.2.48` to `85.203.21.23:7532`. The decoded command implemented a TCP-based remote-command loop. Follow-on activity included `whoami`, creation of `Michael.Myres`, RDP activity involving the same external IP, `cmd.exe` under the new account, registry activity, and `vssadmin.exe delete shadows /all /quiet`.

## Decision

**True Positive — Escalate and contain the endpoint.**

## What I Learned

A suspicious process name is not enough by itself. The strongest conclusion came from correlating the exact PowerShell PID across process and network events, decoding the command safely, then validating what happened afterward without claiming behavior that the evidence did not prove.
