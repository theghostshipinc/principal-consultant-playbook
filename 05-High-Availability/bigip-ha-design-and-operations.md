# BIG-IP High-Availability Design and Operations Guide

**Project:** Principal Consultant Playbook  
**Platform:** F5 BIG-IP Virtual Edition  
**BIG-IP version:** 16.0.0.1, Build 0.0.3  
**Deployment model:** Two-device active/standby high-availability pair  
**Device group:** `F5-Device-Group`  
**Validation state:** Healthy and `In Sync` at the time of capture

---

## Purpose

This guide documents the verified high-availability (HA) design of the lab and provides repeatable procedures for validation, configuration synchronization, controlled failover, and troubleshooting.

The guide is intentionally specific to the captured lab state. It does not assume application objects, VLAN names, floating Self IP addresses, persistence settings, connection-mirroring settings, or network topology details that were not present in the supplied command output.

## Scope

This document covers:

- Device identity and active/standby roles
- Device Trust
- Sync-Failover device-group behavior
- ConfigSync and automatic synchronization
- Network failover communication
- Traffic-group ownership and preference
- Configuration and operational validation
- Controlled synchronization and failover procedures
- Common HA states and recovery principles

It does not prove application availability. A successful failover test must include application-specific validation once pools, virtual servers, and client test paths are documented.

---

## Verified Lab State

### Device inventory

| Device | Role at capture | Management IP | ConfigSync IP | Mirror IP | Version |
|---|---|---:|---:|---:|---|
| `ATL-F5-01.local` | Active | `10.10.10.10` | `10.10.13.2` | `10.10.13.2` | 16.0.0.1 Build 0.0.3 |
| `ATL-F5-02.local` | Standby | `10.10.10.9` | `10.10.13.3` | `10.10.13.3` | 16.0.0.1 Build 0.0.3 |

The appliance hostnames are `ATL-F5-01.local` and `ATL-F5-02.local`. Short labels such as **F5-01** and **F5-02** in this guide refer to those devices.

### HA control objects

| Object | Verified configuration |
|---|---|
| Trust domain | `Root` |
| Trust-domain status | `initialized` |
| Trust members | `ATL-F5-01.local`, `ATL-F5-02.local` |
| Trust group | `device_trust_group` |
| Application device group | `F5-Device-Group` |
| Device-group type | `sync-failover` |
| Automatic sync | `enabled` |
| Save on automatic sync | `true` |
| Members | `ATL-F5-01.local`, `ATL-F5-02.local` |

### Traffic groups

| Traffic group | Verified configuration | Operational meaning |
|---|---|---|
| `traffic-group-1` | HA order: F5-01, then F5-02; unit ID `1` | F5-01 is first in the configured HA preference order. At capture, F5-01 owned this traffic group. |
| `traffic-group-local-only` | `is-floating false` | Objects in this group remain local and do not float to the peer. |

The HA order is a preference, not proof that every future transition will automatically return ownership to F5-01. Administrative actions, device health, and failover behavior still determine current ownership.

### Captured operational health

On F5-01:

```text
Status   ACTIVE
Summary  1/1 active
Details  active for /Common/traffic-group-1
```

On F5-02:

```text
Status   STANDBY
Summary  1/1 standby
```

Both devices reported:

```text
Status   In Sync
Mode     high-availability
Summary  All devices in the device group are in sync
```

The output also showed each peer as connected and both `F5-Device-Group` and `device_trust_group` as `In Sync`.

---

## Design Explanation

### Active/standby architecture

The pair uses a Sync-Failover device group. At the time of validation, F5-01 was active for `traffic-group-1`, while F5-02 was standby. The active device owns the floating objects assigned to the traffic group; the standby device is prepared to assume that ownership after a failover event.

The supplied output confirms the traffic-group roles. It does not enumerate the individual floating configuration objects within `traffic-group-1`, so none are asserted here.

### Device Trust

Device Trust establishes the authenticated relationship required for the devices to exchange configuration and participate in the HA group. The `Root` trust domain was initialized and listed both devices as CA devices. The trust domain uses `device_trust_group` as its trust group.

Trust health and application configuration synchronization are related but distinct. A healthy trust relationship is foundational; it does not by itself guarantee that `F5-Device-Group` is synchronized.

### Sync-Failover device group

`F5-Device-Group` contains both appliances and has type `sync-failover`. This combines:

- Configuration synchronization between members
- Coordinated traffic-group failover

`auto-sync enabled` means eligible changes are automatically synchronized to the group. `save-on-auto-sync true` means the receiving system saves configuration after an automatic sync. Operators should still validate sync status after every material change; enabled automation is not proof of successful completion.

### ConfigSync and mirroring addresses

The captured device objects assign:

- F5-01 ConfigSync and mirror address: `10.10.13.2`
- F5-02 ConfigSync and mirror address: `10.10.13.3`

ConfigSync transfers eligible configuration between device-group members. The mirror IP is the device-level address selected for connection mirroring. The supplied output does not show whether connection mirroring is enabled on any application object, so this guide does not claim that application connections are mirrored.

### Failover communication

Each device listed two unicast failover addresses: its management address and its HA address. The operational output reported both paths as `Ok`:

| Local device | Local failover path | Remote peer | Captured status |
|---|---|---|---|
| F5-01 | `10.10.10.10:1026` | F5-02 | `Ok` |
| F5-01 | `10.10.13.2:1026` | F5-02 | `Ok` |
| F5-02 | `10.10.10.9:1026` | F5-01 | `Ok` |
| F5-02 | `10.10.13.3:1026` | F5-01 | `Ok` |

Multiple working paths reduce dependence on a single failover communication path. The packet counters and recent-packet timestamps in the capture demonstrated live communication at that moment; they are not permanent health guarantees.

### Floating and non-floating objects

`traffic-group-1` is the HA traffic group shown as active on F5-01. `traffic-group-local-only` was explicitly non-floating. In operational terms:

- Floating objects assigned to a floating traffic group can move with traffic-group ownership.
- Non-floating objects remain device-specific.

No Self IP inventory was supplied with the HA capture. Validate the actual assignment of each Self IP or other object before relying on it during failover:

```bash
tmsh list net self traffic-group
```

Treat this command as an additional inspection step, not as captured evidence for this document.

---

## Routine Validation

Run configuration inspection commands from either device unless a procedure says otherwise.

### Inspect device and HA configuration

```bash
tmsh list cm device
tmsh list cm device-group
tmsh list cm trust-domain
tmsh list cm traffic-group
```

Confirm:

- Both expected devices appear with the correct management, ConfigSync, and mirror addresses.
- `F5-Device-Group` contains both members and remains `sync-failover`.
- Automatic sync and save-on-auto-sync retain their intended settings.
- Trust domain `Root` is initialized and contains both devices.
- `traffic-group-1` lists F5-01 before F5-02 in `ha-order`.
- `traffic-group-local-only` remains non-floating.

### Validate failover state on both devices

```bash
tmsh show cm failover-status
```

Healthy expected state for the captured design:

- Exactly one device is `ACTIVE` for `traffic-group-1`.
- The peer is `STANDBY`.
- Failover connections on both management and HA addresses show `Ok`.
- Received-packet timestamps and counters continue to advance during observation.

### Validate synchronization on both devices

```bash
tmsh show cm sync-status
```

Healthy expected state:

```text
Color    green
Status   In Sync
Mode     high-availability
```

Confirm the peer is connected and both relevant device groups report `In Sync`.

### Compact pre-change checklist

- [ ] Both devices are reachable through their management interfaces.
- [ ] One device is active and the other is standby.
- [ ] Both failover paths report `Ok` from each device.
- [ ] Both devices report `In Sync`.
- [ ] Device Trust is initialized and contains both devices.
- [ ] A current recovery point exists for the planned change.
- [ ] The application owner and test method are identified before failover testing.

---

## Command Compatibility Notes

### `failover-status` is a show/statistics object

The following command is invalid on the captured system:

```bash
tmsh list cm failover-status
```

It returned:

```text
Syntax Error: "failover-status" unexpected argument
```

Use the operational command instead:

```bash
tmsh show cm failover-status
```

`list` is used for configuration objects. `failover-status` exposes operational/statistical state and must be queried with `show` on this version.

### Do not use `sys db failover` as a generic check

The following command did not exist on BIG-IP 16.0.0.1 in this lab:

```bash
tmsh list sys db failover
```

It returned:

```text
01020036:3: The requested database variable (failover) was not found.
```

Do not include that command as a generic HA validation step. Database variables are version- and implementation-specific; query a specific, documented variable only when its applicability has been established.

---

## Controlled Configuration Synchronization

