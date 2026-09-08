# Cleartext FTP Analysis

## Problem

Understand what an analyst can observe when authentication and file-transfer activity uses traditional plaintext FTP.

## What I Investigated

I filtered FTP traffic, reviewed authentication responses, and learned how the control connection exposes commands that can be reconstructed in a TCP stream.

## Evidence

- `../../screenshots/09-ftp-login-failure-analysis.png` shows repeated FTP `530 Login incorrect` responses between `10.121.70.151` and `10.234.125.254`.
- `../../screenshots/00-wireshark-traffic-analysis-room-scope.png` shows completion of the cleartext FTP analysis section.

## Findings

FTP's control channel is not encrypted by default. Commands such as `USER` and `PASS` can therefore be visible to an observer with access to the packet capture. The saved screenshot specifically demonstrates repeated unsuccessful authentication attempts; it does not, by itself, prove credential theft.

Repeated `530` responses can also support authentication-failure or brute-force analysis when combined with the number of attempts, timing, and source context.

## Useful Investigation Steps

```text
ftp
ftp.request.command == "USER"
ftp.request.command == "PASS"
tcp.port == 21
```

Following the TCP stream can reconstruct the control conversation in order.

## Defensive Recommendations

- Remove plaintext FTP where possible.
- Prefer encrypted file-transfer methods such as SFTP or an appropriately configured TLS-protected service.
- Alert on high FTP authentication-failure counts.
- Restrict FTP exposure to systems that actually require it.

## What I Learned

The security issue is not merely that FTP uses port 21. The key issue is that its control conversation can reveal authentication information unless encryption is added through a safer protocol or configuration.
