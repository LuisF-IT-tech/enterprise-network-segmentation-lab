# Implementation Progress

## Purpose

This document records the configuration changes completed during the network-segmentation implementation.

Changes were performed in controlled stages, with connectivity testing and configuration backups created at major checkpoints.

Actual wireless SSID names are intentionally excluded from this public documentation.

---

## Implementation Status

| Task | Status |
|---|---|
| Create configuration backups | Complete |
| Create Guest VLAN 40 | Complete |
| Create Production VLAN 50 | Complete |
| Enable 802.1Q on the managed switch | Complete |
| Configure tagged VLAN trunks | In progress |
| Migrate wireless networks | Complete |
| Create VLAN 10 administration port | Complete |
| Move AP management traffic to VLAN 10 | Complete |
| Move switch management traffic fully into VLAN 10 | In progress |
| Move wired PC to Home VLAN 20 | Pending |
| Move printer to Home VLAN 20 | Pending |
| Configure firewall rules | Pending |
| Complete validation testing | Pending |

---

## Backup Checkpoints

Configuration backups were created before beginning the redesign.

Additional backups were created after the successful wireless migration.

Backups include:

- TP-Link ER605 router
- TP-Link TL-SG108E switch
- TP-Link EAP610 access point

The backup files are stored privately and are not included in this public repository.

---

## Router VLAN Configuration

The following VLAN interfaces currently exist on the ER605:

| VLAN ID | Documentation Name | Subnet | Gateway | DHCP |
|---:|---|---|---|---|
| 1 | Default LAN | `192.168.0.0/24` | `192.168.0.1` | Enabled |
| 10 | Management | `192.168.10.0/24` | `192.168.10.1` | Enabled |
| 20 | Trusted Home | `192.168.20.0/24` | `192.168.20.1` | Enabled |
| 30 | IoT | `192.168.30.0/24` | `192.168.30.1` | Enabled |
| 40 | Guest | `192.168.40.0/24` | `192.168.40.1` | Enabled |
| 50 | Production Services | `192.168.50.0/24` | `192.168.50.1` | Enabled |

Public DNS servers are being used temporarily.

A future AdGuard Home deployment will provide internal DNS services and will be documented in a separate GitHub lab.

---

## Switch VLAN Migration

The TL-SG108E originally had:

- Port-based VLAN mode enabled
- All ports placed into the same VLAN group
- 802.1Q VLAN mode disabled
- PVID 1 configured on every port

Port-based VLAN mode was disabled and 802.1Q VLAN mode was enabled.

### Initial Transition-Safe Configuration

The switch was first configured to match the existing ER605 trunk configuration.

| VLAN | Port 1 — Router | Port 2 — AP | Other ports |
|---:|---|---|---|
| 1 | Untagged | Untagged | Untagged |
| 10 | Tagged | Tagged | Not member |
| 20 | Tagged | Tagged | Not member |
| 30 | Tagged | Tagged | Not member |
| 40 | Tagged | Tagged | Not member |
| 50 | Tagged | Not member | Not member |

All PVIDs initially remained set to VLAN 1.

This preserved the existing wired and wireless connectivity while allowing the switch to formally process 802.1Q VLAN tags.

---

## Wireless VLAN Migration

Three wireless network roles were configured on the EAP610.

| Wireless role | VLAN | Subnet |
|---|---:|---|
| Trusted wireless | 20 | `192.168.20.0/24` |
| IoT wireless | 30 | `192.168.30.0/24` |
| Guest wireless | 40 | `192.168.40.0/24` |

Devices were migrated in stages:

1. A test Guest SSID was created on VLAN 40.
2. Internet and DHCP connectivity were verified.
3. A test IoT SSID was created on VLAN 30.
4. IoT devices were migrated individually and tested.
5. A trusted Home SSID was created on VLAN 20.
6. Trusted household devices were migrated and tested.
7. Permanent SSID names were applied.
8. A new EAP610 backup was created.

### Wireless Validation

The following results were confirmed:

