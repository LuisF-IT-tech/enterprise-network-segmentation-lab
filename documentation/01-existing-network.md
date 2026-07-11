# Existing Network Assessment

## Purpose

This document records the state of the network before any new configuration changes are made.

Creating a baseline makes it possible to understand the existing environment, identify configuration problems, plan changes safely, and compare the network before and after implementation.

Sensitive information such as public IP addresses, passwords, and full MAC addresses has intentionally been excluded.

---

## Assessment Summary

The network still contains several parts of a previous VLAN configuration.

The ER605 router currently has VLAN interfaces for VLANs 10, 20, and 30. The EAP610 access point is tagging wireless traffic for VLANs 10 and 20.

However, the TL-SG108E switch does not currently have 802.1Q VLAN functionality enabled. All switch ports are operating in one flat Layer 2 network with PVID 1.

The switch and access point use static management IP addresses in the `192.168.10.0/24` subnet, but their management traffic appears to be traveling untagged. This creates a mismatch between their IP addressing and the router's tagged VLAN 10 interface.

---

## Hardware Inventory

| Device | Model | Hardware Version | Firmware Version | Management IP |
|---|---|---|---|---|
| Router | TP-Link ER605 | v2.20 | 2.2.5 Build 20240522 | `192.168.0.1` and `192.168.10.1` |
| Switch | TP-Link TL-SG108E | v6.0 | 1.0.0 Build 20211209 | `192.168.10.3` |
| Access Point | TP-Link EAP610 | v3.0 | 1.4.4 Build 20240312 | `192.168.10.2` |
| Server | Dell Wyse 5070 | Not yet assessed | Not configured | Not configured |
| Printer | OKI B432 | Not yet assessed | Not recorded | DHCP |

---

## Current Physical Topology

```text
Internet
   |
TP-Link ER605
   |
   | Switch Port 1
   |
TP-Link TL-SG108E
   |
   +-- Port 2: TP-Link EAP610
   +-- Port 3: Main PC
   +-- Port 4: OKI B432 Printer
   +-- Ports 5-8: Currently unused
```

### Current Switch Link Speeds

| Switch Port | Connected Device | Link Status |
|---:|---|---|
| 1 | ER605 Router | 1 Gbps |
| 2 | EAP610 Access Point | 1 Gbps |
| 3 | Main PC | 1 Gbps |
| 4 | OKI B432 Printer | 100 Mbps |
| 5-8 | Unused | Link down |

The printer's 100 Mbps connection is not necessarily a problem because many printers use Fast Ethernet interfaces.

---

## Current Router Networks

| Network | VLAN ID | Gateway | Subnet Mask | DHCP |
|---|---:|---|---|---|
| Default LAN | 1 | `192.168.0.1` | `255.255.255.0` | Enabled |
| VLAN10 | 10 | `192.168.10.1` | `255.255.255.0` | Enabled |
| VLAN20 | 20 | `192.168.20.1` | `255.255.255.0` | Enabled |
| VLAN30 | 30 | `192.168.30.1` | `255.255.255.0` | Enabled |

A VLAN 40 network does not currently exist.

### Default LAN DHCP Scope

| Setting | Value |
|---|---|
| Starting address | `192.168.0.100` |
| Ending address | `192.168.0.199` |
| Lease time | 120 minutes |
| Gateway | Router default |
| DNS | Router default |

### VLAN 10 DHCP Scope

| Setting | Value |
|---|---|
| Starting address | `192.168.10.100` |
| Ending address | `192.168.10.199` |
| Lease time | 2880 minutes |
| Gateway | `192.168.10.1` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `8.8.4.4` |

DHCP is enabled for VLANs 20 and 30, but their complete DHCP settings were not recorded during this assessment.

---

## ER605 Port VLAN Configuration

The ER605 LAN ports currently carry multiple VLANs.

| VLAN | ER605 LAN Ports 2-5 |
|---:|---|
| VLAN 1 | Untagged |
| VLAN 10 | Tagged |
| VLAN 20 | Tagged |
| VLAN 30 | Tagged |

This means ordinary untagged devices connected through these ports normally enter VLAN 1, while VLANs 10, 20, and 30 are transported using 802.1Q tags.

---

## Switch Configuration

