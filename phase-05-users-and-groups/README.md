# Phase 5 — User and Group Administration

## Objective

Create and manage Active Directory users, security groups, and NTFS permissions. Practice the day-to-day account administration tasks common in IT support and helpdesk roles — new hires, leavers, role changes, and access control.

---

## Tasks Completed

- [x] Bulk user creation via PowerShell (CSV import)
- [x] Security groups created per department
- [x] Users placed in correct OUs and assigned to groups
- [x] NTFS permissions applied to shared folders
- [x] Password policy reviewed and Fine-Grained Password Policy configured
- [x] Account unlock and password reset procedures documented
- [x] Stale account audit — identify and disable inactive accounts
- [x] Group membership report generated

---

## User Creation — Bulk Import

Created 10 lab users across three departments using a CSV file to simulate an onboarding batch.

### users.csv

```csv
FirstName,LastName,Department,OU,Title
John,Smith,IT,IT,Systems Administrator
Maria,Chen,IT,IT,Help Desk Technician
David,Wilson,Finance,Finance,Financial Analyst
Sarah,Johnson,Finance,Finance,Accounts Payable
Kevin,Brown,HR,HR,HR Generalist
Lisa,Taylor,HR,HR,HR Manager
Tom,Anderson,IT,IT,Network Administrator
Amy,Martinez,Finance,Finance,Controller
James,Lee,HR,HR,Recruiter
Rachel,White,IT,IT,Help Desk Technician
```

### Bulk Creation Script

```powershell
$users = Import-Csv -Path "C:\Scripts\users.csv"

foreach ($user in $users) {
    $username   = ($user.FirstName[0] + $user.LastName).ToLower()
    $upn        = "$username@lab.local"
    $ouPath     = "OU=$($user.OU),OU=Lab Users,DC=lab,DC=local"
    $password   = ConvertTo-SecureString "Welcome2024!" -AsPlainText -Force

    New-ADUser `
        -GivenName        $user.FirstName `
        -Surname          $user.LastName `
        -Name             "$($user.FirstName) $($user.LastName)" `
        -SamAccountName   $username `
        -UserPrincipalName $upn `
        -Department       $user.Department `
        -Title            $user.Title `
        -Path             $ouPath `
        -AccountPassword  $password `
        -ChangePasswordAtLogon $true `
        -Enabled          $true

    Write-Host "Created: $username in $ouPath"
}
```

![PowerShell output showing 10 users created successfully](screenshots/bulk-user-creation.png)
*Bulk import — 10 users created across IT, Finance, and HR OUs*

---

## Security Groups

Created departmental security groups for resource access control:

```powershell
$groups = @("IT-Staff","Finance-Staff","HR-Staff","IT-Admins","File-Share-IT","File-Share-Finance","File-Share-HR")

