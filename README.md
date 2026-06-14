## Watch me do the lab HERE
https://www.loom.com/share/0a0b551f19264750b3d9c7ed1e8f84f0

# 🏢 Lab — Active Directory: Identity & Access Management
**Windows Server 2025 · Azure Free Account · On-Premises IAM**

**Author:** Jair Smith
**Difficulty:** 🟡 Intermediate
**Estimated Time:** ⏱ 3–5 Hours (across multiple sessions)
**Estimated Cost:** 💲 $0 — Free tier + evaluation licence
**Platform:** Windows Server 2025 (Azure VM or VirtualBox)

| Certification Alignment | Free Tools Required |
|---|---|
| CompTIA Network+ · Security+ · Azure Administrator | Windows Server 2025 Evaluation (180 days) · Azure Free Account |

| Career Relevance |
|---|
| IT Support · Help Desk · Sysadmin · Cloud Engineer · Security Analyst |

---

## 📋 Overview

Every organisation running Windows infrastructure relies on **Active Directory (AD)** to answer one fundamental question: *who is allowed to do what?*

Active Directory is the **identity backbone** of the enterprise. It controls which users can log into which computers, which groups can access which file shares, and which policies apply to which parts of the organisation. When a new employee joins, IT creates their account in AD and adds them to the right groups — email, shared drives, printers, and applications are provisioned automatically. When they leave, IT disables one account and every door closes simultaneously.

This is not legacy technology. Hybrid environments sync on-premises AD identities to **Microsoft Entra ID** (formerly Azure AD) in the cloud. The concepts you build here — users, groups, OUs, GPOs — map directly to cloud roles.

---

## 🏗️ Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                        LAB ENVIRONMENT                               │
│                        Domain: lab.local                             │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │             DOMAIN CONTROLLER — Windows Server 2025          │   │
│  │             Hostname: DC01 · IP: 10.x.x.x (Azure/Local)      │   │
│  │                                                              │   │
│  │   ┌─────────────────── ACTIVE DIRECTORY ─────────────────┐  │   │
│  │   │                   Forest: lab.local                   │  │   │
│  │   │                                                       │  │   │
│  │   │  lab.local (Domain Root)                              │  │   │
│  │   │  ├── OU=IT                                            │  │   │
│  │   │  │   ├── 👤 alice.chen (IT_Admins)                    │  │   │
│  │   │  │   └── 📋 GPO: IT Security Policy ──────────────┐  │  │   │
│  │   │  │         ├── Min Password: 12 chars              │  │  │   │
│  │   │  │         ├── Complexity: Enabled                 │  │  │   │
│  │   │  │         ├── Screen Lock: 15 min                 │  │  │   │
│  │   │  │         └── USB Access: Denied                  │  │  │   │
│  │   │  │                                                 │  │  │   │
│  │   │  ├── OU=Finance                                    │  │  │   │
│  │   │  │   └── 👤 bob.patel (Finance_Users)              │  │  │   │
│  │   │  │                                                 │  │  │   │
│  │   │  ├── OU=HR                                         │  │  │   │
│  │   │  │   └── 👤 carol.jones (HR_Users)                 │  │  │   │
│  │   │  │                                                 │  │  │   │
│  │   │  ├── OU=Sales                                      │  │  │   │
│  │   │  │   └── 👤 david.smith (Sales_Users)              │  │  │   │
│  │   │  │                                                 │  │  │   │
│  │   │  └── OU=Computers                                  │  │  │   │
│  │   │      └── 🖥️  vm-workstation-01 ◄────────────────────┘  │  │   │
│  │   └───────────────────────────────────────────────────────┘  │   │
│  │                                                              │   │
│  │   DNS Server (authoritative for lab.local)                   │   │
│  │   Kerberos Authentication Service                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                             │  RDP (3389)                            │
└─────────────────────────────┼────────────────────────────────────────┘
                              │
                        🧑‍💻 Administrator
                        (Local Machine / RDP Client)

  IDENTITY FLOW:
  User Login → Domain Controller → Kerberos Auth → Token Issued
             → Group Memberships Evaluated → Access Granted/Denied
