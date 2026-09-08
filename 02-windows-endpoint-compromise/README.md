# Windows Endpoint Compromise Investigation

## Overview

This project documents a SOC investigation of suspicious activity on the Windows endpoint **IT-TRYHATME**. The investigation was built from Windows Sysmon telemetry and related case evidence, with emphasis on reconstructing the process chain, validating external communication, and separating confirmed observations from assumptions.

The core sequence began with a hidden, Base64-encoded PowerShell process launched from a Windows service context. Sysmon then recorded that PowerShell process establishing an outbound TCP connection to **85.203.21.23:7532**. Follow-on activity included user discovery, creation of `Michael.Myres`, later RDP and command-shell activity, a registry value-set event, and explicit Volume Shadow Copy deletion.

> **Environment note:** This was an authorized security-training investigation. No production systems were involved.

---

## Executive Summary

The evidence supports a **true-positive Windows endpoint compromise**.

```text
services.exe / UpdateCheck context
            ↓
Hidden encoded PowerShell
            ↓
Outbound TCP connection
85.203.21.23:7532
            ↓
Remote-command-capable PowerShell logic
            ↓
User discovery (whoami)
            ↓
Michael.Myres account created
            ↓
RDP + cmd.exe activity
            ↓
Registry value set
            ↓
vssadmin delete shadows /all /quiet
```

The strongest evidence is the direct correlation between the PowerShell process and the network connection:

- Sysmon Event ID `1` recorded `powershell.exe -WindowStyle Hidden -EncodedCommand ...`.
- The PowerShell process ran as `TRYHATME\Andrew.Miller` on `IT-TRYHATME`.
- Its parent image was `C:\Windows\system32\services.exe` with the parent command line `C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]`.
- Five seconds later, Sysmon Event ID `3` recorded that same PowerShell PID initiating TCP traffic from `10.0.2.48` to `85.203.21.23:7532`.
- Decoding the PowerShell command showed logic that creates a TCP client, reads commands from the connection, executes them with `iex`, and returns command output.

This combination is not explained by ordinary administrative PowerShell use. The later account creation, RDP activity, command shell, registry event, and explicit shadow-copy deletion further support post-compromise behavior and warranted escalation and containment.

---

## Verified Incident Details

| Finding | Verified Value |
|---|---|
| Endpoint | `IT-TRYHATME` |
| Endpoint source IP | `10.0.2.48` |
| User context | `TRYHATME\Andrew.Miller` |
| Suspicious process | `powershell.exe` |
| PowerShell PID | `1405` |
| Parent image | `C:\Windows\system32\services.exe` |
| Parent PID | `6712` |
| Parent command line | `C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]` |
| PowerShell behavior | Hidden window + Base64-encoded command |
| External destination | `85.203.21.23` |
| Destination port | `7532/TCP` |
| Network initiated by endpoint | `true` |
| Later PowerShell child PID | `4700` |
| Observed discovery command | `whoami` |
| New account | `Michael.Myres` created with `net.exe user ... /add` |
| Later RDP activity | `85.203.21.23` → `10.0.2.48:3389` |
| Follow-on shell | `cmd.exe` under `TRYHATME\Michael.Myres` |
| Registry event | Sysmon Event ID `13` value set under the `Michael.Myres` user hive |
| Shadow-copy discovery | `vssadmin.exe List Shadows` |
| Shadow-copy deletion | `vssadmin.exe delete shadows /all /quiet` |
| Administrators membership | Recorded in case context; separate group-add command/event screenshot still recommended |

The screenshots recovered from the completed investigation are documented in [`screenshots/README.md`](screenshots/README.md). Raw/exported evidence can still be added under [`evidence/`](evidence/) if you later export the underlying events.

---

## Data Sources

### Sysmon
Used to correlate:

- process creation — Event ID `1`
- network connection — Event ID `3`
- registry activity — Event IDs `12`, `13`, and `14` when present
- file creation — Event ID `11` when relevant

### Windows Event Logs
Useful for validating:

- account creation
- group membership changes
- service activity
- authentication context

### SIEM / Splunk
Used to pivot across:

