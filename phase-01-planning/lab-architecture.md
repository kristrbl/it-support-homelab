# Lab Architecture — IT Support Homelab

## Overview

This lab simulates a small business IT environment. It is built on a Proxmox hypervisor running on dedicated home server hardware. All VMs are isolated on a dedicated internal network bridge separate from the main LAN.

---

## Hypervisor

| Item | Detail |
|------|--------|
| Platform | Proxmox VE 8.x |
| Host | Dedicated home server |
| CPU | x86_64, virtualization extensions enabled |
| RAM | 32 GB |
| Storage | SSD pool for VM disks |

---

## VM Inventory

| VM Name | Role | OS | vCPU | RAM | Disk | IP Address |
|---------|------|----|------|-----|------|------------|
| lab-dc01 | Domain Controller / DNS / DHCP | Windows Server 2022 Eval | 2 | 4 GB | 60 GB | 10.10.10.10 |
| lab-win10 | Domain workstation / support target | Windows 10 Pro 22H2 | 2 | 4 GB | 60 GB | 10.10.10.20 (DHCP) |
| lab-osticket | Help desk ticketing system | Ubuntu Server 22.04 LTS | 2 | 2 GB | 30 GB | 10.10.10.30 |
| lab-wazuh | SIEM / log monitoring (Phase 11) | Ubuntu Server 22.04 LTS | 4 | 8 GB | 50 GB | 10.10.10.40 |

**Total estimated resources:** 10 vCPU / 18 GB RAM / 200 GB storage

---

## Domain Details

| Setting | Value |
|---------|-------|
| Domain name | lab.local |
| Domain Controller | lab-dc01 (10.10.10.10) |
| DNS Server | 10.10.10.10 |
| DHCP Range | 10.10.10.100 – 10.10.10.200 |
| Default Gateway | N/A (isolated network, no internet routing) |

---

## Design Decisions

- **Isolated network bridge (vmbr1):** Keeps lab DHCP and DNS experiments from affecting the main LAN. Lab VMs cannot reach the internet directly.
- **Windows Server 2022 Evaluation:** Free 180-day license from Microsoft Evaluation Center. Sufficient for all lab phases.
- **osTicket:** Free, open source ITSM platform. Mirrors ticketing tools used in real help desk environments (ServiceNow, Zendesk, Freshdesk).
- **Wazuh:** Free SIEM platform. Provides real log aggregation and alerting experience relevant to both IT Support and SOC roles.

---

## Phases Overview

| Phase | Focus Area |
|-------|-----------|
| 1 | Lab planning and architecture (this document) |
| 2 | Proxmox hypervisor validation and lab network setup |
| 3 | Windows 10 workstation support |
| 4 | Windows Server 2022 and Active Directory |
| 5 | Domain join, users, groups, and permissions |
| 6 | Group Policy and workstation management |
| 7 | Network troubleshooting |
| 8 | Remote administration and support tools |
| 9 | osTicket help desk setup |
| 10 | Software deployment and troubleshooting |
| 11 | Log analysis, Event Viewer, and Wazuh SIEM |
| 12 | Capstone support scenarios |
| 13 | GitHub publishing and resume refinement |
