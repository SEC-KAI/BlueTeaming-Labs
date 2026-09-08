# Authentication Results

## Observed results

The selected header evidence shows:

```text
Received-SPF: fail
Authentication-Results: ... spf=fail smtp.mailfrom=mutawamarine.com;
dmarc=unknown
```

The sender domain's SPF lookup showed:

```text
v=spf1 include:spf.protection.outlook.com -all
```

## Interpretation

The hard-fail (`-all`) SPF policy means mail sources outside the authorized SPF path should fail SPF for that domain. The observed header result is therefore consistent with the source not being authorized to send for `mutawamarine.com`.

Important accuracy points:

- SPF failure is strong evidence that the envelope sender was not authorized by the domain's SPF policy.
- `dmarc=unknown` is recorded as unknown, not rewritten as `dmarc=fail`.
- No DKIM result is visible in the selected evidence, so no DKIM-pass or DKIM-fail claim is made.

## Screenshots

- [SPF/DMARC result](../../screenshots/03-spf-dmarc-authentication-results.png)
- [SPF policy](../../screenshots/04-mutawamarine-spf-policy.png)
