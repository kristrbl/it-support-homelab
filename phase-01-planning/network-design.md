# Network Design

## Topology

```mermaid
graph TD
    HOST["🖥️ Proxmox Host — gw-nexus\n192.168.1.50"]

    HOST --> vmbr0["vmbr0\nMain LAN\n192.168.1.0/24"]
    HOST --> vmbr1["vmbr1\nMonitoring Network\n10.10.10.0/24"]
    HOST --> vmbr2["vmbr2\nAD Lab Network\n10.10.20.0/24"]

    vmbr2 --> PROTO["🗄️ gw-protocol\nDomain Controller\nDNS · DHCP\n10.10.20.10"]
    vmbr2 --> OP1["💻 gw-operative-1\nWindows 11\n10.10.20.20 (DHCP)"]
    vmbr2 --> OP2["💻 gw-operative-2\nWindows 10\n10.10.20.21 (DHCP)"]
    vmbr2 --> TRACE["🔬 gw-tracer\nNetwork Lab\n10.10.20.30"]
    vmbr2 --> EP["🐧 gw-endpoint\nUbuntu Server\n10.10.20.40"]

    vmbr1 --> DISP["🎫 gw-dispatch\nGLPI\n10.10.10.10"]
    vmbr1 --> PAN["🔍 gw-panoptic\nWazuh SIEM\n10.10.10.20"]

    style HOST fill:#2d2d2d,color:#fff
    style vmbr0 fill:#555,color:#fff
    style vmbr1 fill:#1a4a6b,color:#fff
    style vmbr2 fill:#1a6b2d,color:#fff
    style PROTO fill:#1a6b2d,color:#fff
    style OP1 fill:#1a4a6b,color:#fff
    style OP2 fill:#1a4a6b,color:#fff
    style TRACE fill:#555,color:#fff
    style EP fill:#555,color:#fff
    style DISP fill:#6b4a1a,color:#fff
    style PAN fill:#4a1a6b,color:#fff
```

---

![Proxmox network bridges](screenshots/proxmox-network-bridges.png)
*Proxmox network configuration — vmbr0, vmbr1, vmbr2 bridges*

---

## Network Segments

| Bridge | Subnet | Purpose |
|--------|--------|---------|
| vmbr0 | 192.168.1.0/24 | Main LAN — physical host management |
| vmbr1 | 10.10.10.0/24 | Monitoring — SIEM and ticketing |
| vmbr2 | 10.10.20.0/24 | AD lab — domain controller and workstations |

---

## IP Address Plan

| VM | Bridge | IP | Assignment |
|----|--------|----|-----------|
| gw-protocol | vmbr2 | 10.10.20.10 | Static |
| gw-operative-1 | vmbr2 | 10.10.20.20 | DHCP reservation |
| gw-operative-2 | vmbr2 | 10.10.20.21 | DHCP reservation |
| gw-tracer | vmbr2 | 10.10.20.30 | Static |
| gw-endpoint | vmbr2 | 10.10.20.40 | Static |
| gw-dispatch | vmbr1 | 10.10.10.10 | Static |
| gw-panoptic | vmbr1 | 10.10.10.20 | Static |
| DHCP pool | vmbr2 | 10.10.20.100–200 | Dynamic |

---

## DHCP Configuration (gw-protocol)

| Setting | Value |
|---------|-------|
| Scope | Lab-Workstations |
| Network | 10.10.20.0/24 |
| Range | 10.10.20.100 – 10.10.20.200 |
| Subnet mask | 255.255.255.0 |
| Default gateway | 10.10.20.1 |
| DNS server | 10.10.20.10 |
| Lease time | 8 hours |

![DHCP scope on gw-protocol](screenshots/dhcp-scope-config.png)
*DHCP scope configuration on gw-protocol*

---

## DNS Configuration (gw-protocol)

| Zone | Type | Purpose |
|------|------|---------|
| lab.local | Forward lookup | Hostname to IP resolution |
| 20.10.10.in-addr.arpa | Reverse lookup | IP to hostname resolution |

![DNS Manager showing lab.local](screenshots/dns-manager-lab-local.png)
*DNS Manager — lab.local forward lookup zone*