Although auto-sync is enabled, a controlled manual sync may be useful after recovery or when automatic synchronization has not completed.

### Safety rules

1. Identify which device contains the authoritative configuration.
2. Review recent changes before syncing.
3. Never assume the active device is automatically the correct source.
4. Do not push from a stale or partially recovered device.
5. Preserve a recovery point before resolving an unexplained conflict.

### Procedure

On both devices, record the current state:

```bash
tmsh show cm failover-status
tmsh show cm sync-status
```

If the source of truth is known, run the following **on that source device**:

```bash
tmsh run cm config-sync to-group F5-Device-Group
```

Then validate on both devices:

```bash
tmsh show cm sync-status
```

Success criteria:

- Both devices report `In Sync`.
- Each device reports its peer connected.
- `F5-Device-Group` and `device_trust_group` report `In Sync`.
- Intended objects exist and are correct on both members.

Stop and investigate if the authoritative source is unclear, a device is disconnected, the group reports a conflict, or synchronization would overwrite a known-good configuration.

---

## Controlled Failover Test

A failover test is a planned service-impacting change. Perform it only in an approved test window with console or independent management access to both devices. Define application success criteria before starting.

### Phase 1: Baseline

On both devices:

```bash
tmsh show cm failover-status
tmsh show cm sync-status
```

Proceed only if:

- F5-01 is active and F5-02 is standby, matching the captured baseline.
- Both devices are `In Sync`.
- Both failover paths report `Ok`.
- The standby is reachable and stable.

Record application behavior before the change using the lab's documented client-side test. The supplied HA output does not define that test, so no URL or virtual server is invented here.

### Phase 2: Move service to F5-02

From the currently active F5-01, place the device into standby:

```bash
tmsh run sys failover standby
```

Immediately validate on both devices:

```bash
tmsh show cm failover-status
tmsh show cm sync-status
```

Expected HA result:

- F5-02 becomes active for `traffic-group-1`.
- F5-01 becomes standby.
- There is not a sustained condition with both devices active or both unavailable.
- Failover communication remains healthy.
- Sync state returns to or remains `In Sync`.

Run the predetermined application test and record any interruption, failed requests, or state loss. HA control-plane success alone does not prove application success.

### Phase 3: Restore the captured preferred ownership

First confirm that F5-01 is healthy, standby, reachable, and `In Sync`. Then, from the currently active F5-02, run:

```bash
tmsh run sys failover standby
```

Validate again on both devices:

```bash
tmsh show cm failover-status
tmsh show cm sync-status
```

Success criteria:

- F5-01 is active for `traffic-group-1`.
- F5-02 is standby.
- Both devices are `In Sync`.
- Both failover paths report `Ok`.
- The application test passes.

Do not force the return if F5-01 is unhealthy or out of sync. Keep service on the healthy device and investigate.

### Test record

Capture at minimum:

- Date, operator, and change reference
- Initial and final traffic-group owner
- Pre- and post-test sync status
- Status of all failover paths
- Application test method and result
- Observed interruption or session impact
- Any corrective actions

---

## Troubleshooting States

### `Disconnected`

**Meaning:** A peer or device group is not communicating normally.

**Checks:**

```bash
tmsh show cm sync-status
tmsh show cm failover-status
tmsh list cm device
tmsh list cm device-group
tmsh list cm trust-domain
```

Verify management and HA reachability, device identity, trust membership, and the configured failover/ConfigSync addresses. Avoid rebuilding trust until connectivity, time, identity, and certificate-related evidence have been reviewed.

### `Changes Pending`

**Meaning:** A device has configuration changes that have not yet been synchronized to the group.

**Response:** Identify the device with the intended authoritative changes. Review what changed, allow auto-sync time to complete, or perform a controlled `to-group` sync from the verified source. Do not blindly sync from whichever device is active.

### `Awaiting Initial Sync`

**Meaning:** The device group has not completed its first synchronization or a member requires initialization.

**Response:** Confirm trust and membership, determine which device holds the correct baseline, preserve backups, and initialize synchronization only from the verified source.

### `Sync Failure`

**Meaning:** A synchronization attempt failed rather than merely remaining pending.

**Response:** Preserve the error output and inspect system logs. Check peer connectivity, configuration validity, storage/resource health, and version compatibility before retrying. Repeated retries without finding the cause can obscure evidence.

Useful evidence collection:

