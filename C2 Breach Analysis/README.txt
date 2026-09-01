# Command and Control Investigation Using Sysmon
## Overview
This project documents my investigation of a compromised Windows host using **Sysmon logs**.
The goal was to trace the attack from the initial downloaded file to the execution of a secondary payload.
## Tools Used
- Windows Event Viewer
- Sysmon
- Process ID / Parent Process ID correlation
## Investigation
### 1. Initial File Discovery

I started by filtering for:

```text
Sysmon Event ID 11 - File Create
```

This revealed the file:

```text
URGENT!.zip
```

The event showed that the file was downloaded through **Google Chrome**.

Because the file later led to malicious activity, phishing or social engineering was a likely delivery method, although the logs alone do not confirm this.

### 2. Archive Execution

The user extracted `URGENT!.zip`, which contained a `.lnk` shortcut file.

I followed the process activity created after the shortcut was executed to determine what happened next.

### 3. Process Tree Analysis

I correlated events using:

```text
ProcessId
ParentProcessId
```

This allowed me to trace the parent and child process relationships.

The execution chain eventually launched:

```text
powershell.exe
```

### 4. PowerShell Activity

The PowerShell command made a web request to a suspicious external domain and downloaded:

```text
update.exe
```

The executable was saved to:

```text
C:\Users\Administrator\AppData\Roaming\update.exe
```

This showed that the ZIP archive was part of a staged attack chain that retrieved an additional payload.

## Reconstructed Attack Chain

```text
Google Chrome
    ↓
URGENT!.zip downloaded
    ↓
.lnk executed
    ↓
PowerShell launched
    ↓
Web request to suspicious domain
    ↓
update.exe downloaded
```

## Key Findings

| Finding | Result |
|---|---|
| Initial File | `URGENT!.zip` |
| Download Source | Google Chrome |
| Execution Artifact | `.lnk` file |
| Command Interpreter | `powershell.exe` |
| Secondary Payload | `update.exe` |
| Payload Location | `C:\Users\Administrator\AppData\Roaming\update.exe` |

## Conclusion

Using Sysmon logs, I traced the activity from the initial ZIP download to PowerShell execution and the retrieval of `update.exe`.

The most useful part of the investigation was correlating **file creation events and parent/child process relationships** to reconstruct the attack sequence.

This project strengthened my understanding of **Sysmon analysis, process tree investigation, PowerShell activity, malware tracing, and Command and Control investigation**.
