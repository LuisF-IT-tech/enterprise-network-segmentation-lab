# Enterprise Network Segmentation Lab

## Project Overview

This project documents the design and implementation of an enterprise-style segmented home network using TP-Link Omada networking equipment.

The goal is to improve network security, organization, and scalability by separating devices into dedicated VLANs and controlling communication with firewall rules.

---

# Objectives

- Design a scalable network architecture
- Create multiple VLANs
- Configure DHCP scopes
- Configure wireless SSIDs
- Implement firewall rules
- Test VLAN isolation
- Document the complete deployment process

---

# Hardware

| Device | Model |
|---------|-------|
| Router | TP-Link ER605 |
| Switch | TP-Link TL-SG108E |
| Access Point | TP-Link EAP610 |

---

# Planned Network

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 10 | Management | 192.168.10.0/24 | Network infrastructure |
| 20 | Home | 192.168.20.0/24 | PCs, laptops, phones |
| 30 | IoT | 192.168.30.0/24 | Smart devices |
| 40 | Guest | 192.168.40.0/24 | Guest Wi-Fi |

---

# Project Status

- [x] Create GitHub repository
- [ ] Document existing network
- [ ] Design network topology
- [ ] Configure VLANs
- [ ] Configure switch ports
- [ ] Configure wireless SSIDs
- [ ] Configure firewall rules
- [ ] Test network isolation
- [ ] Complete documentation

---

# Skills Demonstrated

- Network Design
- VLAN Configuration
- DHCP
- Firewall Rules
- Wireless Networking
- Network Troubleshooting
- Technical Documentation
