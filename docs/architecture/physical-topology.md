# Physical Topology

## Purpose

This document describes the physical layout of the F5 Principal Consultant Lab.

---

## Topology

![Physical Topology](images/physical-topology-v1.png)

---

## VLANs

| VLAN | Purpose |
|------|----------|
|10|Management|
|20|Internal|
|30|External|
|40|HA|

---

## BIG-IP Interfaces

| Interface | VLAN |
|-----------|------|
|1.1|Unused|
|1.2|Internal|
|1.3|HA|
|1.4|External|
|Mgmt|Management|

---

## Pool

WEB_POOL

10.10.10.15:80

---

## Notes

- Active/Standby HA
- SSL terminated on BIG-IP
- Apache backend
- Single pool member (Phase 1)