```

> **Key Concept:** The Domain Controller is the single source of truth for all identity decisions on the network. Every login, every access request, every policy enforcement flows through it.

---

## 🎯 Objectives

By completing this lab, you will be able to:

- [x] Promote a Windows Server to a **Domain Controller**
- [x] Create and structure **Organisational Units (OUs)** by department
- [x] Create **Security Groups** for role-based access control
- [x] Create **User Accounts** and assign group memberships
- [x] Configure and link **Group Policy Objects (GPOs)** to enforce security settings
- [x] Join a workstation to a domain and verify policy application
- [x] Perform core **Help Desk tasks**: password reset, account unlock, offboarding
- [x] Run **audit queries** to report on inactive accounts and group memberships

---

## 💼 Why This Lab Matters

| Role | Direct Application |
|---|---|
| **IT Support / Help Desk** | Password resets, account unlocks, and group membership changes are the top three ticket types in any enterprise |
| **Sysadmin** | Designing OU structure, deploying GPOs, managing domain-joined machines at scale |
| **Cloud Engineer** | Entra ID uses identical concepts — users, groups, roles, conditional access. On-prem AD knowledge transfers directly |
| **Security Analyst** | AD is the most targeted system in ransomware attacks. Understanding how it works is the foundation of defending it |

---

## ✅ Prerequisites

- [ ] Active Azure Subscription *(Free Tier)* **OR** local machine with 8GB+ RAM for VirtualBox
- [ ] Basic familiarity with Windows Server navigation
- [ ] Remote Desktop (RDP) client installed on your local machine
- [ ] A text editor to save your lab notes and passwords

---

## 🏷️ Lab Variables & Naming Conventions

| Field | Value |
|---|---|
| **Domain Name** | `lab.local` |
| **Forest Name** | `lab.local` |
| **NetBIOS Name** | `LAB` |
| **DC Hostname** | `DC01` |
| **OUs** | `IT` · `Finance` · `HR` · `Sales` · `Computers` |
| **Security Groups** | `IT_Admins` · `Finance_Users` · `HR_Users` · `Sales_Users` |
| **Test Users** | `alice.chen` · `bob.patel` · `carol.jones` · `david.smith` |
| **VM Size (Azure)** | `Standard_B2s` (2 vCPU / 4GB RAM) |

---

## 🖥️ Step 1 — Provision Your Environment

### Option A — Azure (Recommended)

Azure requires no local hardware. Your VM runs in Microsoft's datacentre and you connect via RDP.

1. Go to [azure.microsoft.com/free](https://azure.microsoft.com/free) and create a free account.
2. Sign in to the [Azure Portal](https://portal.azure.com).
3. Search for **Virtual machines** → click **+ Create**.
4. Configure using the settings below, then click **Review + Create** → **Create**.

| Setting | Value | Why |
|---|---|---|
| **Region** | East US | Most available VM sizes under free tier |
| **Image** | Windows Server 2025 Datacenter — Gen2 | Includes free 180-day evaluation licence |
| **Size** | `Standard_B2s` (2 vCPU / 4GB RAM) | Smallest size AD runs comfortably on |
| **Authentication** | Password | Used to RDP in |
| **Public inbound ports** | Allow RDP (3389) | Required to connect from your local machine |
| **OS disk** | Standard SSD | Good performance, included in free tier |

> ⚠️ **Cost Control:** Stop (deallocate) the VM at the end of every session. A B2s runs at ~$0.05/hour. Stopping — not deleting — pauses compute billing. Your $200 free credit will last significantly longer.

**Enable clipboard before connecting (important):**
1. Open the **Remote Desktop** app on your local machine.
2. Enter the VM's public IP address.
3. Click **Show Options** → **Local Resources** tab.
4. Confirm **Clipboard** is checked under *Local devices and resources*.
5. Click **Connect**.

> 💡 If connecting via the Azure Portal browser console, clipboard sharing is unreliable. Instead, download the RDP file from **Connect → Download RDP File** and open it in the native Remote Desktop app. This is the recommended approach for all lab work.

---

### Option B — VirtualBox (Local)

1. Download [VirtualBox](https://www.virtualbox.org) — free, no account required.
2. Download the [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) from Microsoft.
3. In VirtualBox, create a new VM: **4GB RAM**, **60GB disk**, type = *Windows Server 2019/2022*.
4. Mount the ISO, boot, and follow the installation wizard.
5. Select **Windows Server 2025 Datacenter with Desktop Experience** during setup.

> ⚠️ **Minimum local hardware:** 8GB RAM (4GB for VM, 4GB for your OS) · 60GB free disk · Quad-core CPU with virtualisation enabled in BIOS. If your machine has less than 8GB RAM, use Option A.

---

## 🔬 Step 2 — Install Active Directory Domain Services

RDP into your Windows Server VM. **Server Manager** opens automatically on login.

> **What is a Domain Controller?**
> A Domain Controller (DC) is the brain of the entire identity system. When any user logs into the domain, their credentials are validated against the DC. When you create a user account, you create it on the DC. Every machine that joins your network trusts this server to make all authentication decisions.

**GUI method (Server Manager):**
1. Click **Manage** → **Add Roles and Features**.
2. Click **Next** until you reach **Server Roles**.
3. Check **Active Directory Domain Services**.
4. When prompted, click **Add Features** to include management tools.
5. Click **Next** through remaining pages → **Install**.
6. Wait 2–3 minutes. Click **Close** — do not restart yet.

**PowerShell equivalent:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**Also install Group Policy Management Console now:**

> ⚠️ Step 5 requires the Group Policy Management Console (GPMC). Install it now — if you skip this, you will hit a wall mid-lab when the Group Policy Management option does not appear in Server Manager.

```powershell
Install-WindowsFeature -Name GPMC
```

After GPMC installs, close and reopen Server Manager. **Group Policy Management** will now appear under the Tools menu.

---

## 🔬 Step 3 — Promote the Server to a Domain Controller

> Installing the AD DS role does not create a domain. **Promotion** is what creates your forest and domain, and makes this server the authoritative DNS and identity server for everything that joins it.

> **What is a Forest and Domain?**
> A **Forest** is the top-level container of your entire Active Directory structure — the organisation itself. A **Domain** is the named management boundary inside the forest. Ours is `lab.local`. Most small-to-medium organisations run one domain inside one forest.

**GUI method:**
1. In Server Manager, click the **yellow warning flag** at the top right.
2. Click **Promote this server to a domain controller**.
3. Select **Add a new forest**.
4. Set **Root domain name** to: `lab.local`.
5. Click **Next** → set a **DSRM password** and write it down *(disaster recovery use only)*.
6. Click through DNS Options and NetBIOS pages — accept the defaults.
7. Click **Install** — the server will automatically restart when complete.

**PowerShell equivalent:**
```powershell
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

