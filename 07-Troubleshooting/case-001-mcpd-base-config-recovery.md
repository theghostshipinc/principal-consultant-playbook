# Case Study 001 — BIG-IP MCPD Base Configuration Recovery

| Property | Value |
|---|---|
| Project | Principal Consultant Playbook |
| Case Study | 001 |
| Platform | F5 BIG-IP VE |
| Affected Device | ATL-F5-02 |
| Severity | Lab Service Degradation |
| Status | Resolved |
| Date | 2026-08-04 |
| Author | Greg Mitchell |

---

## Executive Summary

ATL-F5-02 entered an `INOPERATIVE:Standalone` state after instability within the GNS3 environment.

The BIG-IP management daemon (`mcpd`) was running, but the system remained in the platform phase because its base and user configuration files were empty. The affected appliance could not load its configuration, participate normally in high availability, or provide Self IP connectivity.

The active configuration files were restored from valid `.bak` copies, successfully loaded, and synchronized with the healthy HA peer. ATL-F5-02 returned to the Standby state and the device group returned to `In Sync`.

---

## Environment

The lab contained:

- ATL-F5-01 — Active BIG-IP
- ATL-F5-02 — Standby BIG-IP
- Cisco router
- Cisco Layer 2 switch
- Ubuntu Apache server
- VMware Workstation Pro
- GNS3 2.2.61
- Management, Internal, External, and HA networks

---

## Initial Symptoms

ATL-F5-02 displayed:

```text
INOPERATIVE:Standalone
```

The appliance could not reach the internal gateway:

```text
From 10.10.10.9 Destination Host Unreachable
```

The MCP state showed:

```text
Running Phase                   platform
Last Configuration Load Status  base-config-load-failed
End Platform ID Received        true
```

The `mcpd` process itself was running:

```text
mcpd run
```

This indicated that the daemon had not crashed. Instead, it was unable to progress beyond the platform phase because configuration loading had failed.

---

## Investigation

### License Validation

The appliance had a valid BIG-IP VE trial license.

No licensing failure was identified.

### Filesystem Validation

Disk usage did not indicate an exhausted filesystem.

The `/config` filesystem had sufficient available capacity.

### Log Review

The `/var/log/ltm` file repeatedly showed that the failover daemon was waiting for MCPD:

```text
Waiting for mcpd to reach phase base, current phase is platform.
```

These messages were symptoms of the stalled configuration load rather than the original cause.

### Configuration Validation

The following command was executed:

```bash
tmsh load sys config verify partitions all
```

The result included:

```text
Unexpected Error: No user configuration is found.
Validating process is aborted.
```

This indicated that there was no active user configuration available for validation.

---

## Root Cause Evidence

The active BIG-IP configuration files were inspected:

```bash
ls -lh /config/bigip*
```

The following active files were zero bytes:

```text
/config/bigip_base.conf
/config/bigip.conf
/config/bigip_user.conf
```

Valid backup copies were still present:

```text
/config/bigip_base.conf.bak
/config/bigip.conf.bak
/config/bigip_user.conf.bak
```

Approximate sizes were:

```text
bigip_base.conf.bak   15 KB
bigip.conf.bak        37 KB
bigip_user.conf.bak   397 bytes
```

The immediate cause of the BIG-IP failure was therefore:

> The active base and user configuration files had been truncated to zero bytes, preventing MCPD from loading the BIG-IP configuration.

The most likely contributing event was an interruption during GNS3 instability. The exact process that truncated the files was not conclusively proven.

---

## Related Platform Issue

At the same time, the GNS3 switching environment exhibited inconsistent forwarding behavior:

- Ubuntu could not ping its gateway.
- The router contained stale ARP entries.
- The switch showed connected interfaces but no dynamic MAC addresses.
- The switch forwarding plane recovered after restarting the GNS3 environment.

This was treated as a separate but potentially related platform instability.

An additional external connectivity issue was found after platform recovery:

- The router-facing switchport for the external network was still assigned to VLAN 1.
- The BIG-IP external interfaces were assigned to VLAN 30.
- Moving the router-facing port into VLAN 30 restored external ARP and IP connectivity.

---

## Recovery Preparation

Before modifying the failed device, the damaged and backup files were preserved:

