# Windows Server 2022 Install Guide — lab-dc01

Step-by-step for installing Windows Server 2022 Evaluation on VM 115 via Proxmox VNC console.

## Before You Start

- VM 115 (lab-dc01) is built and ready in Proxmox
- Open Proxmox web UI → `https://192.168.1.50:8006`
- Navigate to: `Datacenter → gw-nexus → 115 (lab-dc01)`
- Click **Console** (top right) to open VNC

---

## Step 1 — Start the VM

Click **Start** in the Proxmox UI, then immediately click into the VNC console.

When you see **"Press any key to boot from CD or DVD..."** — press a key fast. You have about 5 seconds.

> If you miss it, the VM will try to PXE boot and fail. Just restart and try again.

---

## Step 2 — Windows Setup Screen

| Field | Select |
|-------|--------|
| Language | English (United States) |
| Time/currency | English (United States) |
| Keyboard | US |

Click **Next → Install Now**.

---

## Step 3 — Select Edition

Choose: **Windows Server 2022 Standard Evaluation (Desktop Experience)**

> Desktop Experience = full GUI. The non-Desktop Experience option is Server Core (command line only). You want the GUI for this lab.

Accept the license terms → Next.

---

## Step 4 — Custom Install

Select **Custom: Install Windows only (advanced)**.

> Do NOT select Upgrade — there's nothing to upgrade from.

---

## Step 5 — Load VirtIO Storage Driver (Critical Step)

The disk list will be **empty** — Windows can't see the VirtIO disk without a driver.

1. Click **Load driver**
2. Click **Browse**
3. Navigate to: `CD Drive (E:) virtio-win → amd64 → w11`
4. Select the driver that appears (`Red Hat VirtIO SCSI controller`)
5. Click **Next**

The 64 GB disk will now appear. Select it → **Next**.

> **Why w11?** Windows Server 2022 shares the same kernel as Windows 11. The w11 VirtIO driver works correctly for Server 2022.

---

## Step 6 — Install

Windows copies files and reboots automatically (2–3 times). Takes about 10–15 minutes.

> When it reboots, it will boot from the disk automatically — you don't need to press any key this time.

---

## Step 7 — Set Administrator Password

After the final reboot, Windows prompts for an Administrator password.

Set a strong password and remember it — this is your domain admin account later.

> Example: `Lab@dmin2024!` — something with upper, lower, number, symbol.

---

## Step 8 — Post-Install Setup (Inside the VM)

After logging in, open **PowerShell as Administrator** and run these commands:

### Rename the computer
```powershell
Rename-Computer -NewName "lab-dc01" -Restart
```

### After reboot — set a static IP
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.10 -PrefixLength 24 -DefaultGateway 10.10.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
```

### Enable RDP
```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

### Install VirtIO guest agent (from virtio CD)
Open File Explorer → CD Drive (E:) → run `virtio-win-guest-tools.exe`

This installs the guest agent so Proxmox can see the VM's IP address and do graceful shutdowns.

---

## Step 9 — Snapshot

Once the VM is set up and stable, take a Proxmox snapshot before the next phase:

In Proxmox UI: `115 (lab-dc01) → Snapshots → Take Snapshot`

Name it: `baseline-pre-ad`

> Snapshots let you roll back if AD promotion goes wrong in Phase 4.

---

## What's Next

Phase 4 will promote lab-dc01 to a Domain Controller with Active Directory Domain Services (AD DS), DNS, and DHCP.
