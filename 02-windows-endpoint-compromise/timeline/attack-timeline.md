# Attack Timeline

## Timeline Notes

The initial process/network timestamps come from preserved Sysmon text, while the later timestamps below were recovered from the investigation screenshots now included under `screenshots/`. `whoami` and the separate Administrators-group membership event were identified during the case investigation but are still listed without invented timestamps because their direct screenshots are not currently included.

| Time | Event | Evidence | Assessment |
|---|---|---|---|
| `2026-09-08 00:06:20.254` | Hidden encoded PowerShell process created | Sysmon Event ID 1 | Suspicious execution |
| `2026-09-08 00:06:25.254` | PowerShell PID `1405` initiated TCP connection to `85.203.21.23:7532` | Sysmon Event ID 3 | External remote-command / C2 evidence |
| `2026-09-08 00:08:30.254` | PowerShell PID `4700` created with parent PowerShell PID `1405` | Sysmon Event ID 1 | Follow-on process activity |
| `2026-09-08 00:10:40.254` | `net.exe` command created `Michael.Myres` | Sysmon Event ID 1 screenshot | New account creation |
| `2026-09-08 00:23:05.254` | RDP network event involving `85.203.21.23` and `10.0.2.48:3389` | Sysmon Event ID 3 screenshot | Follow-on remote-access activity |
| `2026-09-08 00:25:42.254` | Registry value set under `TRYHATME\Michael.Myres` | Sysmon Event ID 13 screenshot | Registry modification; persistence not established |
| `2026-09-08 00:30:53.254` | `cmd.exe` executed under `TRYHATME\Michael.Myres` | Sysmon Event ID 1 screenshot | Windows command shell activity |
| `2026-09-08 00:36:13.254` | `vssadmin.exe List Shadows` | Sysmon Event ID 1 screenshot | Shadow-copy discovery |
| `2026-09-08 00:42:57.254` | `vssadmin.exe delete shadows /all /quiet` | Sysmon Event ID 1 screenshot | Inhibit System Recovery |
| Time not preserved | `whoami` executed | Case investigation notes | User/context discovery |
| Time not preserved | `Michael.Myres` added to `Administrators` | Case investigation notes | Privilege/persistence concern; direct group-add screenshot still recommended |

---

## Reconstructed Sequence

```text
services.exe -k netsvcs [UpdateCheck]
PID 6712
        ↓
powershell.exe
PID 1405
-WindowStyle Hidden
-EncodedCommand
        ↓
Outbound connection
10.0.2.48:49795 -> 85.203.21.23:7532/TCP
        ↓
PowerShell child activity
PID 4700
        ↓
Michael.Myres created
        ↓
RDP activity involving 85.203.21.23 -> 10.0.2.48:3389
        ↓
Registry value set under Michael.Myres
        ↓
cmd.exe under Michael.Myres
        ↓
vssadmin.exe List Shadows
        ↓
vssadmin.exe delete shadows /all /quiet
```

## Timeline Interpretation

The strongest causal link is between the first PowerShell process and the external TCP connection because the same PID, `1405`, appears in both events. The later screenshots extend the incident from initial remote-command activity into account creation, remote-access activity, a command shell under the new account, and explicit deletion of Volume Shadow Copies.

The sequence is strongly consistent with post-compromise activity, but the repository still avoids claiming an initial-access method or a registry-persistence mechanism without direct evidence.