After the restart, log back in as `LAB\Administrator`. You now own the domain.

---

## 🔬 Step 4 — Build the Organisational Structure

Open **Active Directory Users and Computers (ADUC)** from the **Tools** menu in Server Manager.

> **What is an Organisational Unit (OU)?**
> An OU is a folder inside Active Directory. You organise users, computers, and groups by department or function. The real power of an OU is that you can **link a Group Policy** to it — every object inside automatically gets those policies applied. IT gets one policy set. Finance gets another. All from one central place.

### Create Organisational Units

**GUI:** Right-click `lab.local` in ADUC → **New** → **Organizational Unit**. Repeat for each department.

**PowerShell (run all at once):**
```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

---

### Create Security Groups

> **What is a Security Group?**
> A Security Group holds user accounts. Instead of granting resource access to 50 individuals, you grant it to one group — then add users to that group. This is **role-based access control (RBAC)**. Add someone to Finance_Users and they inherit all Finance access instantly. Remove them and it's revoked simultaneously.

**GUI:** Right-click each OU → **New** → **Group**. Set scope to **Global**, type to **Security**.

**PowerShell:**
```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

---

### Create User Accounts & Assign Group Memberships

> **What is a User Account?**
> A User Account represents a person in Active Directory. When someone logs into a domain-joined computer, Windows sends credentials to the DC. The DC validates the password, confirms the account is enabled, and returns a **Kerberos token** listing what this user is permitted to access — all determined by group membership.

> ⚠️ **Run the entire block below at once — do not run it line by line.** The `$password` variable must be defined before the `New-ADUser` commands execute. Select all, then paste the full block into PowerShell and press Enter.

```powershell
# Step 1 — Define the password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — Create all 4 users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — Assign group memberships
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

## 🔬 Step 5 — Configure Group Policy

Open **Group Policy Management** from the **Tools** menu in Server Manager.

> **What is a Group Policy Object (GPO)?**
> A GPO is a rulebook that Windows enforces automatically across every user or computer in an OU. Create one GPO, link it to an OU, and every object inside immediately receives those settings on next login or after running `gpupdate`. Password rules, screen lock timers, USB restrictions, software controls — all enforced centrally without touching each machine.

**Create and link the GPO:**
1. Expand **Forest: lab.local → Domains → lab.local** in Group Policy Management.
2. Right-click the **IT** OU → **Create a GPO in this domain and link it here**.
3. Name it: `IT Security Policy`.
4. Right-click the new GPO → **Edit**.

**Configure the following settings:**

| Policy Path | Setting | Value |
|---|---|---|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | `12` |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | `Enabled` |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | `900 seconds` |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | `Enabled` |

**Validate your GPO:**
1. Join the second VM to `lab.local`.
2. Move its computer account into the **IT** OU.
3. Run `gpupdate /force` on the workstation.
4. Log in as `alice.chen` and verify the screen lock activates after 15 minutes of inactivity.

---

## 🎫 Step 6 — Core Help Desk Tasks

These are the tasks every IT support role expects you to perform on day one. Practice each one on your test accounts.

### Reset a Password
```powershell
# Reset password and force change on next login
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

### Unlock a Locked Account
```powershell
# Unlock after too many failed login attempts
Unlock-ADAccount -Identity "carol.jones"
```

