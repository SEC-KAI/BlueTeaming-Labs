# User Activity & Persistence Evidence

These screenshots come from related Linux investigation exercises and demonstrate the techniques used to analyze post-compromise activity. They are not presented as screenshots from the exact `jack-brown` incident.

### `01-auditd-process-tree-analysis.png`

Shows Auditd process records linking a Python application process to `/bin/sh -c whoami` and then `whoami`, using PID/PPID relationships.

**Demonstrates:** Linux process-tree reconstruction and command-execution analysis.

### `02-suspicious-systemd-service-creation.png`

Shows Auditd records for creation of a systemd service file under `/etc/systemd/system/`, including a `wget` operation and root context.

**Demonstrates:** detection of service-based persistence and file-creation activity.

### `03-systemd-service-analysis.png`

Shows direct inspection of `tux.service` and `badr.service`, including service execution directives and suspicious cleanup behavior.

**Demonstrates:** validation of suspicious persistence by inspecting the service configuration itself.