| Network | Expected result | Status |
|---|---|---|
| Trusted wireless | Receives `192.168.20.x` | Passed |
| IoT wireless | Receives `192.168.30.x` | Passed |
| Guest wireless | Receives `192.168.40.x` | Passed |
| Trusted Internet access | Working | Passed |
| IoT Internet access | Working | Passed |
| Guest Internet access | Working | Passed |

Firewall isolation has not yet been completed. Successful VLAN assignment does not by itself guarantee security between networks.

---

## Temporary VLAN 10 Administration Port

An unused switch port was configured as a dedicated VLAN 10 administration and recovery port.

### Switch Port 5

| Setting | Value |
|---|---|
| VLAN 1 membership | Not member |
| VLAN 10 membership | Untagged |
| PVID | 10 |
| Purpose | Temporary administration and recovery |

A computer connected to this port receives a `192.168.10.x` address and can access the management interfaces of the network devices.

This port provides a known-good recovery path while the infrastructure trunk configuration is being migrated.

---

## EAP610 Management VLAN Migration

The EAP610 switch port was changed so that the access point's own management traffic is placed into VLAN 10.

### Switch Port 2

| VLAN | Membership |
|---:|---|
| VLAN 1 | Not member |
| VLAN 10 | Untagged |
| VLAN 20 | Tagged |
| VLAN 30 | Tagged |
| VLAN 40 | Tagged |
| VLAN 50 | Not member |

Port 2 PVID:

```text
10
```

This configuration means:

- Untagged management traffic from the EAP610 enters VLAN 10.
- Trusted wireless traffic remains tagged as VLAN 20.
- IoT wireless traffic remains tagged as VLAN 30.
- Guest wireless traffic remains tagged as VLAN 40.

### Post-Change Validation

After the change:

- The EAP610 management page remained reachable.
- Trusted wireless devices remained connected.
- IoT devices remained connected.
- Guest wireless devices retained Internet access.
- DHCP continued to assign addresses from the correct VLAN scopes.

---

## Current Physical Port Plan

| Switch port | Connected device | Current role |
|---:|---|---|
| 1 | ER605 router | Infrastructure trunk |
| 2 | EAP610 access point | Management-native wireless trunk |
| 3 | Main PC | Temporary default LAN connection |
| 4 | Printer | Temporary default LAN connection |
| 5 | Administration computer | VLAN 10 recovery/admin port |
| 6–8 | Unused | Reserved |

---

## Production VLAN 50

VLAN 50 was created for stable production services.

Planned uses include:

- AdGuard Home
- Uptime monitoring
- Tailscale
- NAS services
- Stable servers and containers

VLAN 50 is currently configured as:

- Tagged on the ER605-to-switch trunk
- Tagged on switch port 1
- Not carried to the access point
- Not yet assigned to an access port

Production services will preferably use wired connections.

---

## Issues Encountered and Lessons Learned

### Java Runtime Missing

The TP-Link Omada Discovery Utility could not launch because Java was not installed or available in the Windows system path.

A general-purpose IP scanner was used instead.

### Management IP and VLAN Mismatch

The switch and access point used `192.168.10.x` addresses while their management traffic was traveling untagged on the default Layer 2 network.

This allowed local access from a manually assigned `192.168.10.x` address but prevented access to the router's tagged VLAN 10 gateway.

### Secondary IP Addressing

A temporary second IP address was added to the Windows Ethernet adapter.

This allowed simultaneous access to:

- The Internet and default LAN through `192.168.0.x`
- The switch and access point through `192.168.10.x`

This eliminated the need to repeatedly change network-adapter settings during troubleshooting.

### Safe VLAN Migration

The switch configuration could not be converted in one step without risking loss of management access.

A transition-safe configuration and a dedicated recovery port were used instead.

---

## Next Implementation Steps

The next phase will:

1. Complete the management VLAN transition on the ER605-to-switch trunk.
2. Verify switch and router management access through VLAN 10.
3. Move the wired PC from VLAN 1 to VLAN 20.
4. Move the printer from VLAN 1 to VLAN 20.
5. Configure a wired access port for VLAN 50 when the first production server is deployed.
6. Create least-privilege inter-VLAN firewall rules.
7. Test allowed and denied traffic flows.
8. Remove obsolete configuration after validation.
