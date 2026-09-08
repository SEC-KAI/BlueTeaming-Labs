# Investigation Methodology

## Objective

Determine whether the suspicious Windows process activity represented legitimate administration or an endpoint compromise, then reconstruct the sequence using process, network, user, account, and system-change evidence.

## 1. Start with the Alerted Process

Capture the fields that make the process unique:

- `Computer`
- `User`
- `Image`
- `CommandLine`
- `ProcessID`
- `ProcessGUID`
- `ParentImage`
- `ParentCommandLine`
- `ParentProcessID`
- timestamp

Do not classify PowerShell as malicious only because PowerShell is present.

## 2. Evaluate Context

The suspicious combination in this case was:

```text
powershell.exe
+ -WindowStyle Hidden
+ -EncodedCommand
+ services.exe parent context
```

That combination justified a pivot into related telemetry.

## 3. Decode Without Executing

The `-EncodedCommand` content should be decoded as data only.

PowerShell `-EncodedCommand` commonly uses UTF-16LE. Decode the Base64 string and inspect the result without invoking it.

The decoded script in this case used:

- `Net.Sockets.TCPClient`
- stream reader / writer objects
- `ReadLine()`
- `iex`
- output returned through the network stream

## 4. Pivot on PID

Search for `ProcessID=1405` across Sysmon events.

This is stronger than searching only for `powershell.exe` because PID correlation connects a specific process instance to later activity.

## 5. Correlate Network Telemetry

Sysmon Event ID `3` tied PID `1405` to:

```text
10.0.2.48:49795 -> 85.203.21.23:7532/TCP
Initiated: true
```

This establishes that the endpoint process initiated the external connection.

## 6. Follow the Process Tree

A later PowerShell PID `4700` had PowerShell PID `1405` as its parent.

Track:

```text
services.exe (PID 6712)
        ↓
powershell.exe (PID 1405)
        ↓
powershell.exe (PID 4700)
```

Do not infer the purpose of PID `4700` beyond the evidence available.

## 7. Hunt for Post-Compromise Behavior

Search the same user, endpoint, and time range for:

- discovery commands such as `whoami`
- new account creation
- local group membership changes
- registry modifications
- `vssadmin.exe` and its exact command line
- RDP connections involving the same external infrastructure
- command shells under newly created accounts
- new files
- new services
- additional external connections

## 8. Separate Evidence Levels

Use three categories:

### Directly confirmed
Present in the raw event record.

### Confirmed in recovered screenshots
Observed directly in the screenshots now included under `screenshots/`.

### Unconfirmed
Possible interpretation that should not be written as fact.

## 9. Classification Logic

One suspicious process could be legitimate administration. The final classification was based on the chain:

```text
Encoded hidden PowerShell
        +
External connection from the same PID
        +
Decoded remote-command behavior
        +
Post-compromise discovery/account/system activity
        ↓
True Positive
```

## 10. Final Analyst Questions

Before closing the case, answer:

1. What exact process started the suspicious activity?
2. What parent launched it?
3. What did the encoded content actually do?
4. Did the same PID create network traffic?
5. Which system and user were involved?
6. What happened after the initial connection?
7. Which conclusions are directly proven?
8. Which conclusions still need more evidence?
