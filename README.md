# Cloud Help Desk Lab: Active Directory + Support Tasks (AWS)

This project simulated a small business Active Directory (AD) environment, focused on user, group, and organizational unit (OU) management, as well as common help desk support scenarios using Windows Server and Group Policy. It allowed me to gain hands-on experience configuring a domain controller, structuring user accounts by department, enforcing security policies, and resolving real-world IT support tickets through GUI and PowerShell.

---

## ✅ Overview

| Item             | Details                                     |
|------------------|---------------------------------------------|
| **Platform**     | AWS Free Tier                               |
| **OS**           | Windows Server 2019/2022                    |
| **Main Role**    | Domain Controller (AD DS)                   |
| **Access**       | Remote Desktop (RDP) via EC2                |
| **Skills**       | Active Directory, PowerShell, GPO, EC2      |

---

## 🛠️ PHASE 1: AWS SETUP

### 1️⃣ Launch EC2 Windows Server

- Name: `AD-DC1`
- AMI: `Windows Server 2019 Base`
- Instance Type: `t2.micro` (Free Tier)
- Key Pair: `cloud-helpdesk-key` (.pem format)
- Networking:
  - Enable RDP (Port 3389)
  - Auto-assign public IP


<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/e00e9e50-0c76-4cac-8df7-ba06095766f4" 
    height="50%" 
    width="50%" 
    alt="SS_EC2_DB_2"
  />
</p>
 
---

### 2️⃣ Connect to Server (RDP)

#### Mac Instructions:

1. Download **Microsoft Remote Desktop** from App Store.
2. Add PC → Use **Public IPv4** from AWS
3. Username: `Administrator`
4. Password: Decrypted via AWS with `.pem` key


---
## 🛠️ PHASE 2: Server Configuration

### 3️⃣ Rename Computer
```powershell
Rename-Computer -NewName "AD-DC1" -Restart
```

#### 4️⃣ Install Active Directory Domain Services (AD DS)

**Steps:**

1. Open **Server Manager**
2. Click **Add Roles and Features**
3. Choose **Role-based or feature-based installation**
4. Select the local server (e.g., `AD-DC1`)
5. Under **Server Roles**, check **Active Directory Domain Services**
6. Click **Next** through Features and Confirmation
7. Click **Install** (do not close the wizard)
8. Wait for the installation to complete

---

#### 5️⃣ Promote Server to Domain Controller

**Steps:**

1. After AD DS installs, click **"Promote this server to a domain controller"** in the post-installation tasks
2. Select **Add a new forest**
3. Set **Root domain name**: `corp.local`
4. On Domain Controller Options:
   - Leave default selections (Domain Name System [DNS] Server, Global Catalog)
   - Set a **DSRM (Directory Services Restore Mode)** password
5. Click **Next** through all other options, accepting defaults
6. Click **Install**
7. The server will automatically **restart** when done

**Tip:** After reboot, the system becomes a domain controller for the `corp.local` domain.


---

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/b038a740-7283-4d23-bf88-ea5bc4ecf145" 
    height="50%" 
    width="50%" 
    alt="SS_AD-DS_1"
  />
</p>

--- 

## 🏗️ PHASE 3: Build Active Directory Help Desk Environment

---

### 6️⃣ Create Organizational Units (OUs)

**Tool:** Active Directory Users and Computers (ADUC)

**Steps:**

1. Open **Active Directory Users and Computers**
2. Right-click on the domain (`corp.local`) and select **New → Organizational Unit**
3. Create the following OUs:
   - `IT`
   - `Sales`
   - `HR`
   - `Security`

> ✅ Tip: Make sure “Protect container from accidental deletion” is checked for each OU.


<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/8808cb2c-0529-4b2c-a0e4-6e437b4bcb17" 
    height="50%" 
    width="50%" 
    alt="SS_OU_Tree_1"
  />
</p>

---

### 7️⃣ Create 10 Sample Users

**Tool:** PowerShell (Run as Administrator on the Domain Controller)

**Script:**

```powershell
Import-Module ActiveDirectory

$users = @(
    @{name="James Smith"; sam="jsmith"; ou="IT"},
    @{name="Alicia Morgan"; sam="amorgan"; ou="HR"},
    @{name="Brian Lee"; sam="blee"; ou="Sales"},
    @{name="Tasha Johnson"; sam="tjohnson"; ou="IT"},
    @{name="Jordan White"; sam="jwhite"; ou="Sales"},
    @{name="Erica Chen"; sam="echen"; ou="HR"},
    @{name="Samuel Green"; sam="sgreen"; ou="Security"},
    @{name="Nicole Brown"; sam="nbrown"; ou="HR"},
    @{name="Carl Robinson"; sam="crobinson"; ou="Sales"},
    @{name="Victor Daniels"; sam="vdaniels"; ou="IT"}
)

foreach ($u in $users) {
    $securePass = ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force
    New-ADUser -Name $u.name -SamAccountName $u.sam -AccountPassword $securePass -Enabled $true -Path "OU=$($u.ou),DC=corp,DC=local"
}
```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/78a9828b-191d-4a4f-bff6-3e6a82d8bb57" 
    height="50%" 
    width="50%" 
    alt="SS_PowerShell_Script_1"
  />
