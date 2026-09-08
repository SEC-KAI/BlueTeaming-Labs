# Network Evidence

## 1. Suspicious-link firewall correlation

The firewall event for the Microsoft-lookalike URL showed:

```text
Action: allowed
Application: web-browsing
SourceIP: 10.20.2.25
DestinationIP: 45.148.10.131
DestinationPort: 443
Protocol: TCP
URL: https://m1crosoftsupport.co/login
```

This confirms that the endpoint generated traffic to the suspicious URL. It turns a suspicious email indicator into observed user/network interaction.

Evidence: [Firewall event](../../screenshots/11-microsoft-lookalike-firewall-connection.png)

## 2. SMTP packet-analysis example

A separate Wireshark stream shows SMTP traffic between:

```text
Client: 10.12.19.101:49185
SMTP server: 173.194.66.27:25
```

The stream contains SMTP `DATA` fragments and a visible sender line:

```text
from: "Post Office" <postmaster@mozilla.org>
```

The server then responds with a `552-5.7.0` message indicating the content was blocked, followed by TCP FIN/ACK teardown.

This capture is **not** the same message as the `mutawamarine.com` header case. It is included as a separate transport-layer example showing how packet evidence can reveal SMTP behavior and server disposition.

Evidence: [SMTP packet stream](../../screenshots/06-smtp-message-blocked-packet-stream.png)
