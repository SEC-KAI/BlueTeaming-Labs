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

## Accuracy Note

The asset table is part of the supplied scenario. Several solved investigation values in the available source are redacted. This portfolio does **not** assume that an asset's descriptive hostname proves that it was the exact answer to a redacted investigation question.

For example, the existence of `VPN-CLIENT-ATTK` in the inventory is documented, but this file does not use that fact alone to reconstruct a redacted assigned-IP answer.
