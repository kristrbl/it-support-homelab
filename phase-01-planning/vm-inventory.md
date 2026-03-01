# VM Inventory — IT Support Homelab

## Active VMs

### lab-dc01
- **Role:** Domain Controller, DNS Server, DHCP Server
- **OS:** Windows Server 2022 Standard Evaluation
- **vCPU:** 2
- **RAM:** 4 GB
- **Disk:** 60 GB (thin provisioned)
- **IP:** 10.10.10.10 (static)
- **Network:** vmbr1 (isolated lab bridge)
- **Phase:** Deployed in Phase 4

### lab-win10
- **Role:** Domain workstation, end-user support simulation target
- **OS:** Windows 10 Pro 22H2
- **vCPU:** 2
- **RAM:** 4 GB
- **Disk:** 60 GB (thin provisioned)
- **IP:** 10.10.10.20 (DHCP reservation)
- **Network:** vmbr1 (isolated lab bridge)
- **Phase:** Deployed in Phase 3

### lab-osticket
- **Role:** Help desk ticketing system (osTicket)
- **OS:** Ubuntu Server 22.04 LTS
- **vCPU:** 2
- **RAM:** 2 GB
- **Disk:** 30 GB (thin provisioned)
- **IP:** 10.10.10.30 (static)
- **Network:** vmbr1 (isolated lab bridge)
- **Phase:** Deployed in Phase 9

### lab-wazuh
- **Role:** SIEM and log monitoring (Wazuh)
- **OS:** Ubuntu Server 22.04 LTS
- **vCPU:** 4
- **RAM:** 8 GB
- **Disk:** 50 GB (thin provisioned)
- **IP:** 10.10.10.40 (static)
- **Network:** vmbr1 (isolated lab bridge)
- **Phase:** Deployed in Phase 11

---

## Snapshot Policy

All VMs use Proxmox snapshots before any major configuration change. Naming convention:

```
VMNAME-pre-PHASE-DESCRIPTION
Example: lab-dc01-pre-phase4-ad-install
```

This allows rollback without redeploying from scratch.

---

## Resource Summary

| VM | vCPU | RAM | Disk |
|----|------|-----|------|
| lab-dc01 | 2 | 4 GB | 60 GB |
| lab-win10 | 2 | 4 GB | 60 GB |
| lab-osticket | 2 | 2 GB | 30 GB |
| lab-wazuh | 4 | 8 GB | 50 GB |
| **Total** | **10** | **18 GB** | **200 GB** |
