# Log Analysis Commands

These commands are **reproducible investigation filters built from the confirmed case indicators**. They are not presented as a verbatim record of every command used during the original lab.

## 1. Search the suspected SSH source IP

```bash
grep '10.14.94.82' /var/log/auth.log
```

Purpose: isolate authentication events associated with the source IP identified in the incident.

## 2. Review failed SSH authentication

```bash
grep '10.14.94.82' /var/log/auth.log | grep -E 'Failed password|authentication failure|Invalid user'
```

Purpose: identify repeated failed authentication attempts from the source.

## 3. Search the compromised account

```bash
grep 'jack-brown' /var/log/auth.log
```

Purpose: review authentication and privilege-related activity involving the compromised account.

## 4. Identify successful SSH authentication

```bash
grep 'jack-brown' /var/log/auth.log | grep -E 'Accepted password|Accepted publickey'
```

Purpose: isolate the successful SSH login that followed the failed attempts.

## 5. Review sudo activity

```bash
grep 'jack-brown' /var/log/auth.log | grep 'sudo'
```

Purpose: identify privilege-related activity performed after the successful login.

## 6. Search for the newly created account

Depending on Linux distribution and logging configuration, account-management events may appear in `auth.log`, `secure`, the system journal, or audit logs.

```bash
grep -R 'ssh-remote' /var/log 2>/dev/null
```

Or with the journal:

```bash
journalctl | grep 'ssh-remote'
```

Purpose: locate events associated with creation or use of the `ssh-remote` account.

## 7. Review cron activity

```bash
grep -i 'cron' /var/log/syslog
```

If searching for the known persistence-related account or destination:

```bash
grep -i 'cron' /var/log/syslog | grep -E 'ssh-remote|10\.10\.33\.31|7654'
```

Purpose: identify scheduled activity related to the persistence sequence.

## 8. Inspect user crontabs

```bash
crontab -l
```

For another user, when authorized:

```bash
sudo crontab -u ssh-remote -l
```

Purpose: verify the configured cron entries directly.

## 9. Search the suspicious destination in collected logs

```bash
grep -R '10.10.33.31' /var/log 2>/dev/null
```

```bash
grep -R '7654' /var/log 2>/dev/null
```

Purpose: identify log entries containing the confirmed outbound destination.

## 10. Useful Splunk-style pivots

If the Linux events are indexed in Splunk, the same investigation can be reproduced using fields available in the dataset.

### Source IP pivot

```spl
index=<linux_index> "10.14.94.82"
```

### Account pivot

```spl
index=<linux_index> "jack-brown"
```

### New account pivot

```spl
index=<linux_index> "ssh-remote"
```

### Persistence / network pivot

```spl
index=<linux_index> ("cron" OR "CRON") ("10.10.33.31" OR "7654" OR "ssh-remote")
```

`<linux_index>` is intentionally left generic because the exact index name for the original incident is not confirmed in the recovered case material.
