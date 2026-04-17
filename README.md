# IT Support Homelab

A hands-on homelab built on a bare-metal Proxmox hypervisor to develop and document real IT support, systems administration, and security monitoring skills.

Each phase is fully documented with configuration steps, screenshots, and troubleshooting notes — built to mirror what entry-level IT roles encounter in production environments.

---

## Lab Environment

| Component | Details |
|-----------|---------|
| Hypervisor | Proxmox VE 8.x — bare metal, 32 GB RAM, 800 GB storage |
| Domain Controller | lab-dc01 — Windows Server 2022, Active Directory, DNS, DHCP |
| Workstations | lab-win10 (Windows 10), lab-win11 (Windows 11) |
| Help Desk | lab-osticket — osTicket ticketing system |
| SIEM | Wazuh — centralized log monitoring and alerting |
| Network Lab | Isolated virtual network segments (vmbr1, vmbr2) |
| Linux Endpoint | Ubuntu Server — admin target, Wazuh agent, hardening |
| Domain | lab.local |

---

## Projects

| Phase | Project | Skills |
|-------|---------|--------|
| 1 | [Lab Planning and Architecture](phase-01-planning/) | Network design, infrastructure documentation, VM planning |
| 2 | [Proxmox and VM Setup](phase-02-proxmox-setup/) | Hypervisor management, VM lifecycle, virtual networking |
| 3 | Windows Workstation Support | OS troubleshooting, local accounts, Windows Update |
| 4 | Active Directory | AD DS promotion, DNS, DHCP, domain configuration |
| 5 | User and Group Administration | Users, OUs, security groups, NTFS permissions |
| 6 | Group Policy | GPO creation, workstation enforcement, baseline hardening |
| 7 | Network Troubleshooting | TCP/IP, DNS/DHCP diagnostics, Wireshark, nmap |
| 8 | Remote Support | RDP, remote administration tools and procedures |
| 9 | Help Desk Ticketing | osTicket setup, ticket workflows, SLA management |
| 10 | Software Deployment | MSI deployment, application troubleshooting |
| 11 | Log Analysis and SIEM | Wazuh agents, Event Viewer, alert investigation |
| 12 | Capstone Support Scenarios | End-to-end IT support methodology and documentation |

*Phases added as completed.*

---

## Network Design

```
Proxmox Host — gw-nexus (192.168.1.50)
│
├── vmbr1 — 10.10.10.0/24 (Monitoring)
│   ├── Wazuh SIEM         10.10.10.10
│   └── osTicket           10.10.10.30
│
└── vmbr2 — 10.10.20.0/24 (AD Lab)
    ├── lab-dc01 (DC)      10.10.20.10
    ├── lab-win10          DHCP
    └── lab-win11          DHCP
```

Both segments are isolated from the home network. Lab VMs have no direct internet access.

---

## Tools and Technologies

`Proxmox VE` `Windows Server 2022` `Active Directory` `DNS` `DHCP` `Group Policy`
`Windows 10/11` `Ubuntu Server` `Wazuh` `osTicket` `Wireshark` `nmap` `RDP` `PowerShell` `Bash`

---

## Contributors

- [@kristrbl](https://github.com/kristrbl)
