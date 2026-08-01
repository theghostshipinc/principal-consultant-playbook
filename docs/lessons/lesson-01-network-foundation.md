# Lesson 01: Enterprise F5 Network Foundation

## Objective

Build and validate an enterprise-style network capable of supporting F5 BIG-IP application delivery.

The environment separates management, internal application, external client, and high-availability traffic into dedicated networks.

## Components

- Cisco IOS router
- Cisco IOS Layer 2 switch
- Two F5 BIG-IP VE appliances in an HA pair
- Multi-homed Ubuntu server
- Apache HTTP server
- GNS3
- VMware Workstation

## Network Design

| VLAN | Name | Subnet | Purpose |
|---:|---|---|---|
| 10 | Management | 10.10.10.0/24 | Device administration and BIG-IP GUI access |
| 20 | Internal | 10.10.11.0/24 | BIG-IP-to-pool-member traffic |
| 30 | External | 10.10.12.0/24 | Client-facing virtual-server traffic |
| 40 | HA | 10.10.13.0/24 | BIG-IP synchronization and failover communication |

## Ubuntu Server Interfaces

| Interface | Address | Purpose |
|---|---|---|
| ens33 | 10.10.10.15/24 | Management and administrative access |
| ens37 | 192.168.124.133/24 | VMware NAT and Internet access |
| ens38 | 10.10.11.16/24 | Apache pool-member traffic |

The internal interface does not install a default route.

## BIG-IP Internal Addresses

- `10.10.11.3` — Non-floating internal Self IP
- `10.10.11.4` — Floating internal Self IP

## Validation Completed

- [x] VLAN 20 switchport configuration
- [x] Ubuntu internal interface configuration
- [x] Layer 2 MAC-address learning
- [x] Ubuntu connectivity to the internal router interface
- [x] Ubuntu connectivity to the non-floating BIG-IP Self IP
- [x] Ubuntu connectivity to the floating BIG-IP Self IP
- [x] Apache listening on TCP port 80
- [x] BIG-IP HA pair synchronized
- [x] Management and application traffic separated

## Troubleshooting Summary

The initial connectivity test failed even though the addressing, routing, VLAN configuration, and BIG-IP Self IP configuration appeared correct.

Troubleshooting included:

1. Validating Ubuntu interface addressing and routes
2. Checking Cisco access VLAN assignments
3. Inspecting the VLAN 20 MAC-address table
4. Reviewing BIG-IP Self IP and traffic-group ownership
5. Capturing ARP and ICMP traffic on BIG-IP
6. Verifying BIG-IP virtual-interface MAC addresses
7. Correcting the GNS3 adapter-name offset caused by the BIG-IP management NIC
8. Reconnecting the Ubuntu internal virtual NIC

## Root Causes

Two separate lab-layer issues were identified:

1. BIG-IP data-interface labels were offset because the first QEMU adapter is used for management.
2. The Ubuntu internal GNS3 link had been removed during troubleshooting and needed to be reconnected.

## Lessons Learned

- Begin troubleshooting at the physical and virtual-link layers.
- An interface shown as configured inside an operating system may still lack a functioning virtual cable.
- Validate MAC learning before assuming an IP-routing issue.
- BIG-IP VE interface numbering must account for its separate management interface.
- Use packet captures and interface counters to prove where traffic stops.
- Avoid changing several layers simultaneously when evidence has already isolated the fault.

## Next Milestone

Build the first HTTP application-delivery flow:

```text
Client
  |
External Virtual Server
  |
BIG-IP Full Proxy
  |
HTTP Pool
  |
Ubuntu Apache Server
```