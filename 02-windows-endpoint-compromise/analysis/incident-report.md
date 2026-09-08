# Incident Report — Windows Endpoint Compromise

## Incident Summary

A Windows endpoint investigation identified a hidden, Base64-encoded PowerShell process running on `IT-TRYHATME` under `TRYHATME\Andrew.Miller`. The process was launched from a `services.exe` context associated with `UpdateCheck` and then established an outbound TCP connection to `85.203.21.23:7532`.

Decoding the PowerShell content showed logic capable of reading commands from a TCP stream, executing those commands through `iex`, and returning output over the same connection. Follow-on evidence included `whoami`, creation of the account `Michael.Myres`, RDP activity involving the same external IP, `cmd.exe` execution under the new account, a Sysmon registry value-set event, `vssadmin.exe List Shadows`, and `vssadmin.exe delete shadows /all /quiet`. The case context also recorded the account being added to `Administrators`, although a separate group-membership event screenshot is still recommended.

The combined evidence supports a **true-positive endpoint compromise** and warranted escalation and containment.

---

## Incident Classification

| Field | Assessment |
|---|---|
| Classification | True Positive |
| Severity | Not assigned by the preserved source evidence |
| Affected endpoint | `IT-TRYHATME` |
| Endpoint IP | `10.0.2.48` |
| User context | `TRYHATME\Andrew.Miller` |
| External destination | `85.203.21.23:7532/TCP` |
| Primary suspicious process | `powershell.exe` PID `1405` |
| Initial access vector | Not established by current evidence |
| Escalation | Required |

---

## Key Evidence

### 1. Suspicious Process Creation

Sysmon Event ID `1` showed:

```text
CommandLine: powershell.exe -WindowStyle Hidden -EncodedCommand ...
Computer: IT-TRYHATME
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
ParentCommandLine: C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]
ParentImage: C:\Windows\system32\services.exe
ParentProcessID: 6712
ProcessID: 1405
User: TRYHATME\Andrew.Miller
```

The combination of hidden execution and an encoded command justified deeper analysis.

### 2. Decoded Remote-Command Logic

The decoded PowerShell code created a TCP client, read commands from a network stream, executed them with `iex`, and wrote output back to the stream.

The captured encoded content contains a malformed address literal (`=5.203.21.23`). The network telemetry below, rather than an assumption about that malformed string, is used to establish the actual destination.

### 3. Network Connection from the Same PID

Sysmon Event ID `3` showed:

```text
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

The shared PID directly correlates the suspicious PowerShell process with the external connection.

### 4. Follow-On Process Activity

A later Sysmon process creation event showed PowerShell PID `4700` with PowerShell PID `1405` as its parent. The observed command line again contained the encoded PowerShell logic.

This confirms follow-on PowerShell execution but does not independently establish the reason for the child process.

### 5. Post-Compromise Evidence

Recovered screenshots now directly show:

- creation of `Michael.Myres` with `net.exe user ... /add`
- later RDP activity involving `85.203.21.23` and `10.0.2.48:3389`
- `cmd.exe` execution under `TRYHATME\Michael.Myres`
- Sysmon Event ID `13` registry value-set activity under `Michael.Myres`
- `vssadmin.exe List Shadows`
- `vssadmin.exe delete shadows /all /quiet`

The prior case notes also recorded `whoami` and addition of `Michael.Myres` to `Administrators`. Those two details remain stronger with their own direct screenshots or exported events.

---

## Analyst Conclusion

The alert should be escalated because the evidence is not limited to a suspicious command line. Process telemetry and network telemetry independently correlate to the same PowerShell PID, and the decoded script implements a remote command loop. The later discovery and administrative-account activity further support post-compromise actions.

The current evidence does **not** establish the initial access method, a registry-based persistence mechanism, data theft, ransomware execution, or the full compromise scope. The `vssadmin.exe delete shadows /all /quiet` event does, however, directly establish deletion of Volume Shadow Copies.

---

## Recommended Actions

- isolate `IT-TRYHATME`
- preserve Sysmon and Security logs
- investigate and contain `85.203.21.23:7532`
- investigate the `UpdateCheck` service configuration
- disable or validate `Michael.Myres`
- review local Administrators membership for unauthorized changes
- review the observed registry value-set event in context and determine whether it is benign shell activity or attack-related
- assess recovery impact from the confirmed shadow-copy deletion
- hunt for the same encoded PowerShell, destination, and service parent across the environment
- investigate how the endpoint was initially compromised