foreach ($group in $groups) {
    New-ADGroup -Name $group -GroupScope Global -GroupCategory Security `
                -Path "OU=Security Groups,OU=Lab Groups,DC=lab,DC=local"
}
```

### Group Membership Assignment

```powershell
# Add IT users to IT-Staff group
Add-ADGroupMember -Identity "IT-Staff" -Members "jsmith","mchen","tanderson","rwhite"

# Add Finance users to Finance-Staff
Add-ADGroupMember -Identity "Finance-Staff" -Members "dwilson","sjohnson","amartinez"

# Add HR users to HR-Staff
Add-ADGroupMember -Identity "HR-Staff" -Members "kbrown","ltaylor","jlee"
```

![ADUC showing IT-Staff group members tab](screenshots/group-members-it-staff.png)
*ADUC — IT-Staff group members populated from bulk script*

---

## NTFS Permissions — Shared Folders

Created department file shares on `lab-dc01` with NTFS permissions scoped to security groups.

### Folder Structure

```
C:\Shares\
├── IT\
├── Finance\
├── HR\
└── Common\
```

### Permissions Script

```powershell
$shares = @{
    "IT"      = "File-Share-IT"
    "Finance" = "File-Share-Finance"
    "HR"      = "File-Share-HR"
}

foreach ($share in $shares.GetEnumerator()) {
    $path = "C:\Shares\$($share.Key)"
    New-Item -ItemType Directory -Path $path -Force

    # Remove inheritance, apply explicit ACL
    $acl = Get-Acl $path
    $acl.SetAccessRuleProtection($true, $false)

    $rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
        "LAB\$($share.Value)", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow"
    )
    $acl.AddAccessRule($rule)
    Set-Acl -Path $path -AclObject $acl

    # Create SMB share
    New-SmbShare -Name $share.Key -Path $path -FullAccess "LAB\IT-Admins" -ChangeAccess "LAB\$($share.Value)"
}
```

![File Explorer — Security tab for Finance share showing Finance-Staff permissions](screenshots/ntfs-finance-share.png)
*NTFS permissions — Finance share, Modify access scoped to Finance-Staff group only*

---

## Fine-Grained Password Policy

Created a stricter password policy for the `IT-Admins` group:

```powershell
New-ADFineGrainedPasswordPolicy `
    -Name "IT-Admin-PSO" `
    -Precedence 10 `
    -MinPasswordLength 14 `
    -PasswordHistoryCount 24 `
    -MaxPasswordAge (New-TimeSpan -Days 60) `
    -MinPasswordAge (New-TimeSpan -Days 1) `
    -LockoutThreshold 5 `
    -LockoutDuration (New-TimeSpan -Minutes 30) `
    -LockoutObservationWindow (New-TimeSpan -Minutes 30) `
    -ComplexityEnabled $true `
    -ProtectedFromAccidentalDeletion $true

Add-ADFineGrainedPasswordPolicySubject -Identity "IT-Admin-PSO" -Subjects "IT-Admins"
```

---

## Account Lifecycle Procedures

### New Hire Checklist

```
1. Create AD account with correct OU placement
2. Add to department security group
3. Verify NTFS access to department share (test login)
4. Create osTicket ticket — record completion
5. Notify manager via email
```

### Leaver Procedure (Simulated: jdoe)

```powershell
# Disable account
Disable-ADAccount -Identity "jdoe"

# Move to Disabled Users OU
Move-ADObject -Identity (Get-ADUser "jdoe").DistinguishedName `
              -TargetPath "OU=Disabled Users,DC=lab,DC=local"

# Clear group memberships (retain for 30 days per policy)
# Strip after 30-day hold period
```

### Password Reset

```powershell
Set-ADAccountPassword -Identity "mchen" -Reset `
    -NewPassword (ConvertTo-SecureString "TempP@ss2024!" -AsPlainText -Force)
Set-ADUser -Identity "mchen" -ChangePasswordAtLogon $true
Unlock-ADAccount -Identity "mchen"
```

![ADUC — Account tab showing user properties and unlock option](screenshots/aduc-account-unlock.png)

---

## Stale Account Audit

```powershell
# Find accounts inactive for 90+ days
$cutoff = (Get-Date).AddDays(-90)

Search-ADAccount -AccountInactive -TimeSpan (New-TimeSpan -Days 90) -UsersOnly |
    Where-Object { $_.Enabled -eq $true } |
    Select-Object Name, SamAccountName, LastLogonDate |
    Sort-Object LastLogonDate |
    Export-Csv -Path "C:\Reports\stale-accounts-$(Get-Date -Format yyyyMMdd).csv" -NoTypeInformation
```

**Result:** 0 stale accounts found (lab environment, all accounts recently created).

---

## Skills Demonstrated

- Bulk AD user creation via PowerShell and CSV import
- OU structure management and computer/user placement
- Security group creation and membership management
- NTFS permissions — inheritance, explicit ACEs, group-based access
- Fine-Grained Password Policies (PSO)
- Account lifecycle: onboarding, offboarding, password resets, unlocks
- Stale account auditing and reporting
