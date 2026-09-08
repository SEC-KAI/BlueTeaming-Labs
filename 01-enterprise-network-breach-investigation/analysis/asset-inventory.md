# Asset Inventory

The following systems were provided by the lab scenario as reference assets.

| IP Address | Hostname | Role | Operating System | Team | Criticality |
|---|---|---|---|---|---|
| `10.0.0.20` | `FINANCE-SRV1` | File / Finance Server (SMB) | Windows Server | Finance IT | High |
| `10.0.0.50` | `VPN-GW` | VPN Gateway | Linux | NetOps | Critical |
| `10.0.0.51` | `APP-WEB-01` | Internal Web / App | Linux | Apps Team | High |
| `10.0.0.60` | `WORKSTATION-60` | Employee Workstation | Windows 10 | Sales | Medium |
| `10.8.0.23` | `VPN-CLIENT-ATTK` | VPN Assigned Client (Ephemeral) | N/A | N/A | Critical |
| `10.0.1.10` | `DMZ-WEB` | DMZ Web Server | Linux | NetOps | Medium |

## Verified Investigation Roles

The completed lab and supplied screenshots confirm the following roles in the incident:

- `10.0.0.20` (`FINANCE-SRV1`) — internal host targeted by reconnaissance scans.
- `10.8.0.23` (`VPN-CLIENT-ATTK`) — internal VPN address assigned after the successful login.
- `10.0.0.60` (`WORKSTATION-60`) — internal host that generated C2 beaconing to `198.51.100.77:4444`.
- `10.0.0.51` (`APP-WEB-01`) — host that showed exfiltration-attempt activity toward `198.51.100.77` over ports 80/8080.

The external reconnaissance source `203.0.113.45` and C2 IP `198.51.100.77` are not internal assets, so they are not part of the scenario asset table above.
