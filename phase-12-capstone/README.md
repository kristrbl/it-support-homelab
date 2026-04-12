# Phase 12 — Capstone Support Scenarios

## Objective

Apply all skills from Phases 1-11 to solve end-to-end IT support scenarios from first contact to ticket closure. Each scenario is documented as a complete support case — how it was reported, diagnosed, resolved, and escalated where appropriate.

---

## Scenario Overview

| Case | Title | Category | Priority | Resolution Time |
|------|-------|----------|----------|----------------|
| CS-01 | New employee onboarding — full stack | Account / Access | Normal | 28 min |
| CS-02 | Cannot log in — multiple root causes | Authentication | High | 15 min |
| CS-03 | File share disappeared for Finance | Permissions | High | 22 min |
| CS-04 | Workstation won't boot after update | OS | Critical | 45 min |
| CS-05 | Mass login failure alert — 6AM | Security | Critical | 30 min |
| CS-06 | Printer spooler crashed — department down | Print | High | 10 min |

---

## CS-01 — New Employee Onboarding

**Reported by:** Lisa Taylor (HR Manager) — osTicket ticket #0012
**Priority:** Normal | **SLA:** 8 hours

**Request:**
> "New hire Alex Rivera starts Monday as a Finance Analyst. Please set up IT access."

**Actions taken:**

```powershell
# 1. Create AD account
New-ADUser -GivenName "Alex" -Surname "Rivera" `
    -Name "Alex Rivera" -SamAccountName "arivera" `
    -UserPrincipalName "arivera@lab.local" `
    -Department "Finance" -Title "Finance Analyst" `
    -Path "OU=Finance,OU=Lab Users,DC=lab,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Welcome2024!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true -Enabled $true

# 2. Add to Finance-Staff group
Add-ADGroupMember -Identity "Finance-Staff" -Members "arivera"

# 3. Verify drive mapping policy will apply
Get-GPResultantSetOfPolicy -Computer lab-win10 -User arivera
```

**Verification:**
- Logged into `lab-win10` as `arivera` — prompted to change password ✓
- F: drive (\\lab-dc01\Finance) mapped automatically ✓
- Wazuh logged Event ID 4624 (new logon) ✓
- Confirmed with manager via ticket reply

**Ticket status:** Closed. Time to resolution: 28 minutes.

---

## CS-02 — Cannot Log In

**Reported by:** Kevin Brown (HR) — osTicket ticket #0014
**Priority:** High | **SLA:** 4 hours

**Initial report:**
> "I can't log in to my computer this morning. It just says my password is wrong but I haven't changed it."

**Troubleshooting:**

Step 1 — Check account status:
```powershell
Get-ADUser kbrown -Properties LockedOut, PasswordExpired, Enabled, BadLogonCount
```
```
LockedOut:       False
PasswordExpired: True    ← Root cause
Enabled:         True
BadLogonCount:   0
```

Step 2 — Confirm: password hit the 90-day maximum age policy.

Step 3 — Reset and force change at logon:
```powershell
Set-ADAccountPassword -Identity kbrown -Reset `
    -NewPassword (ConvertTo-SecureString "TempP@ss2024!" -AsPlainText -Force)
