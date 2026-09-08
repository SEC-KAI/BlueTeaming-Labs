# Cron Persistence Evidence

### `01-suspicious-cron-reverse-shell.png`

Shows a Splunk search focused on Linux CRON activity and suspicious commands. The visible results include execution of a shell script from `/tmp` and a Perl socket command that connects to `10.10.101.12:9999` and launches `/bin/sh -i`.

**Demonstrates:**
- searching Linux syslog/cron telemetry;
- identifying suspicious scheduled execution;
- correlating cron activity with reverse-shell behavior.

**Evidence scope:** The IP/port in this screenshot belongs to a separate Linux persistence exercise. It must **not** be confused with the confirmed destination `10.10.33.31:7654` from the `jack-brown` incident.
