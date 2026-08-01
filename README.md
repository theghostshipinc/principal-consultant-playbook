# Principal Consultant Playbook

**By Greg Mitchell — Theghostshipinc**

## Purpose

The Principal Consultant Playbook documents my development as a Principal Network Consultant specializing in application delivery, enterprise networking, automation, and cloud infrastructure.

The first major learning track focuses on F5 BIG-IP and the technical and consulting skills required to design, deploy, validate, and troubleshoot enterprise application-delivery solutions.

## Learning Method

Every major topic follows the same process:

1. Understand the architecture
2. Define the customer requirement
3. Design the solution
4. Build the lab
5. Validate expected behavior
6. Break and troubleshoot the solution
7. Document lessons learned
8. Explain the solution as a consultant

## Repository Structure

- `00-Environment` — Workstation, GNS3, VMware, and lab documentation
- `01-F5-Foundations` — BIG-IP architecture and foundational concepts
- `02-LTM` — Local Traffic Manager design and implementation
- `03-DNS` — BIG-IP DNS and global traffic-management concepts
- `04-SSL` — Certificates, TLS, SSL offload, and SSL bridging
- `05-High-Availability` — Device trust, configuration sync, and failover
- `06-iRules` — Traffic-management logic and Tcl fundamentals
- `07-Troubleshooting` — Packet analysis, logs, diagnostics, and case studies
- `08-Automation` — APIs, AS3, Terraform, Ansible, and scripting
- `Customer-Engagements` — Sanitized consulting scenarios and deliverables
- `Portfolio` — Polished, non-confidential examples of completed work
- `Templates` — HLD, LLD, MOP, validation, and rollback templates
- `Scripts` — Reusable lab and automation utilities
- `Images` — Diagrams and repository images
- `Theghostshipinc` — Future article, video, and educational-content ideas

## Current Focus

The current learning path is focused on:

- BIG-IP full-proxy architecture
- F5 LTM administration and design
- Citrix ADC to F5 BIG-IP migration concepts
- SSL/TLS application delivery
- High availability
- Packet-level troubleshooting
- Automation and repeatable deployment practices

## Current Lab Status

### Infrastructure

- Cisco Layer 2 switching
- Cisco Layer 3 routing
- F5 BIG-IP HA pair
- Multi-homed Ubuntu Apache server
- Dedicated Management, Internal, External, and HA VLANs

### Validated

- [x] VLAN connectivity
- [x] Layer 2 MAC-address learning
- [x] Layer 3 routing
- [x] BIG-IP non-floating Self IP connectivity
- [x] BIG-IP floating Self IP connectivity
- [x] Ubuntu multi-homed networking
- [x] Apache HTTP service
- [x] BIG-IP HA synchronization
- [x] Management and application-network separation

### Next Milestone

- Create an HTTP pool
- Add the Ubuntu Apache pool member
- Configure an HTTP health monitor
- Create an external virtual server
- Validate HTTP traffic through BIG-IP
- Prove the two-session full-proxy model

## Confidentiality

This repository contains personal lab work and independently created educational material only.

No customer names, proprietary configurations, credentials, production data, internal company documents, or confidential engagement details should be committed to this repository.

## Brand

**Theghostshipinc** is an emerging engineering and educational identity created by Greg Mitchell. Its future direction may include technical writing, lab demonstrations, and practical enterprise-networking education.
