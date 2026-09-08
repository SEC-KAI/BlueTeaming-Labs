# Asset and Indicator Inventory

## Compromised Endpoint

| Field | Value | Evidence Status |
|---|---|---|
| Computer | `IT-TRYHATME` | Confirmed in Sysmon |
| Endpoint IP | `10.0.2.48` | Confirmed in Sysmon Event ID 3 |
| User | `TRYHATME\Andrew.Miller` | Confirmed in Sysmon |
| Suspicious process | `powershell.exe` | Confirmed |
| PowerShell PID | `1405` | Confirmed |
| Parent process | `services.exe` | Confirmed by `ParentImage` |
| ParentProcess field | `C:\Windows\system32\service.exe` | Preserved as recorded; differs from `ParentImage` / `ParentCommandLine` |
| Parent PID | `6712` | Confirmed |
| Service context | `UpdateCheck` | Confirmed in parent command line |
| Follow-on PowerShell PID | `4700` | Confirmed |

## Network Indicator

| Field | Value | Evidence Status |
|---|---|---|
| External IP | `85.203.21.23` | Confirmed in Sysmon Event ID 3 |
| Destination port | `7532/TCP` | Confirmed |
| Source port | `49795` | Confirmed |
| Endpoint initiated connection | `true` | Confirmed |

## Account Indicator

| Field | Value | Evidence Status |
|---|---|---|
| Account | `Michael.Myres` | Confirmed created by recovered Sysmon screenshot |
| Account creation process | `net.exe` | Confirmed in Sysmon Event ID 1 screenshot |
| Later shell user | `TRYHATME\Michael.Myres` | Confirmed in Sysmon Event ID 1 screenshot |
| Privilege change | Added to `Administrators` | Recorded in case context; separate group-add event screenshot still recommended |

## Additional Post-Compromise Indicators

| Field | Value | Evidence Status |
|---|---|---|
| RDP destination | `10.0.2.48:3389` | Confirmed in recovered Sysmon Event ID 3 screenshot |
| RDP-related source IP | `85.203.21.23` | Confirmed |
| Command shell | `cmd.exe` under `TRYHATME\Michael.Myres` | Confirmed |
| Registry activity | Sysmon Event ID `13` value set under `HKU\...\Shell Extensions\Cached\...` | Confirmed |
| Shadow-copy enumeration | `vssadmin.exe List Shadows` | Confirmed |
| Shadow-copy deletion | `vssadmin.exe delete shadows /all /quiet` | Confirmed |

## Analysis-Platform Context

The SIEM search interface exposed a host value of `10.10.39.16:8989`. This project does **not** treat that value as the compromised endpoint because the event itself identifies the endpoint through `Computer: IT-TRYHATME` and `SourceIp: 10.0.2.48`.

## Important Evidence Distinction

The decoded PowerShell string contains a malformed IP literal beginning with `=5.203.21.23`. The actual network event separately records `DestinationIp: 85.203.21.23`, so the Sysmon network event is used as the authoritative connection evidence.
