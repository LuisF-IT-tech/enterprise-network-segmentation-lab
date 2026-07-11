# Proposed Network Design

## Purpose

This document defines the planned architecture for the home network before any configuration changes are made.

The design separates trusted devices, network infrastructure, IoT devices, and guests into dedicated VLANs. Firewall rules will control communication between those networks according to the principle of least privilege.

---

## Design Goals

- Separate devices according to purpose and trust level
- Protect network-management interfaces from normal users and guests
- Prevent IoT devices from initiating connections to trusted devices
- Provide guests with Internet access without access to the internal network
- Allow trusted administrators to manage all network infrastructure
- Create a scalable foundation for future homelab services
- Avoid relying on the default VLAN for normal network traffic
- Document testing and rollback procedures before implementation

---

## Proposed VLAN Architecture

| VLAN ID | Name | Subnet | Gateway | Purpose |
|---:|---|---|---|---|
| 10 | Management | `192.168.10.0/24` | `192.168.10.1` | Network equipment, servers, and homelab infrastructure |
| 20 | Home | `192.168.20.0/24` | `192.168.20.1` | Trusted computers, phones, tablets, and printer |
| 30 | IoT | `192.168.30.0/24` | `192.168.30.1` | Smart TVs, streaming devices, doorbells, and smart-home equipment |
| 40 | Guest | `192.168.40.0/24` | `192.168.40.1` | Guest-owned devices with Internet-only access |

VLAN 1 will not be used for normal client traffic after migration. It may remain temporarily available on a dedicated ER605 port for recovery during implementation.

---

## IP Addressing Plan

Each VLAN uses a `/24` subnet with the following address structure:

| Address range | Purpose |
|---|---|
| `.1` | Default gateway |
| `.2-.49` | Network equipment and servers |
| `.50-.99` | DHCP reservations |
| `.100-.199` | General DHCP clients |
| `.200-.254` | Reserved for future use |

### VLAN 10 — Management and Homelab

| Device or role | Planned address |
|---|---|
| ER605 gateway | `192.168.10.1` |
| EAP610 access point | `192.168.10.2` |
| TL-SG108E switch | `192.168.10.3` |
| Future Omada Controller | `192.168.10.10` |
| Future AdGuard Home server | `192.168.10.11` |
| Future homelab servers | `192.168.10.20-49` |
| DHCP clients | `192.168.10.100-199` |

The exact addresses of future servers may change when those services are deployed.

### VLAN 20 — Home

| Device or role | Planned addressing |
|---|---|
| Gateway | `192.168.20.1` |
| Main PC | DHCP reservation |
| Trusted phones and laptops | DHCP |
| OKI B432 printer | DHCP reservation |
| DHCP range | `192.168.20.100-199` |

The printer will initially remain on the Home VLAN to simplify printing and device discovery. It may later be moved to the IoT VLAN after the required firewall rules are tested.

### VLAN 30 — IoT

| Device or role | Planned addressing |
|---|---|
| Gateway | `192.168.30.1` |
| Smart TVs and streaming devices | DHCP |
| Ring doorbell | DHCP reservation |
| Amazon and smart-home devices | DHCP |
| DHCP range | `192.168.30.100-199` |

### VLAN 40 — Guest

| Device or role | Planned addressing |
|---|---|
| Gateway | `192.168.40.1` |
| Guest devices | DHCP |
| DHCP range | `192.168.40.100-199` |

---

## Proposed Physical Topology

```text
Internet
   |
TP-Link ER605
   |
   | Switch-facing trunk
   |
TP-Link TL-SG108E
   |
   +-- Port 1: ER605 trunk
   +-- Port 2: EAP610 trunk
   +-- Port 3: Main PC — Home VLAN 20
   +-- Port 4: OKI B432 Printer — Home VLAN 20
   +-- Port 5: Future homelab server — Management VLAN 10
   +-- Ports 6-8: Reserved for future devices
```

---

## Proposed ER605 Uplink

The ER605 port connected to the TL-SG108E will carry:

| VLAN | Port behavior |
|---:|---|
| 10 | Untagged/native |
| 20 | Tagged |
| 30 | Tagged |
| 40 | Tagged |

VLAN 10 will be the native or untagged network on the infrastructure trunks. This allows the switch and access point to use untagged management traffic while remaining inside the Management VLAN.

---

## Proposed Switch Port Configuration

| Switch port | Connected device | VLAN membership | PVID | Port role |
|---:|---|---|---:|---|
| 1 | ER605 | VLAN 10 untagged; VLANs 20, 30, and 40 tagged | 10 | Trunk |
| 2 | EAP610 | VLAN 10 untagged; VLANs 20, 30, and 40 tagged | 10 | Trunk |
| 3 | Main PC | VLAN 20 untagged | 20 | Access |
| 4 | OKI B432 printer | VLAN 20 untagged | 20 | Access |
| 5 | Future homelab server | VLAN 10 untagged | 10 | Access |
| 6 | Future device | Assigned when required | TBD | Unassigned |
| 7 | Future device | Assigned when required | TBD | Unassigned |
| 8 | Future device | Assigned when required | TBD | Unassigned |

### Port terminology

- **Tagged:** The Ethernet frame includes an 802.1Q VLAN tag.
- **Untagged:** The frame is transmitted without a VLAN tag.
- **PVID:** The VLAN assigned to untagged traffic entering that switch port.
- **Trunk port:** Carries traffic for multiple VLANs.
- **Access port:** Carries one untagged VLAN for an ordinary endpoint.

