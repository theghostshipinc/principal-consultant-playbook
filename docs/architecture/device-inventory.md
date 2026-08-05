# Device Inventory

| Property | Value |
|----------|-------|
| Project | Principal Consultant Playbook |
| Sprint | Sprint 2 |
| Artifact | S2-A002 |
| Version | 1.0 |
| Status | Draft |
| Author | Greg Mitchell |
| Last Updated | 2026-08-04 |

---

# Purpose

This document inventories every device that participates in the F5 Enterprise Lab.

---

# Virtual Machines

| Device | Vendor | Function | Operating System | Status | Notes |
|---------|--------|----------|------------------|--------|-------|
| BIG-IP01 | F5 | Active Load Balancer | TMOS | Active | HA Pair |
| BIG-IP02 | F5 | Standby Load Balancer | TMOS | Standby | HA Pair |
| Ubuntu-Web01 | Ubuntu | Apache Web Server | Ubuntu Linux | Active | WEB_POOL member |
| Cisco Router | Cisco | Layer 3 Routing | IOS | Active | Client gateway |
| Cisco Switch | Cisco | Layer 2 Switching | IOS | Active | VLAN connectivity |

---

# Network Services

| Service | Device | Port |
|---------|--------|------|
| HTTPS Virtual Server | BIG-IP | 443 |
| Apache | Ubuntu-Web01 | 80 |
| SSH | All Managed Devices | 22 |

---

# Current Pool Members

| Pool | Member | Status |
|------|--------|--------|
| WEB_POOL | Ubuntu-Web01 | Active |

---

# Planned Devices

| Device | Purpose | Planned Sprint |
|---------|----------|----------------|
| Ubuntu-Web02 | Second Pool Member | Sprint 2 |
| Database Server | Backend Database | Future |
| Client Workstation | Client Testing | Future |

---

# Notes

- BIG-IP operates in Active/Standby mode.
- SSL is terminated on the BIG-IP.
- Apache currently serves HTTP to the BIG-IP.
- Ubuntu-Web02 will be added during Sprint 2.