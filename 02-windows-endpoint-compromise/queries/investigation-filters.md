# Investigation Filters and Queries

These searches are designed around the fields observed in the training SIEM. Field names can vary between Splunk deployments, so verify the actual event schema before copying them into another environment.

## 1. Scope the Endpoint

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" Computer="IT-TRYHATME"
| sort _time
```

## 2. Scope the User

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" User="TRYHATME\\Andrew.Miller"
| sort _time
```

## 3. Find PowerShell Process Creation

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" EventId=1 Image="*\\powershell.exe"
| sort _time
| table _time Computer User ProcessID ParentProcessID Image ParentImage CommandLine ParentCommandLine
```

## 4. Find Encoded PowerShell

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" EventId=1 CommandLine="*-EncodedCommand*"
| sort _time
| table _time Computer User ProcessID ParentProcessID Image ParentImage CommandLine ParentCommandLine
```

## 5. Find Hidden PowerShell

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" EventId=1 CommandLine="*-WindowStyle Hidden*"
| sort _time
```

## 6. Pivot on PowerShell PID 1405

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" (ProcessID=1405 OR ParentProcessID=1405)
| sort _time
| table _time EventId Description Computer User ProcessID ParentProcessID Image ParentImage CommandLine DestinationIp DestinationPort
```

## 7. Pivot on Parent PID 6712

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" (ProcessID=6712 OR ParentProcessID=6712)
| sort _time
```

## 8. Find the UpdateCheck Parent Context

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" ParentCommandLine="*UpdateCheck*"
| sort _time
| table _time Computer User ProcessID ParentProcessID Image ParentImage CommandLine ParentCommandLine
```

## 9. Find the Verified External Connection

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" EventId=3 DestinationIp="85.203.21.23" DestinationPort=7532
| sort _time
| table _time Computer User ProcessID Image SourceIp SourcePort DestinationIp DestinationPort Protocol Initiated
```

## 10. Correlate PowerShell with Network Connections

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" EventId=3 Image="*\\powershell.exe"
| sort _time
| table _time Computer User ProcessID Image SourceIp SourcePort DestinationIp DestinationPort Protocol Initiated
```

## 11. Find the Follow-On PowerShell PID 4700

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" (ProcessID=4700 OR ParentProcessID=4700)
| sort _time
| table _time EventId Computer User ProcessID ParentProcessID Image ParentImage CommandLine ParentCommandLine
```

## 12. Find whoami

```spl
index=* Computer="IT-TRYHATME" (Image="*\\whoami.exe" OR CommandLine="*whoami*")
| sort _time
| table _time datasource User ProcessID ParentProcessID Image ParentImage CommandLine
```

## 13. Hunt for Account Creation

Windows Security Event ID `4720` indicates a user account was created.

```spl
index=* Computer="IT-TRYHATME" EventCode=4720
| sort _time
```

If the dataset uses `EventId` instead of `EventCode`:

```spl
index=* Computer="IT-TRYHATME" EventId=4720
| sort _time
```

Then search directly for the observed account:

```spl
index=* "Michael.Myres"
| sort _time
```

## 14. Hunt for Local Administrators Group Changes

Security Event ID `4732` records a member added to a security-enabled local group.

```spl
index=* Computer="IT-TRYHATME" (EventCode=4732 OR EventId=4732) "Michael.Myres"
| sort _time
```

Verify the group name in the raw event before concluding the exact membership change.

## 15. Review Sysmon Registry Activity

```spl
index=* datasource="Microsoft-Windows-Sysmon/Operational" (EventId=12 OR EventId=13 OR EventId=14) Computer="IT-TRYHATME"
| sort _time
| table _time EventId User ProcessID Image TargetObject Details
```

Useful meanings:

- Event ID `12` — Registry object create/delete
- Event ID `13` — Registry value set
- Event ID `14` — Registry object rename

## 16. Find vssadmin.exe

```spl
index=* Computer="IT-TRYHATME" (Image="*\\vssadmin.exe" OR CommandLine="*vssadmin*")
| sort _time
| table _time datasource User ProcessID ParentProcessID Image ParentImage CommandLine
```

In this case, the recovered screenshot shows `vssadmin.exe delete shadows /all /quiet`, which directly proves shadow-copy deletion. In other cases, verify the exact command line before making that conclusion.

## 17. Build a Narrow Incident Timeline

```spl
index=* Computer="IT-TRYHATME" (User="TRYHATME\\Andrew.Miller" OR ProcessID=1405 OR ParentProcessID=1405 OR DestinationIp="85.203.21.23" OR "Michael.Myres" OR Image="*\\vssadmin.exe")
| sort _time
```

## 18. Search for the Same Indicators Across Other Endpoints

```spl
index=* (CommandLine="*-EncodedCommand*" OR DestinationIp="85.203.21.23" OR "Michael.Myres" OR ParentCommandLine="*UpdateCheck*")
| stats count values(Computer) values(User) by Image CommandLine DestinationIp DestinationPort
```

## Investigation Reminder

A field appearing in a search result is evidence of that field value. A technique or attacker intent is an interpretation that must be supported by the surrounding sequence.
