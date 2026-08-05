# F5 Lab IP Addressing Plan

## Purpose

This document defines the VLANs, subnets, gateways, device interfaces, and service addresses used by the F5 Principal Consultant Lab.

---

# VLAN and Subnet Summary

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|--------|---------|---------|
| 10 | Management | | | Device management |
| 20 | Internal | | | BIG-IP server-side and backend servers |
| 30 | External | | | BIG-IP client-side and VIP network |
| 40 | HA | | N/A | BIG-IP failover and configuration synchronization |

---

# Cisco Router

| Interface | VLAN / Network | IP Address | Purpose |
|-----------|----------------|------------|---------|
| | | | |

---

# Cisco Switch

| Interface | Mode | VLAN(s) | Connected Device / Interface |
|-----------|------|---------|------------------------------|
| | | | |

---

# BIG-IP 01

| Interface | VLAN / Network | Self IP | Floating Self IP | Purpose |
|-----------|----------------|---------|------------------|---------|
| Management | Management | | N/A | Administrative access |
| 1.2 | Internal | | | Server-side traffic |
| 1.3 | HA | | N/A | Failover and configuration sync |
| 1.4 | External | | | Client-side traffic |

---

# BIG-IP 02

| Interface | VLAN / Network | Self IP | Floating Self IP | Purpose |
|-----------|----------------|---------|------------------|---------|
| Management | Management | | N/A | Administrative access |
| 1.2 | Internal | | | Server-side traffic |
| 1.3 | HA | | N/A | Failover and configuration sync |
| 1.4 | External | | | Client-side traffic |

---

# Ubuntu Web Servers

| Server | Interface | VLAN / Network | IP Address | Gateway | Service |
|--------|-----------|----------------|------------|---------|---------|
| Web01 | | Internal | 10.10.10.15 | | Apache TCP/80 |
| Web02 | | Internal | Planned | | Apache TCP/80 |

---

# BIG-IP Application Objects

| Object Type | Name | Address / Members | Port | Notes |
|-------------|------|-------------------|------|-------|
| Pool | WEB_POOL | 10.10.10.15 | 80 | Current single-member pool |
| Virtual Server | | | 443 | Client-side HTTPS VIP |
| Floating Self IP | | | N/A | External traffic |
| Floating Self IP | | | N/A | Internal traffic |

---

# Routing Summary

## Client-to-VIP Path

```text
Client
  → Cisco Router
  → BIG-IP External Floating Self IP / VIP
```

## BIG-IP-to-Server Path

```text
BIG-IP Internal Self IP
  → Cisco Switch
  → Ubuntu Web01
```

## Return Path

Document whether the lab uses:

- SNAT Automap
- SNAT Pool
- Direct Server Return
- Static Routes

---

# Addressing Rules

- Management addresses are never used for application traffic.
- Each BIG-IP traffic VLAN has unique non-floating Self IPs.
- Floating Self IPs provide high availability.
- The HA VLAN is isolated.
- New pool members receive fixed IP addresses.

---

# Open Items

- [ ] Confirm subnet masks.
- [ ] Confirm router interface addresses.
- [ ] Confirm BIG-IP Self IPs.
- [ ] Confirm Floating Self IPs.
- [ ] Confirm VIP.
- [ ] Confirm Ubuntu default gateway.
- [ ] Reserve Web02 address.