### Disable an Account (Offboarding)
```powershell
# Disable — preserves account history and group memberships for audit
Disable-ADAccount -Identity "david.smith"

# Find all currently disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

> 💡 Always **disable** on offboarding — never delete immediately. Deletion is permanent and removes the audit trail. Most compliance frameworks require account data to be retained for a set period.

### Audit & Reporting
```powershell
# Find accounts inactive for 90+ days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Check all groups a user belongs to
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## ✅ Verification Checklist

Run these commands on your Domain Controller to confirm everything is configured correctly.

| Check | Command | Expected Result |
|---|---|---|
| DC is running | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs |
| Users are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## 🔧 Troubleshooting

| Problem | Fix |
|---|---|
| PowerShell prompts for `Name:` when creating users | You ran `New-ADUser` before defining `$password`. Run the **entire script block at once** — the `$password` line must come first |
| Cannot copy and paste into the VM | Open RDP client → **Show Options** → **Local Resources** → check **Clipboard** → reconnect. Or download the RDP file from Azure portal and open with native Remote Desktop app |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting, or use the static IP of the VM |
| Cannot RDP after domain join | Log in as `LAB\Administrator` (with domain prefix) — not just `Administrator` |
| GPO not applying | Run `gpupdate /force` on the target machine, then `gpresult /r` to view applied policies |
| User cannot log in after creation | Confirm account is **Enabled** and check if `ChangePasswordAtLogon` is blocking first login |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog, or install RSAT: `Add-WindowsFeature RSAT-ADDS` |

---

## 🔒 Security Notes

**Active Directory is the #1 target in enterprise attacks.** Ransomware operators specifically pursue domain admin credentials because compromising the DC means compromising every machine in the domain simultaneously. Understanding how AD is built is the foundation for understanding how it's attacked and defended.

Key security principles reinforced in this lab:

- **Principle of Least Privilege** — Users are placed in the lowest-privilege group their role requires. No user has domain admin rights by default.
- **Group-Based Access Control** — Access decisions are made at the group level, not the individual level. This makes provisioning and deprovisioning auditable and scalable.
- **GPO Enforcement** — Security baselines (password policy, screen lock, USB controls) are enforced by policy — not reliant on user behaviour or individual machine configuration.
- **Account Lifecycle Management** — Disable, never delete, on offboarding. The audit trail stays intact.

---

## 💡 Key Concepts Reference

| Concept | Definition |
|---|---|
| **Active Directory (AD)** | Microsoft's directory service — the identity and access backbone of Windows enterprise environments |
| **Domain Controller (DC)** | The server that runs AD and handles all authentication and policy enforcement for the domain |
| **Forest / Domain** | Forest = the org container; Domain = the named management boundary inside it (`lab.local`) |
| **Organisational Unit (OU)** | A folder in AD used to organise objects and link Group Policies by department or function |
| **Security Group** | A container for user accounts used to grant access to resources at scale |
| **GPO** | A collection of settings automatically enforced across all objects in a linked OU |
| **Kerberos** | The authentication protocol AD uses to issue tokens after validating credentials |
| **DSRM** | Directory Services Restore Mode — an emergency recovery mode requiring a separate password set during promotion |
| **RBAC** | Role-Based Access Control — granting permissions to roles (groups) rather than individuals |
| **ADUC** | Active Directory Users and Computers — the primary GUI for managing the directory |
| **GPMC** | Group Policy Management Console — the tool for creating, linking, and editing GPOs |
| **Entra ID** | Microsoft's cloud-based identity service (formerly Azure AD) — uses the same concepts as on-prem AD |

---

## 📁 Repository Structure

```
lab-ad01-active-directory/
│
├── README.md                  # This file — full lab documentation
└── screenshots/               # Add your validation screenshots here
    ├── dc-promotion.png
    ├── ou-structure.png
    ├── user-creation.png
    ├── gpo-settings.png
    └── verification-commands.png
```

---

## 🧹 Clean Up

**Azure users:**
1. Navigate to **Virtual Machines** in the Azure Portal.
2. Select your VM → click **Stop** to pause billing, or **Delete** to remove it entirely.
3. If deleting, also remove the associated **Resource Group** to clean up disks, NICs, and public IPs.

**VirtualBox users:**
1. In VirtualBox Manager, right-click your VM → **Remove** → **Delete all files**.

---

## 🔗 References

- [Active Directory Domain Services Overview — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Group Policy Overview — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)
- [Microsoft Entra ID (formerly Azure AD)](https://learn.microsoft.com/en-us/entra/identity/)
- [Windows Server 2025 Evaluation Download](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
- [Active Directory Security Best Practices — CISA](https://www.cisa.gov/sites/default/files/publications/best-practices-for-privileged-access.pdf)
- [Azure Free Account](https://azure.microsoft.com/en-us/free/)

---

*Lab authored by Jair Smith
