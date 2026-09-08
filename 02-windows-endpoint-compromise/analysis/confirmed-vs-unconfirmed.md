# Confirmed vs. Unconfirmed Findings

This file intentionally separates direct evidence from interpretation so the repository does not overstate what the investigation proved.

## Confirmed from Preserved Sysmon Evidence

### Process execution

- `powershell.exe` executed on `IT-TRYHATME`.
- The command line used `-WindowStyle Hidden`.
- The command line used `-EncodedCommand`.
- The process ran as `TRYHATME\Andrew.Miller`.
- The PowerShell process ID was `1405`.
- `ParentImage` was `C:\Windows\system32\services.exe`.
- The parent PID was `6712`.
- The parent command line was `C:\Windows\system32\services.exe -k netsvcs [UpdateCheck]`.
- The event also exposed a `ParentProcess` field written as `C:\Windows\system32\service.exe`; because `ParentImage` and `ParentCommandLine` both show `services.exe`, this repository preserves the discrepancy instead of silently treating the fields as identical.

### Network activity

- Sysmon Event ID `3` recorded PowerShell PID `1405` initiating a TCP connection.
- The endpoint source IP was `10.0.2.48`.
- The external destination was `85.203.21.23`.
- The destination port was `7532`.
- The source port was `49795`.
- `Initiated` was `true`.

### Decoded command behavior

- The Base64 command decodes as UTF-16LE PowerShell.
- It creates a `Net.Sockets.TCPClient`.
- It reads command strings from a network stream.
- It executes received commands using `iex`.
- It writes command output back to the stream.
- It contains retry logic with a five-second sleep.

### Follow-on process

- A later PowerShell process had PID `4700`.
- Its parent was PowerShell PID `1405`.
- The observed command line again contained the encoded PowerShell logic.

## Confirmed from Recovered Screenshots

The recovered investigation screenshots directly show:

- `net.exe` creating `Michael.Myres` with `user ... /add`
- later RDP activity involving `85.203.21.23` and `10.0.2.48:3389`
- `cmd.exe` executing under `TRYHATME\Michael.Myres`
- Sysmon Event ID `13` setting a registry value under the `Michael.Myres` user hive
- `vssadmin.exe List Shadows`
- `vssadmin.exe delete shadows /all /quiet`

The prior case notes also recorded `whoami` and addition of `Michael.Myres` to `Administrators`. A direct screenshot/export for those two events is still recommended.

## Important Data Discrepancy

The decoded PowerShell content includes a malformed address literal:

```text
=5.203.21.23
```

The Sysmon Event ID `3` network record independently shows:

```text
85.203.21.23:7532
```

This repository keeps both observations separate instead of silently changing the decoded text.

## Not Confirmed by Current Evidence

Do **not** state these as established facts unless additional evidence is added:

- the initial access method
- that `UpdateCheck` was definitely created by the attacker
- that `UpdateCheck` was definitely a legitimate service that was hijacked
- registry-based persistence
- the exact purpose of the observed registry value-set event
- ransomware execution
- data exfiltration
- credential theft
- lateral movement to another host
- the full scope of compromise beyond `IT-TRYHATME`
- attribution of the external IP to a specific threat actor or malware family

## Final Evidence-Based Conclusion

The combination of encoded hidden PowerShell, service-based parent context, PID-correlated outbound communication, decoded remote-command logic, new-account creation, later RDP/shell activity, and confirmed shadow-copy deletion is sufficient to classify the case as a **true-positive endpoint compromise requiring escalation** without making unsupported claims about initial access, registry persistence, ransomware, or data theft.