- user
- process ID
- parent process ID
- image / parent image
- destination IP and port
- event IDs
- timestamps

---

## Investigation Methodology

1. Start with the suspicious process alert and identify the exact command line.
2. Validate the parent process and process ancestry rather than judging the process name alone.
3. Decode the Base64 PowerShell content without executing it.
4. Pivot on PowerShell PID `1405` to locate related Sysmon events.
5. Correlate Event ID `1` process creation with Event ID `3` network activity.
6. Confirm whether the endpoint initiated the connection and record the destination.
7. Follow child processes and subsequent commands from the same user/session.
8. Review new-account creation, RDP activity, command shells, registry changes, and recovery-related activity.
9. Build a chronological timeline and distinguish confirmed evidence from interpretation.
10. Classify the alert only after the combined process + network + post-compromise behavior is correlated.

Detailed search examples are in [`queries/investigation-filters.md`](queries/investigation-filters.md).

---

# Investigation

## Phase 1 — Suspicious PowerShell Execution

Sysmon recorded a process creation event for:

```text
powershell.exe -WindowStyle Hidden -EncodedCommand ...
```

Important fields:

```text
Computer: IT-TRYHATME
User: TRYHATME\Andrew.Miller
ProcessID: 1405
ParentProcessID: 6712
ParentImage: C:\Windows\system32\services.exe
ParentCommandLine: C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]
```

### Analyst assessment

PowerShell is legitimate by itself. In this case, the combination of a hidden window, encoded command, service-based parent context, and immediately related external connection made the execution highly suspicious.

---

## Phase 2 — PowerShell Decoding

The Base64 content was decoded as UTF-16LE, which is the encoding commonly used with PowerShell `-EncodedCommand`.

The decoded logic:

- creates a `Net.Sockets.TCPClient`
- opens a network stream
- reads commands from the stream
- executes received commands with `iex`
- converts output to text
- writes command output back through the stream
- retries after a short sleep if an exception occurs

The captured decoded string contains a malformed address literal beginning with `=5.203.21.23`. This project does **not** silently correct that string. The separate Sysmon network event is the evidence used to identify the actual connection destination as `85.203.21.23:7532`.

### Analyst assessment

The decoded logic is remote-command-capable PowerShell behavior. Combined with the network event, it strongly supports command-and-control or remote shell activity.

---

## Phase 3 — External Network Connection

Sysmon Event ID `3` recorded:

```text
Computer: IT-TRYHATME
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
ProcessID: 1405
SourceIp: 10.0.2.48
SourcePort: 49795
DestinationIp: 85.203.21.23
DestinationPort: 7532
Protocol: tcp
Initiated: true
User: TRYHATME\Andrew.Miller
```

The network event occurred approximately five seconds after the suspicious PowerShell process creation event.

### Analyst assessment

This directly links the encoded PowerShell process to outbound communication with external infrastructure.

---

## Phase 4 — Follow-On PowerShell Activity

A later Sysmon Event ID `1` showed another PowerShell process:

