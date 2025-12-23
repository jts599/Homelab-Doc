# Proxmox Storage Configuration – Dell PowerEdge R730xd

## Hardware
- Server: Dell PowerEdge R730xd
- RAID Controller: PERC H730 Mini (Battery-backed cache)
- Disks:
  - 2× SATA SSD (OS)
  - 12× HDD (Media storage)

---

## Virtual Disk Configuration

### 1. OS / Proxmox Boot Disk
- Disks: 2× SATA SSD
- RAID Level: RAID 1
- Capacity: ~446 GB
- Stripe Size: 64 KB
- Read Policy: No Read Ahead
- Write Policy: Write Back
- Disk Cache Policy: Default
- T10 PI Capability: Disabled

**Purpose**
- Proxmox VE OS
- VM / CT system disks
- LVM-thin storage

**Notes**
- Write Back enabled with healthy PERC battery
- Designed for reliability and low-latency random I/O

---

### 2. Media / Bulk Storage Disk
- Disks: 12× HDD
- RAID Level: RAID 6
- Stripe Size: 256 KB
- Read Policy: Read Ahead
- Write Policy: Write Back
- Disk Cache Policy: Default
- T10 PI Capability: Disabled

**Purpose**
- Media storage
- ISO images
- Backups
- Large sequential files

**Notes**
- Optimized for capacity and fault tolerance
- Can tolerate up to 2 disk failures
- Sequential read performance prioritized over IOPS

---

## Proxmox Storage Layout

- OS Disk:
  - Filesystem: ext4
  - Storage Type: LVM-thin
  - Use: VM and container disks

- Media Disk:
  - Filesystem: XFS
  - Storage Type: Directory
  - Use: Media files, backups, bind mounts

---

## Design Rationale
- Hardware RAID chosen to avoid additional HBA hardware
- ZFS intentionally not used due to PERC H730 limitations
- Setup favors simplicity, reliability, and easy recovery for a homelab