---

## Proposed Wireless Networks

| SSID | VLAN | Intended devices |
|---|---:|---|
| MainNet | 20 | Trusted phones, laptops, and tablets |
| IoT | 30 | Smart TVs, streaming devices, doorbells, and smart-home equipment |
| GuestWiFi | 40 | Guest-owned devices |

The EAP610 management interface will remain untagged on VLAN 10.

Both 2.4 GHz and 5 GHz may be enabled for MainNet and GuestWiFi. The IoT SSID will support 2.4 GHz because many smart devices do not support 5 GHz.

---

## Proposed Traffic Policy

VLAN creation separates broadcast domains, but firewall and access-control rules are required to enforce security between those networks.

### High-Level Policy

| Source | Internet | Management VLAN 10 | Home VLAN 20 | IoT VLAN 30 | Guest VLAN 40 |
|---|---:|---:|---:|---:|---:|
| Management | Allow | Allow | Allow | Allow | Allow |
| Home | Allow | Deny by default | Allow | Limited access | Deny |
| IoT | Allow | Deny | Deny | Allow | Deny |
| Guest | Allow | Deny | Deny | Deny | Client isolation |

### Management VLAN 10

Management devices and administrators may initiate connections to all internal VLANs.

Examples include:

- Accessing the ER605
- Managing the TL-SG108E
- Managing the EAP610
- Managing future homelab servers
- Troubleshooting devices in other VLANs

Other VLANs should not be permitted to initiate unrestricted connections into the Management VLAN.

### Home VLAN 20

Trusted Home devices will have:

- Internet access
- Access to the Home printer
- Limited access to approved IoT devices and services
- No unrestricted access to network-management interfaces

### IoT VLAN 30

IoT devices will have:

- Internet access
- DNS and time synchronization
- No ability to initiate connections to the Home VLAN
- No ability to access network-management interfaces
- No unrestricted access to homelab servers

Trusted Home devices may later receive narrowly scoped permission to initiate connections to specific IoT devices.

### Guest VLAN 40

Guest devices will receive:

- Internet access
- DHCP and DNS services
- No access to Management
- No access to Home
- No access to IoT
- Wireless client isolation where supported

---

## Planned Access-Control Strategy

Rules will be created using the following model:

```text
Source network -> Destination network -> Service -> Action
```

Specific allow rules will be placed before broader deny rules.

Example:

```text
Home PC -> Approved IoT device -> Required service -> Allow
Home VLAN -> IoT VLAN -> All other traffic -> Deny
```

The existing broad allow rules between VLAN 10 and VLAN 30 will be reviewed and removed after replacement rules are tested.

---

## DNS Plan

During the VLAN implementation, clients will initially use the ER605 or approved public DNS resolvers.

AdGuard Home will be deployed later as a separate GitHub lab. After deployment, DHCP scopes may be updated to distribute the AdGuard server address as DNS.

Planned AdGuard address:

```text
192.168.10.11
```

Firewall rules will then permit DNS traffic from the Home, IoT, and Guest VLANs to that specific server.

---

## Migration Strategy

The network will be changed in controlled stages.

1. Verify that all three device backups are available.
2. Create VLAN 40 and its DHCP scope on the ER605.
3. Prepare the ER605 switch-facing port for the planned trunk.
4. Enable 802.1Q on the TL-SG108E.
5. Configure switch port 1 as the ER605 trunk.
6. Confirm management access before changing additional ports.
7. Configure switch port 2 as the EAP610 trunk.
8. Move the MainNet SSID from VLAN 10 to VLAN 20.
9. Move GuestWiFi from VLAN 20 to VLAN 40.
10. Create the IoT SSID on VLAN 30.
11. Configure wired access ports.
12. Test DHCP, Internet access, DNS, and VLAN assignment.
13. Create and test firewall rules.
14. Remove obsolete rules and unused configuration.
15. Update final documentation.

Only one major change will be made at a time. Connectivity will be tested before proceeding to the next change.

---

## Recovery and Rollback Plan

The following recovery options will be available during implementation:

- Restore the ER605 configuration backup
- Restore the TL-SG108E configuration backup
- Restore the EAP610 configuration backup
- Connect directly to an unused ER605 LAN port
- Temporarily assign a static IP when required for management
- Revert the most recent change before continuing
- Factory reset a device only as a last resort

Configuration backups will remain private and will not be uploaded to GitHub.

---

## Validation Plan

Each VLAN will be tested for:

- Correct DHCP address
- Correct default gateway
- Correct DNS server
- Internet access
- Access to permitted destinations
- Blocking of prohibited destinations
- Router-management restrictions
- Wireless VLAN assignment
- Wired switch-port assignment

Example validation commands:

```powershell
ipconfig /all
ping <gateway-address>
tracert 8.8.8.8
nslookup example.com
Test-NetConnection <destination-address> -Port <port>
```

Testing results will be recorded in a separate validation document.

---

## Expected Final Outcome

The completed network will provide:

- A dedicated management and homelab network
- A trusted home network
- An isolated IoT network
- An Internet-only guest network
- Controlled inter-VLAN communication
- Clearly documented switch-port assignments
- A safe foundation for the future AdGuard Home deployment