```bash
tmsh show cm sync-status
tmsh show cm failover-status
tail -n 100 /var/log/ltm
```

### `In Sync` but unexpected configuration

**Meaning:** The peers agree on synchronized state, but the synchronized content may still be wrong.

**Response:** Remember that synchronization consistency is not configuration correctness. Compare the affected objects with the approved design or backup, identify the last known-good state, and use the normal change/recovery process.

### Failover path not `Ok`

**Meaning:** One configured heartbeat path is impaired.

**Response:** Determine which local address is affected, test the relevant network path, and inspect interface/VLAN/routing/firewall dependencies appropriate to that address. A second healthy path may preserve HA communication, but redundancy is degraded and should be repaired before planned changes.

### Both devices appear active

**Meaning:** Potential split-brain or an observation made from incomplete/stale data.

**Response:** Treat this as high risk. Stop configuration changes, obtain console access, capture failover status from both devices, and verify network paths before taking corrective action. Protect the data plane from duplicate ownership according to the lab network design. Do not issue repeated failover commands without understanding current traffic-group ownership.

### Both devices appear standby

**Meaning:** No device may currently own the service traffic group, or status may be transitional.

**Response:** Check both devices directly, confirm traffic-group details and device health, and determine whether a device is administratively forced offline or standby. Restore service on the known-good device using a controlled, evidence-based recovery action.

### Trust-domain problem

**Meaning:** The authenticated device relationship is unhealthy or inconsistent.

**Response:** Validate names, management reachability, time, certificates, and current trust membership. Back up both devices before removing or recreating trust. Rebuilding trust is a recovery operation, not a first diagnostic step.

---

## Recovery Principles

1. **Preserve evidence first.** Record status, errors, logs, and recent changes before modifying HA state.
2. **Separate control-plane questions.** Device Trust, ConfigSync, failover communication, traffic-group ownership, and application health are related but independently testable.
3. **Establish a source of truth.** Never synchronize until the authoritative configuration is known.
4. **Prefer reversible actions.** Use current backups and documented checkpoints before trust reconstruction, forced synchronization, or configuration restoration.
5. **Validate both devices.** A healthy prompt on one appliance is not proof that the pair is healthy.
6. **Validate the application.** `ACTIVE`, `STANDBY`, and `In Sync` describe HA state; they do not prove end-to-end delivery.
7. **Avoid unsupported generic commands.** Confirm command and database-variable availability for the installed BIG-IP version.
8. **Return deliberately.** Restore preferred traffic-group ownership only after the preferred device is healthy and synchronized.

---

## Operational Checklist

### Before a configuration change

- [ ] Identify the authoritative device and intended change.
- [ ] Confirm both devices are reachable.
- [ ] Confirm one active and one standby device.
- [ ] Confirm both devices are `In Sync`.
- [ ] Confirm both failover paths are `Ok`.
- [ ] Create or verify an appropriate recovery point.

### After a configuration change

- [ ] Confirm automatic synchronization completed.
- [ ] Confirm both devices returned to `In Sync`.
- [ ] Verify the changed objects on both members.
- [ ] Confirm failover roles did not change unexpectedly.
- [ ] Validate the affected service.

### After a failover test

- [ ] Confirm the intended device owns `traffic-group-1`.
- [ ] Confirm the peer is standby.
- [ ] Confirm both failover paths are `Ok`.
- [ ] Confirm both devices are `In Sync`.
- [ ] Confirm application validation passed.
- [ ] Save the test evidence and record any interruption.

---

## Verified Baseline Summary

The captured lab state demonstrates a healthy two-device BIG-IP HA pair running version 16.0.0.1:

- F5-01 was active at management address `10.10.10.10`.
- F5-02 was standby at management address `10.10.10.9`.
- ConfigSync and mirror addresses were `10.10.13.2` and `10.10.13.3`.
- `F5-Device-Group` was a two-member Sync-Failover group with auto-sync enabled and save-on-auto-sync set to true.
- Trust domain `Root` was initialized with both devices.
- `traffic-group-1` preferred F5-01 before F5-02 in its HA order and was active on F5-01.
- `traffic-group-local-only` was non-floating.
- Management and HA failover paths reported `Ok` from both appliances.
- Both appliances reported `In Sync`.

This baseline is the reference point for future change validation. Any later deviation should be understood in the context of an approved design change or treated as a condition requiring investigation.