```bash
mkdir -p /var/tmp/f5-02-damaged-config

cp -p /config/bigip*.conf \
/var/tmp/f5-02-damaged-config/

cp -p /config/bigip*.conf.bak \
/var/tmp/f5-02-damaged-config/
```

The preserved files were verified:

```bash
ls -lh /var/tmp/f5-02-damaged-config/
```

A fresh UCS archive was also created on the healthy peer:

```bash
tmsh save sys ucs \
/var/local/ucs/ATL-F5-01-healthy.ucs
```

---

## Recovery Procedure

Only the empty active files were restored:

```bash
cp -p /config/bigip_base.conf.bak \
/config/bigip_base.conf

cp -p /config/bigip.conf.bak \
/config/bigip.conf

cp -p /config/bigip_user.conf.bak \
/config/bigip_user.conf
```

The restored sizes were verified:

```bash
ls -lh \
/config/bigip_base.conf \
/config/bigip.conf \
/config/bigip_user.conf
```

The configuration was then validated and loaded.

Following the successful load, MCPD reported:

```text
Running Phase                   running
Last Configuration Load Status  high-config-load-succeed
End Platform ID Received        true
```

The appliance transitioned through:

```text
Standby:Not All Devices Synced
```

and then reached:

```text
Standby:In Sync
```

---

## Post-Recovery Validation

### ATL-F5-01

```text
Status   ACTIVE
Sync     In Sync
MCPD     running
```

### ATL-F5-02

```text
Status   STANDBY
Sync     In Sync
MCPD     running
```

### Failover Communication

Both failover paths reported:

```text
Status Ok
```

### Configuration Sync

The device group reported:

```text
All devices in the device group are in sync
```

### Application Validation

The Apache pool and virtual server on ATL-F5-01 were available.

Internal and external gateway reachability was restored.

---

## Post-Recovery Backups

Fresh UCS archives were created:

```text
ATL-F5-01-post-recovery.ucs
ATL-F5-02-post-recovery.ucs
```

The running configuration was saved on both devices:

```bash
tmsh save sys config
```

A VMware snapshot was created after recovery.

The GNS3 Windows project and GNS3 VM project directory were archived. The transferred archive was validated using SHA-256 checksums on both Linux and Windows.

---

## Root Cause Statement

### Confirmed Immediate Cause

The active BIG-IP configuration files on ATL-F5-02 were zero bytes, preventing MCPD from loading the base and user configuration.

### Likely Contributing Cause

GNS3 instability may have interrupted or corrupted a configuration write operation.

This relationship was not conclusively proven, so it should be recorded as a likely contributing cause rather than a confirmed root cause.

### Independent Configuration Issue

The router-facing external switchport had not been assigned to VLAN 30. This caused external BIG-IP ARP failure but did not cause the MCPD configuration-load failure.

---

## Preventive Actions

- Create UCS backups after significant configuration changes.
- Save the running configuration explicitly.
- Take VMware snapshots before high-risk changes.
- Back up both the Windows and GNS3 VM project data.
- Validate archives using SHA-256 checksums.
- Shut down GNS3 nodes and the GNS3 VM cleanly.
- Confirm switchport VLAN assignments during topology builds.
- Verify MCPD state before beginning implementation work.
- Maintain an operational recovery runbook.

---

## Lessons Learned

1. A running `mcpd` process does not prove that BIG-IP configuration loading succeeded.
2. `tmsh show sys mcp-state` is the authoritative first check for configuration-load state.
3. Repeated service messages may be downstream symptoms rather than the original failure.
4. Zero-byte configuration files explain the `No user configuration is found` validation message.
5. Preserve damaged evidence before restoring files.
6. Restore only the affected files and validate before applying the configuration.
7. A healthy HA peer provides an essential recovery reference.
8. Multiple simultaneous faults must be isolated individually.
9. Interface state `up` does not prove correct VLAN forwarding.
10. Backup integrity should be verified, not assumed.

---

## Outcome

ATL-F5-02 was restored without rebuilding the appliance.

The HA pair returned to:

```text
ATL-F5-01  ACTIVE   In Sync
ATL-F5-02  STANDBY  In Sync
```

The recovery demonstrated configuration forensics, layered troubleshooting, controlled restoration, HA validation, and disaster-recovery discipline.