Set-ADUser -Identity kbrown -ChangePasswordAtLogon $true
```

Step 4 — Walked user through CTRL+ALT+DEL → Change Password at login screen.

**Resolution:** Password expired — not a lockout. Reset and user online in 15 minutes.

**Lesson noted:** Reviewed password expiry notification — users receive no warning in current config. Logged improvement request to enable password expiry notification GPO.

---

## CS-03 — Finance Share Disappeared

**Reported by:** Sarah Johnson (Finance) — osTicket ticket #0017
**Priority:** High | **SLA:** 4 hours

**Initial report:**
> "My F: drive is gone this morning. I need my files for month-end."

**Troubleshooting:**

Step 1 — Verified lab-win10 via RDP: no F: drive mapped.

Step 2 — Checked group membership:
```powershell
Get-ADGroupMember "Finance-Staff" | Select-Object Name
```
`sjohnson` not listed — she had been removed from the group.

Step 3 — Reviewed AD audit log (Event ID 4735 — Group modified):
```
Event ID: 4735
Group:    Finance-Staff
Member removed: sjohnson
Changed by: lab\tanderson (IT account)
```

**Root cause:** IT staff member accidentally removed Finance-Staff members while doing maintenance.

Step 4 — Restored membership:
```powershell
Add-ADGroupMember -Identity "Finance-Staff" -Members "sjohnson","dwilson","amartinez"
```

Step 5 — GPO drive map reapplied after `gpupdate /force` — F: drive returned.

**Time to resolution:** 22 minutes. Notified manager of accidental change. Recommend role separation for group modification rights.

---

## CS-04 — Workstation Won't Boot After Update

**Reported by:** Tom Anderson (IT) — osTicket ticket #0019
**Priority:** Critical | **SLA:** 1 hour

**Report:**
> "lab-win11 is stuck on the spinning dots after installing updates last night. Can't get into Windows."

**Troubleshooting:**

Step 1 — Accessed Proxmox console (physical access equivalent — no RDP available).

Step 2 — Interrupted normal boot → Advanced Options → Startup Settings → Safe Mode with Networking.

Step 3 — Identified recent update:
```
KB5034441 — installed 04/09/2026 11:47 PM
```

Step 4 — Uninstalled via DISM in Safe Mode:
```cmd
dism /online /remove-package /packagename:Package_for_RollupFix~amd64~~10.0.1.0
```

Step 5 — Rebooted normally — Windows loaded successfully.

Step 6 — Restored snapshot (`lab-win11-pre-phase6-gpo`) as baseline, re-applied updates through tested batch.

**Time to resolution:** 45 minutes (Proxmox console access critical to resolution).

---

## CS-05 — Mass Login Failure Alert

**Reported by:** Wazuh SIEM — auto-alert 06:04 AM
**Priority:** Critical | **SLA:** 1 hour

**Alert:**
```
Rule ID: 100001 (custom)
Level:   12
Description: Possible brute force: 5+ failed logons in 60 seconds
Agent:   lab-dc01
Account: administrator
Source:  10.10.20.107 (lab-win11)
Time:    06:04:37 AM
```

**Investigation:**

Step 1 — Reviewed Wazuh timeline: 47 Event ID 4625 events in 8 minutes against `administrator` account.

Step 2 — RDP to lab-dc01 — `administrator` not locked (built-in account — lockout policy doesn't apply by default).

Step 3 — Reviewed lab-win11 at Proxmox console — no active session. Checked running processes:

```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```
Found: `pwsh.exe` running as SYSTEM with unusual CPU spike.

Step 4 — Identified scheduled task running a test script left over from Phase 8 PowerShell remoting exercise. Script was looping login attempts.

Step 5 — Killed process, deleted scheduled task, confirmed alerts stopped.

```
Incident verdict: Lab artifact — no real threat.
Remediation: Document test scripts, clean up after exercises.
```

**Time to resolution:** 30 minutes.

---

## CS-06 — Print Spooler Crash

**Reported by:** Rachel White (IT Help Desk) — osTicket ticket #0021
**Priority:** High | **SLA:** 4 hours

**Report:**
> "No one in the Finance department can print. Printer shows 'offline' even though it's on."

**Troubleshooting:**

Step 1 — Remote into lab-dc01 (where print server would run):

```powershell
Get-Service Spooler
# Status: Stopped
```

Step 2 — Restart service:

```powershell
Restart-Service Spooler
Get-Service Spooler
# Status: Running
```

Step 3 — Confirmed printer queue cleared and Finance users able to print.

Step 4 — Reviewed System log for Spooler crash:

```
Event ID: 7034 — Print Spooler service terminated unexpectedly (1 time)
Event ID: 1000 — Faulting module: ntdll.dll (corrupt print driver)
```

Step 5 — Removed and re-added the affected print driver:

```powershell
Remove-PrinterDriver -Name "HP Universal Printing PCL 6" -ComputerName lab-dc01
Add-PrinterDriver -Name "HP Universal Printing PCL 6" -ComputerName lab-dc01
```

**Time to resolution:** 10 minutes.

---

## Capstone Summary

| Skill Area | Phases Applied | Demonstrated In |
|-----------|---------------|-----------------|
| AD Account Management | 4, 5 | CS-01, CS-02, CS-03 |
| Group Policy | 6 | CS-01, CS-03 |
| Remote Support | 8 | CS-02, CS-03, CS-06 |
| Log Analysis / SIEM | 11 | CS-05 |
| OS Troubleshooting | 3 | CS-04 |
| Network Diagnostics | 7 | CS-03 |
| Help Desk Ticketing | 9 | All cases |
| Hypervisor Access | 2 | CS-04 |

**All 6 capstone scenarios resolved within SLA. 100% ticket closure rate.**

---

## Skills Demonstrated

- End-to-end IT support methodology: triage → diagnose → resolve → document → close
- Cross-platform investigation (Wazuh + Event Viewer + ADUC + Proxmox console)
- Root cause analysis vs. symptom treatment
- Escalation judgment — when to escalate, when to resolve independently
- Post-incident documentation and improvement recommendations
- Full IT support lifecycle across all 12 lab phases
