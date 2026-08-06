# Backup and Disaster Recovery Guide

**Project:** Principal Consultant Playbook

**Lab:** F5 BIG-IP High Availability Lab

**Version:** 1.0

**Status:** Production Ready

---

# Purpose

This document describes the backup strategy, disaster recovery process, and validation procedures for the F5 BIG-IP lab environment.

The objective is to ensure that every layer of the environment can be restored independently in the event of hardware failure, virtual-machine corruption, configuration loss, or accidental deletion.

The recovery strategy intentionally includes multiple backup layers rather than relying on a single backup mechanism.

---

# Recovery Layers

The lab is protected by six independent recovery mechanisms.

| Layer | Protects | Recovery Time |
|---------|-----------|---------------|
| VMware Snapshot | Entire virtual machine state | Minutes |
| BIG-IP UCS | BIG-IP configuration | Minutes |
| Saved Configuration Files | Running configuration | Minutes |
| Windows GNS3 Project | Local project metadata | Minutes |
| GNS3 VM Project Archive | Virtual disks and node data | Minutes |
| Git Repository | Documentation and diagrams | Immediate |

---

# Backup Storage

## Windows

```text
D:\Engineering\Backups\F5\
```

Contents include:

```text
F5-GNS3VM-Project-YYYY-MM-DD.tar.gz

Windows-Project\
```

---

## BIG-IP

UCS archives are stored on each appliance.

Example:

```text
/var/local/ucs/
```

Examples:

```text
ATL-F5-01-post-recovery.ucs

ATL-F5-02-post-recovery.ucs
```

---

## Git Repository

The repository stores:

- Documentation
- Draw.io diagrams
- Markdown files
- Architecture documents
- Operational runbooks

The repository intentionally excludes licensed software and virtual-machine images.

---

# VMware Protection

The VMware snapshot provides the fastest recovery mechanism.

Recommended snapshot naming:

```
YYYY-MM-DD_PreChange
```

Example:

```
2026-08-05_PostRecovery
```

Snapshots should only be retained until the corresponding UCS and project backups have been validated.

Snapshots are not considered permanent backups.

---

# BIG-IP UCS Backup

Create a UCS archive.

```bash
tmsh save sys ucs /var/local/ucs/<filename>.ucs
```

Example:

```bash
tmsh save sys ucs /var/local/ucs/ATL-F5-01-post-recovery.ucs
```

Verify:

```bash
ls -lh /var/local/ucs
```

Expected result:

- UCS file exists
- Non-zero size
- Current timestamp

---

# Save Running Configuration

Always save the active configuration after major changes.

```bash
tmsh save sys config
```

Verify that:

- bigip.conf updated
- bigip_base.conf updated
- bigip_user.conf updated

---

# Windows GNS3 Project Backup

The Windows project contains:

- Project definition
- Local captures
- Metadata
- Local node information

Example location:

```text
C:\Users\<username>\GNS3\projects\F5\
```

Backup destination:

```text
D:\Engineering\Backups\F5\Windows-Project\
```

Example:

```powershell
robocopy "C:\Users\theta\GNS3\projects\F5" `
"D:\Engineering\Backups\F5\Windows-Project" `
/E /COPY:DAT /DCOPY:T
```

Verify:

- F5.gns3 exists
- project-files directory exists

---

# GNS3 VM Project Backup

Because VMware nodes cannot be exported directly from the GNS3 GUI, the project directory is archived directly from the GNS3 VM.

Determine the project UUID.

Example:

```powershell
(Get-Content F5.gns3 -Raw | ConvertFrom-Json).project_id
```

Archive:

```bash
cd /opt/gns3/projects

sudo tar -czf \
/home/gns3/F5-GNS3VM-Project-YYYY-MM-DD.tar.gz \
<ProjectUUID>
```

Verify archive:

```bash
gzip -t \
/home/gns3/F5-GNS3VM-Project-YYYY-MM-DD.tar.gz
```

Expected result:

```
echo $?
0
```

---

# Copy Archive to Windows

Example:

```powershell
scp gns3@<GNS3VM-IP>:/home/gns3/F5-GNS3VM-Project-YYYY-MM-DD.tar.gz `
"D:\Engineering\Backups\F5\"
```

---

# SHA-256 Validation

Generate checksum on Linux.

```bash
sha256sum \
/home/gns3/F5-GNS3VM-Project-YYYY-MM-DD.tar.gz
```

Generate checksum on Windows.

```powershell
Get-FileHash `
"D:\Engineering\Backups\F5\F5-GNS3VM-Project-YYYY-MM-DD.tar.gz" `
-Algorithm SHA256
```

The hashes must match exactly.

Matching hashes verify that the archive transferred without corruption.

---

# Recovery Order

If complete recovery is required:

1. Restore VMware snapshot (if appropriate).
2. Restore the GNS3 VM project archive.
3. Restore the Windows project metadata.
4. Boot BIG-IP appliances.
5. Restore UCS archives if necessary.
6. Save configuration.
7. Verify HA synchronization.
8. Verify application traffic.
9. Verify backups.

---

# Validation Checklist

## Infrastructure

- [ ] Cisco router operational
- [ ] Cisco switch operational
- [ ] Ubuntu Apache reachable
- [ ] BIG-IP 01 online
- [ ] BIG-IP 02 online

---

## High Availability

- [ ] Device Trust established
- [ ] ConfigSync operational
- [ ] Failover status healthy
- [ ] Devices In Sync

---

## Application

- [ ] Pool healthy
- [ ] Monitor green
- [ ] Virtual server available
- [ ] HTTP application reachable

---

## Backup Validation

- [ ] VMware snapshot created
- [ ] UCS backup verified
- [ ] Running configuration saved
- [ ] Windows project copied
- [ ] GNS3 VM archive created
- [ ] SHA-256 hashes verified
- [ ] Backup stored in permanent location

---

# Lessons Learned

The recovery performed on 2026-08-05 demonstrated the importance of maintaining multiple independent recovery layers.

A damaged BIG-IP configuration was successfully restored using backup configuration files and resynchronized within the HA pair.

The complete GNS3 environment was archived independently of VMware, allowing the lab topology and node data to be restored even if the GNS3 VM itself becomes unavailable.

Future lab milestones will follow the same backup and validation process before major architectural changes.