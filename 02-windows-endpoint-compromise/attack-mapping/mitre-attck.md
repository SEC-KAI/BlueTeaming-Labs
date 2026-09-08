# MITRE ATT&CK Mapping

This mapping is intentionally evidence-driven. A technique is marked high confidence only when the observed command, process, or network behavior directly supports it.

| ATT&CK ID | Technique | Evidence | Confidence |
|---|---|---|---|
| `T1059.001` | Command and Scripting Interpreter: PowerShell | Hidden PowerShell execution with `-EncodedCommand` | High |
| `T1027` | Obfuscated Files or Information | PowerShell command content was Base64 encoded | High |
| `T1564.003` | Hide Artifacts: Hidden Window | `-WindowStyle Hidden` | High |
| `T1095` | Non-Application Layer Protocol | PowerShell TCP client communicated directly over TCP/7532 with no higher-level protocol identified | Moderate |
| `T1033` | System Owner/User Discovery | `whoami` identified during follow-on investigation | High for the observed command; screenshot still worth adding |
| `T1136.001` | Create Account: Local Account | `net.exe user Michael.Myres ... /add` shown in recovered Sysmon screenshot | High |
| `T1021.001` | Remote Services: Remote Desktop Protocol | Later Sysmon network event involving `85.203.21.23` and `10.0.2.48:3389` | Moderate |
| `T1059.003` | Command and Scripting Interpreter: Windows Command Shell | `cmd.exe` executed under `TRYHATME\Michael.Myres` | High |
| `T1490` | Inhibit System Recovery | `vssadmin.exe delete shadows /all /quiet` | High |
| `T1098.007` | Account Manipulation: Additional Local or Cloud Roles | Case investigation recorded `Michael.Myres` added to `Administrators` | Moderate until the direct group-membership event is added |

---

## Behavior Not Mapped as Confirmed

### Service Execution / Service Persistence

The parent command line contains:

```text
C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]
```

This proves service context for the PowerShell parent, but the available evidence does not show the `UpdateCheck` service being created or modified. Therefore the repository does not claim service persistence as confirmed.

### Registry Persistence

A recovered Sysmon Event ID `13` screenshot shows a registry value set under:

```text
HKU\...\SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Cached\...
```

This confirms registry modification under `Michael.Myres`, but the observed key is not enough to claim `T1547.001` (Registry Run Keys / Startup Folder) or another persistence technique.

---

## ATT&CK Chain Summary

```text
Execution
  T1059.001 PowerShell
        ↓
Defense Evasion
  T1027 Encoded content
  T1564.003 Hidden Window
        ↓
Command and Control
  T1095 Direct TCP communication
        ↓
Discovery / Account Activity
  T1033 whoami
  T1136.001 Local account creation
        ↓
Remote Access / Execution
  T1021.001 RDP activity
  T1059.003 Windows Command Shell
        ↓
Impact
  T1490 Inhibit System Recovery
```

The account-to-Administrators change remains documented as case context, but its specific ATT&CK mapping stays at moderate confidence until the direct group-membership event is added.