```text
ProcessID: 4700
ParentProcessID: 1405
ParentImage: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

The observed command line again contained the encoded PowerShell logic.

### Analyst assessment

This establishes a PowerShell-to-PowerShell process relationship after the initial external connection. The evidence confirms the relationship but does not, by itself, explain why the second PowerShell process was created.

---

## Phase 5 — Discovery and Account Activity

Follow-on investigation identified:

- execution of `whoami`
- creation of the account `Michael.Myres`
- addition of that account to the `Administrators` group

### Analyst assessment

`whoami` is consistent with user/context discovery. Creating a new account and granting administrative membership is a significant privilege and persistence indicator when it is not authorized administrative activity.

---

## Phase 6 — RDP and Follow-On Shell Activity

Later Sysmon telemetry recorded RDP activity involving the same external IP, `85.203.21.23`, and destination `10.0.2.48:3389` on `IT-TRYHATME`. A subsequent process-creation event recorded `cmd.exe` running as `TRYHATME\Michael.Myres` with `explorer.exe` as its parent.

### Analyst assessment

This sequence supports follow-on interactive access and command-shell activity after creation of the new account. The screenshots establish the RDP network event and the shell user context, but they do not by themselves prove which authentication event created the session.

---

## Phase 7 — Registry Activity

Sysmon Event ID `13` recorded a registry value set by `explorer.exe` under `TRYHATME\Michael.Myres`. The target was under:

```text
HKU\...\SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Cached\...
```

### Analyst assessment

The event confirms a registry modification under the newly created account. The observed key is not treated as proof of a persistence mechanism without additional evidence connecting the change to persistence behavior.

---

## Phase 8 — Volume Shadow Copy Activity

Two later Sysmon process-creation events recorded:

```text
vssadmin.exe List Shadows
```

followed by:

```text
vssadmin.exe delete shadows /all /quiet
```

both under `TRYHATME\Michael.Myres`.

### Analyst assessment

The first command enumerated available Volume Shadow Copies. The second explicitly deleted all shadow copies without prompting, directly supporting an **Inhibit System Recovery** finding.

---

## Incident Classification

**True Positive — Escalation Required**

The classification is based on the combined evidence:

```text
Encoded hidden PowerShell
+ service-based parent process
+ outbound external TCP connection from the same PID
+ decoded remote-command logic
+ discovery activity
+ new-account creation and recorded admin-membership context
+ RDP + command-shell activity
+ registry modification
+ confirmed shadow-copy deletion
= endpoint compromise requiring containment and deeper scoping
```

---

## Response Recommendations

1. Isolate `IT-TRYHATME` from the network while preserving evidence.
2. Block or monitor `85.203.21.23:7532` according to organizational policy.
3. Disable or investigate the unauthorized `Michael.Myres` account and review Administrators membership.
4. Review the `UpdateCheck` service configuration and determine whether it was created, modified, or abused.
5. Collect the complete Sysmon and Windows Security event sequence around the compromise window.
6. Review the observed registry modification in context before assigning a persistence technique.
7. Treat the confirmed `vssadmin.exe delete shadows /all /quiet` event as recovery-impact evidence and assess whether restoration capability was affected.
8. Hunt for the same PowerShell command pattern, destination, service context, account, and related indicators across other endpoints.
9. Determine the initial access vector; it is not established by the current evidence.

---

## What This Project Demonstrates

- Windows endpoint triage
- Sysmon process and network correlation
- process-tree analysis
- Base64 PowerShell decoding
- PID / parent-PID pivoting
- command-and-control identification
- user and account activity review
- RDP and command-shell correlation
- Volume Shadow Copy / recovery-impact analysis
- conservative evidence-based incident reporting
- MITRE ATT&CK mapping without overstating unsupported behavior

---

## Repository Structure

```text
02-windows-endpoint-compromise/
├── README.md
├── analysis/
│   ├── incident-report.md
│   ├── asset-inventory.md
│   ├── confirmed-vs-unconfirmed.md
│   └── investigation-methodology.md
├── evidence/
│   ├── README.md
│   ├── sysmon/
│   ├── event-logs/
│   ├── process-tree/
│   └── network/
├── queries/
│   ├── investigation-filters.md
│   └── powershell-decoding.txt
├── timeline/
│   └── attack-timeline.md
├── attack-mapping/
│   └── mitre-attck.md
├── screenshots/
│   ├── README.md
│   ├── 01-encoded-powershell-alert.png
│   ├── 02-powershell-outbound-connection.png
│   ├── 03-new-admin-account-creation.png
│   ├── 04-suspicious-rdp-connection.png
│   ├── 05-attacker-shell-execution.png
│   ├── 06-registry-value-set.png
│   ├── 07-vssadmin-list-shadows.png
│   ├── 08-vssadmin-delete-shadows.png
│   └── 09-true-positive-classification.png
├── portfolio-card.md
└── .gitignore
```

---

## Disclaimer

This project documents an **authorized cybersecurity training scenario**. The screenshots are from the completed lab and are used only to support the investigation findings. The repository distinguishes direct evidence from analyst interpretation and avoids claiming an initial-access method, registry persistence, ransomware execution, or data theft without supporting evidence.