</p>


> 🔐 Note: All users will be created with the password: `P@ssw0rd123` and placed in their respective OUs.


### 8️⃣ Create Groups and Assign Roles

**🛠️ Tool:** Active Directory Users and Computers (ADUC)

---

#### 📌 Steps:

1. Open **Active Directory Users and Computers (ADUC)**
2. Navigate to your domain (e.g., `corp.local`)
3. Create the following **Security Groups** under the appropriate Organizational Units (OUs) or directly under the domain root:

   - `IT_Admins`
   - `HR_ReadOnly`
   - `Sales_Team`

4. Add the following users to each group:

   - `IT_Admins`:  
     - `jsmith`  
     - `tjohnson`  
     - `vdaniels`  

   - `HR_ReadOnly`:  
     - `amorgan`  
     - `echen`  
     - `nbrown`  

   - `Sales_Team`:  
     - `blee`  
     - `jwhite`  
     - `crobinson`  


<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/a22b0a97-f340-40bc-8cbb-8184e11b7176" 
    height="50%" 
    width="50%" 
    alt="SS_ADUC_OU_Tree_1"
  />
</p>

---

> 💡 **Note:** Group memberships help simplify permissions and access control management for shared resources like files, folders, printers, and Group Policy applications.

---


## 🧰 PHASE 4: Simulate Help Desk Support Tasks

---

### 🎟️ Ticket 1: User Onboarding

**Task:**  
- Create new user `jallen` in the **HR** Organizational Unit.  
- Add `jallen` to the `HR_ReadOnly` group.  
- Set password and verify login (if a test client is available).

**Tool:** Active Directory Users and Computers (ADUC)

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/7b00aeb0-f07e-4d97-8075-f1d2e221744a" 
    height="50%" 
    width="50%" 
    alt="SS_Add_New_User_1"
  />
</p>

---

### 🎟️ Ticket 2: Password Reset

**Task:**  
- Reset the password for user `blee`.
- This can be down via PowerShell or manually Active Directory User and Computers interface.

**PowerShell Command:**

```powershell
Set-ADAccountPassword -Identity blee -Reset -NewPassword (ConvertTo-SecureString "NewP@ssw0rd2025" -AsPlainText -Force)
```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/941a2584-a793-4681-a940-4a814b3290f1" 
    height="50%" 
    width="50%" 
    alt="SS_User_PW_Reset_2"
  />
</p>

---

### 🎟️ Ticket 3: Account Lockout

**Task:**  
Unlock user `jwhite` after multiple failed login attempts.

**Steps:**

1. Open **PowerShell** as Administrator
2. Run the following command:

   ```powershell
   Unlock-ADAccount -Identity jwhite
   ```

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/18080481-3f76-4fd4-a9d4-92b21a5daf0d" 
    height="50%" 
    width="50%" 
    alt="SS_Unlock_UserAcct_1"
  />
</p>

> **Note:** For the user account, under **Reset Password**, you can see if the checkbox for **Unlock the user's account** is checked or not. If checked, hit **OK** to Unlock the account.
---

### 🎟️ Ticket 4: Disable Control Panel via GPO for Sales

**Task:**  
Disable access to the Control Panel for users in the **Sales** Organizational Unit using Group Policy.

**Steps:**

1. Open **Group Policy Management**
2. Create `Create new GPO: DisableControlPanel-Sales`
3. Link it to **Sales OU**

**Path:**  `User Configuration → Administrative Templates → Control Panel → Prohibit access to Control Panel`

<p align="center">
  <img 
    src="https://github.com/user-attachments/assets/e9c2e0b0-3b4e-4c71-bdb8-2424766a3020" 
    height="50%" 
    width="50%" 
     alt="SS_GPM-GPO_1"
  />
</p>

---

> 💡 **Note:** Disabling Control Panel access helps enforce security policies and prevent unauthorized configuration changes on workstations, especially in non-IT departments like Sales.
---


## Key Skills Demonstrated:
- Installed and promoted a Windows Server to Domain Controller with DNS.
- Structured AD using Organizational Units for HR, Sales, IT, and Security.
- Automated the creation of users and assigned them to security groups.
- Simulated help desk tasks like onboarding, password resets, and account unlocks.
- Used Group Policy to apply security restrictions (e.g., disable Control Panel).

## Lessons Learned:
- **OU and Group Structure Matters**: Organizing users logically makes permissions easier to manage.
- **Group Policy is Powerful**: Centralized settings reduce repetitive manual work and help standardize user environments.
- **PowerShell Saves Time**: Automating repetitive tasks (like creating users or resetting passwords) is essential in larger environments.
- **Help Desk Tasks are Foundational**: Even basic support tickets contribute to security, uptime, and employee productivity.
- **Documentation is Key**: Clear steps and screenshots make a project replicable and show readiness for real-world troubleshooting.



