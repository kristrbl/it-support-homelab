# Phase 2 — Proxmox Validation

Verified gw-nexus (Proxmox VE) has the resources and configuration needed to host the IT support lab environment.

## Host Specs Confirmed

| Resource | Total | Used | Available |
|----------|-------|------|-----------|
| RAM | 31 GB | 19 GB | ~11 GB |
| Storage (local-lvm) | ~813 GB | ~118 GB | ~677 GB |
| Storage (local/ISOs) | ~94 GB | ~62 GB | ~28 GB |

## Lab Network Bridge

| Setting | Value |
|---------|-------|
| Bridge | vmbr1 |
| Subnet | 10.10.10.0/24 |
| Gateway | 10.10.10.1 (Proxmox host) |
| Type | Internal only (no bridge-ports) |
| IP Forwarding | Enabled |

The bridge is fully isolated from the home network — lab VMs cannot reach the internet directly.

## ISOs Staged

| ISO | Filename | Size |
|-----|----------|------|
| Windows Server 2022 Eval | `26100.32230...SERVER_EVAL_x64FRE_en-us.iso` | 7.6 GB |
| Windows 10 22H2 | `Win10_22H2_en-us_x64.iso` | 5.7 GB (downloading) |
| Ubuntu 24.04 LTS Server | `ubuntu-24.04.4-live-server-amd64.iso` | 3.2 GB |
| VirtIO Drivers | `virtio-win.iso` | 754 MB |

## VM Assignments

| VM | ID | Role | RAM | Disk | IP |
|----|----|------|-----|------|----|
| lab-dc01 | 115 | Domain Controller (WS2022) | 4 GB | 64 GB | 10.10.10.10 |
| lab-win10 | 116 | Windows 10 Workstation | 4 GB | 60 GB | 10.10.10.20 |
| lab-osticket | 117 | osTicket / Help Desk (Ubuntu 24.04) | 2 GB | 40 GB | 10.10.10.30 |

## lab-dc01 VM Configuration

Created via Proxmox CLI (`qm create`).

| Setting | Value |
|---------|-------|
| Machine | q35 |
| BIOS | OVMF (UEFI) |
| CPU | host (2 cores) |
| RAM | 4096 MB |
| Disk | 64 GB, local-lvm, SSD + discard |
| SCSI Controller | virtio-scsi-pci |
| Network | virtio, vmbr1 |
| Boot order | IDE0 (ISO) → SCSI0 (disk) |
| IDE0 | Windows Server 2022 Eval ISO |
| IDE2 | VirtIO drivers ISO |