### Management Settings

| Setting | Value |
|---|---|
| IP assignment | Static |
| Management IP | `192.168.10.3` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |

### VLAN Settings

| Setting | Current State |
|---|---|
| 802.1Q VLAN | Disabled |
| Port-based VLAN | Enabled |
| Port-based VLAN membership | Ports 1-8 in VLAN group 1 |
| PVID | 1 on all ports |

Although port-based VLAN mode is enabled, every port belongs to the same group. The switch is therefore operating as one flat Layer 2 network.

The switch appears to forward tagged frames from the EAP610 to the ER605, but it is not actively enforcing VLAN membership, trunk ports, access ports, or PVID assignments.

---

## Access Point Configuration

### Management Settings

| Setting | Value |
|---|---|
| IP assignment | Static |
| Management IP | `192.168.10.2` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `8.8.4.4` |

### Wireless Networks

| SSID | Bands | VLAN ID | Guest Mode |
|---|---|---:|---|
| MainNet | 2.4 GHz and 5 GHz | 10 | Disabled |
| GuestWiFi | 2.4 GHz and 5 GHz | 20 | Enabled |

The EAP610 adds VLAN 10 tags to MainNet wireless traffic and VLAN 20 tags to GuestWiFi traffic.

There is currently no wireless SSID assigned to VLAN 30.

---

## PC Network Baseline

Under normal DHCP operation, the main PC receives:

| Setting | Value |
|---|---|
| IP network | `192.168.0.0/24` |
| Default gateway | `192.168.0.1` |
| DHCP server | `192.168.0.1` |
| DNS server | `192.168.0.1` |

This confirms that the PC currently operates on the default untagged LAN, VLAN 1.

---

## Management Access Issue

During discovery, the PC was temporarily assigned:

```text
IP address: 192.168.10.50
Subnet mask: 255.255.255.0
```

With this temporary address, the PC could access:

- EAP610 at `192.168.10.2`
- TL-SG108E at `192.168.10.3`

However, it could not access:

- ER605 VLAN 10 gateway at `192.168.10.1`
- ER605 default LAN gateway at `192.168.0.1`
- The Internet

This behavior suggests that the switch and access point management interfaces are using `192.168.10.x` addresses while their management traffic is still traveling untagged.

The ER605's `192.168.10.1` interface expects tagged VLAN 10 traffic. Assigning a `192.168.10.x` IP address to the PC did not cause the PC to add an 802.1Q VLAN 10 tag.

This is a Layer 2 and Layer 3 configuration mismatch that will be corrected during the redesign.

---

## Existing Access-Control Rules

Two LAN-to-LAN allow rules currently exist.

| Rule Name | Source Network | Destination Network | Service | Policy |
|---|---|---|---|---|
| Proxmox | VLAN 10 | VLAN 30 | All | Allow |
| PC | VLAN 30 | VLAN 10 | All | Allow |

These rules allow broad two-way communication between VLANs 10 and 30.

They are not limited to one PC or one Proxmox server. Every device in the selected networks is included.

No deny rules were found during the assessment. The current VLANs should therefore not be considered securely isolated.

---

## Security and Design Observations

- VLAN 10 is currently used for both MainNet wireless clients and infrastructure management addresses.
- VLAN 20 is currently used as the guest wireless network.
- VLAN 30 contains a Proxmox server reservation but has no wireless SSID.
- VLAN 40 does not currently exist.
- The switch is not actively enforcing 802.1Q VLAN segmentation.
- The AP and switch management addressing does not align correctly with their Layer 2 VLAN placement.
- Existing access-control rules are broad and do not follow least-privilege principles.
- No deny rules currently isolate the management, home, IoT, and guest networks.
- The existing VLAN configuration is partially functional but incomplete.

---

## Planned Next Phase

The next phase will design a four-VLAN architecture:

| VLAN | Planned Purpose |
|---:|---|
| 10 | Management |
| 20 | Home and trusted devices |
| 30 | IoT devices |
| 40 | Guest devices |

The design phase will define:

- IP addressing
- DHCP scopes
- Switch trunk ports
- Switch access ports
- Wireless SSID assignments
- Management access
- Inter-VLAN firewall rules
- Testing and rollback procedures
