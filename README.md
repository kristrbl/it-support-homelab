# IT Support Specialist Homelab

A hands-on homelab built to develop real IT support skills and create a portfolio of documented lab projects for entry-level IT Support and Help Desk roles.

## About This Lab

This lab simulates a small business IT environment running on a Proxmox hypervisor with 15 virtual machines across 4 network zones. Each project is fully documented with setup steps, real-world support scenarios, troubleshooting steps, and outcomes.

## Lab Environment

| Component | Details |
|-----------|---------|
| Hypervisor | Proxmox VE 8.x — gw-nexus (192.168.1.50) |
| Domain Controller | gw-protocol — Active Directory, DNS, DHCP |
| Workstations | gw-operative-1 (Win11), gw-operative-2 (Win10) |
| Ticketing System | gw-dispatch — GLPI help desk |
| SIEM | gw-panoptic — Wazuh log monitoring |
| Network Lab | gw-tracer — packet analysis and diagnostics |
| Linux Endpoint | gw-endpoint — Ubuntu Server |
| Domain | lab.local |
| Lab Network | 10.10.20.0/24 (AD/workstations), 10.10.10.0/24 (monitoring) |

## Projects

| Phase | Project | Skills Demonstrated |
|-------|---------|-------------------|
| 1 | [Lab Planning and Architecture](phase-01-planning/) | Documentation, network design, infrastructure planning |
| 2 | Virtualization Setup | Proxmox, VM lifecycle, snapshots, networking |
| 3 | Windows Workstation Support | Windows OS, end-user support, local accounts |
| 4 | Active Directory Lab | AD DS, Windows Server, DNS, DHCP |
| 5 | User and Group Administration | User accounts, OUs, NTFS permissions |
| 6 | Group Policy Lab | GPO, endpoint configuration, policy enforcement |
| 7 | Network Troubleshooting | TCP/IP, DNS/DHCP diagnostics, Wireshark |
| 8 | Remote Support Lab | RDP, remote administration tools |
| 9 | GLPI Help Desk | ITSM, ticketing workflows, SLA management |
| 10 | Software Deployment | MSI deployment, application troubleshooting |
| 11 | Log Analysis and Wazuh SIEM | Event Viewer, log investigation, security monitoring |
| 12 | Capstone Support Scenarios | End-to-end IT support methodology |

## Lab Environment Details

| Component | Details |
|-----------|---------|
| Hypervisor | [Proxmox VE 8.x](https://www.proxmox.com/en/proxmox-virtual-environment/overview) |
| Domain Controller | gw-protocol — [Active Directory Domain Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) |
| Ticketing System | gw-dispatch — [GLPI](https://glpi-project.org/) |
| SIEM | gw-panoptic — [Wazuh](https://wazuh.com/) |
| Packet Analysis | gw-tracer — [Wireshark](https://www.wireshark.org/) |

## Status

In progress — phases added as completed.

---
*Built for IT Support Specialist and Help Desk job readiness.*
