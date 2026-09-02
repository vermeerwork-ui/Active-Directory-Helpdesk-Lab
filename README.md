# Windows Server Active Directory & Client Workstation Home Lab
By: vermeerwork-ui

## 1. Project Overview & Objective
The objective of this project was to architect and deploy an isolated, enterprise-grade sandbox environment using Oracle VM VirtualBox. This lab serves as a hands-on training ground for simulating IT Helpdesk operations, systems administration workflows, and core infrastructure management. 

By deploying a Windows Server Domain Controller alongside a Windows 11 Client workstation on a dedicated internal virtual switch, I simulated real-world enterprise infrastructure to practice user lifecycle management, Group Policy deployments, and network troubleshooting.

---

## 2. Infrastructure Architecture & Network Design
To ensure complete isolation from my physical home network (preventing rogue DHCP/DNS issues), the environment utilizes an explicit **VirtualBox Internal Network**.

| Asset | Role | Operating System | Network Configuration | IP Address | DNS Server |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VM 2 (DC)** | Domain Controller / DNS | Windows Server 2022/2025 Standard (Desktop Experience) | Internal Network (`AD-Helpdesk-Net`) | `192.168.10.10` (Static) | `192.168.10.10` (Loopback) |
| **VM 3 (Client)**| Corporate Workstation | Windows 11 Pro | Internal Network (`AD-Helpdesk-Net`) | `192.168.10.50` (Static) | `192.168.10.10` (Points to DC) |

---

## 3. Step-by-Step Implementation Guide

### Phase 1: Deploying and Configuring VM 2 (The Domain Controller)

1. **Virtual Machine Creation:**
   * Initiated a new VM profile in VirtualBox named `Windows-Server-DC`.
   * Mounted the official Microsoft Windows Server Standard Evaluation ISO.
   * Allocated **4096 MB RAM** and **2 vCPUs** to guarantee reliable service operation.
   * Provisioned a **50 GB dynamically allocated virtual hard disk**.
   
2. **Virtual Switch Isolation:**
   * Navigated to VM Settings → **Network**.
   * Modified Adapter 1 from NAT to **Internal Network** and assigned the unique identifier string: `AD-Helpdesk-Net`.
   
   ![VirtualBox Network Settings for VM 2](images/network-settings.png)

3. **Operating System Installation:**
   * Booted the VM and selected the **Standard Evaluation (Desktop Experience)** tier to establish a Graphical User Interface (GUI).
   * Configured localized parameters and finalized the built-in local Administrator credentials upon completion.

4. **Network Interface Optimization (Static IP Mapping):**
   * Accessing `ncpa.cpl`, I targeted the virtual Ethernet adapter properties.
   * Stripped automatic addressing and hardcoded a fixed address topology:
     * **IP Address:** `192.168.10.10`
     * **Subnet Mask:** `255.255.255.0`
     * **Preferred DNS Server:** `192.168.10.10` (Self-referencing layout required for AD infrastructure initialization).
     
   ![Windows Server IPv4 Properties](images/ipv4.png)

5. **Active Directory Domain Services (AD DS) Promotion:**
   * Launched **Server Manager** → **Add Roles and Features** → Checked **Active Directory Domain Services**.
   * Following the binary installations, invoked the promotion wizard to construct a root-level forest: `helpdesk.local`.
   * Assigned operational DSRM fail-safes and completed the automated deployment cycle.

---

### Phase 2: Deploying and Configuring VM 3 (The Windows 11 Corporate Client)

1. **Workstation Provisioning:**
   * Initiated a parallel VM matrix profile in VirtualBox named `Windows11-Client`.
   * Attached the official multi-edition Windows 11 ISO.
   * Hardened the system footprint layout allocating **4096 MB RAM** and **2 vCPUs** to meet Windows 11 architectural prerequisites.
   * Initialized a **50 GB virtual disk container**.

2. **Network Alignment:**
   * Mapped VM 3's network configuration precisely to the **Internal Network** tier named `AD-Helpdesk-Net`. This successfully locked both instances onto the exact same virtual layer-2 broadcast domain.

3. **IP Configurations:**
   * Logged into the localized Windows 11 desktop environment and altered network properties via control management parameters.
   * Overrode APIPA addressing defaults (`169.254.X.X`) with a customized subnet path configuration:
     * **IP Address:** `192.168.10.50`
     * **Subnet Mask:** `255.255.255.0`
     * **Preferred DNS Server:** `192.168.10.10` *(Crucial: Directed traffic directly to VM 2 so the client could actively locate and resolve the target domain controller).*

---

### Phase 3: Directory Architecture & Domain Interconnectivity

1. **Logical Enterprise Organization (OUs & Users):**
   * Inside VM 2, launched **Active Directory Users and Computers (ADUC)**.
   * Designed a root-tier Organizational Unit (OU) labeled `Company Users` to segment normal human capital from default administrative containers.
   * Created a practice identity object: `Ellice Patindol` (User Logon Name: `ElliceP`). Enabled the security flag: *User must change password at next logon*.

   ![Active Directory Users and Computers displaying the new user account](images/feline1.png)

2. **Domain Ingestion (Joining VM 3 to helpdesk.local):**
   * Navigated to **Advanced System Settings** on VM 3 -> **Computer Name** -> **Change**.
   * Shifted membership from Workgroup to **Domain** and called: `helpdesk.local`.
   * Successfully passed the authentication check using domain administrator permissions (`HELPDESK\Administrator`).
   * Rebooted the client machine, successfully breaking the local isolation barrier and exposing the network to corporate network parameters.

   ![Welcome to Domain Dialog](images/welcome%20to%20domain.png)
---

## 4. Hands-on Helpdesk Simulation Tickets & Resolution Logs

To simulate the day-to-day operations of an Enterprise Desktop Support Technician, I used my Active Directory lab sandbox to run five real-world helpdesk troubleshooting scenarios. 

### 🎫 Ticket #001: New Employee Onboarding & Account Provisioning
* **User Issue:** HR submitted a ticket requesting a new corporate account for a new hire, **Ellice Patindol**, starting today in the Operations department.
* **Troubleshooting & Actions Taken:**
  1. Logged into **VM 2 (Domain Controller)** and opened **Active Directory Users and Computers (ADUC)**.
  2. Targeted the custom `Company Users` Organizational Unit (OU).
  3. Provisioned a new User Object: First Name: `Ellice`, Last Name: `Patindol`, User Logon Name: `ElliceP`.
  4. Assigned a strong temporary infrastructure password (`InitialWelcome2026!`).
  5. Enforced security alignment by checking the box: **"User must change password at next logon"**.
* **Resolution Verification:** Switched to **VM 3 (Windows 11 Client)**. Logged in using `ElliceP`. The client machine successfully intercepted the credential and forced a mandatory password change before granting desktop access. 

### 🎫 Ticket #002: Account Lockout Due to Credential Mismatch
* **User Issue:** User `ElliceP` called the helpdesk stating she is locked out of her computer. She receives an error: *"The referenced account is currently locked out and may not be logged on to."*
* **Troubleshooting & Actions Taken:**
  1. Interviewed the user; she admitted to typing her new password incorrectly multiple times on a mobile device sync loop.
  2. Navigated to **VM 2 (Domain Controller)** → **ADUC**.
  3. Located the `ElliceP` account inside the `Company Users` OU, right-clicked, and opened **Properties**.
  4. Selected the **Account** tab.
  5. Identified the active warning checkmark: *"Account is currently locked out on this Active Directory Domain Controller"*.
  6. Checked the box to **Unlock account**, then clicked **Apply** and **OK**.
* **Resolution Verification:** Instructed the user to try logging in again on VM 3. The account successfully authenticated without further lockouts.

### 🎫 Ticket #003: "The Trust Relationship Failed" Secure Channel Broken
* **User Issue:** The Windows 11 client machine was left turned off for several months. Upon booting up, the user receives an error: *"The trust relationship between this workstation and the primary domain failed."*
* **Troubleshooting & Actions Taken:**
  1. Recognized that the workstation's secure machine account password had desynchronized from the Active Directory database.
  2. Logged into **VM 3 (Client)** using the local fallback administrator account (`.\TechSupport`) instead of the domain path.
  3. Opened PowerShell as an Administrator and tested the secure channel integrity by running: `Test-ComputerSecureChannel`. The terminal returned `False`, confirming a broken secure identity channel.
  4. Repaired the active trust identity without removing the machine from the domain by executing:
     ```powershell
     Test-ComputerSecureChannel -Credential (Get-Credential) -Repair
     ```
  5. Authenticated the repair request using the `helpdesk.local\Administrator` credentials when prompted.
* **Resolution Verification:** Ran the test command again, returning a status of `True`. Rebooted VM 3 and successfully authenticated into the user profile.

### 🎫 Ticket #004: Corporate Control Panel Restriction via Group Policy
* **User Issue:** Corporate Security policy mandates that standard workstation users should be blocked from tampering with system settings via the legacy Windows Control Panel.
* **Troubleshooting & Actions Taken:**
  1. Logged into **VM 2 (Domain Controller)** and opened **Group Policy Management** (`gpmc.msc`).
  2. Right-clicked the domain root `helpdesk.local` and selected **Create a GPO in this domain, and Link it here...**
  3. Named the rule **`GPO-Restrict-ControlPanel`**.
  4. Right-clicked the new GPO and chose **Edit** to open the Group Policy Management Editor.
  5. Navigated to: `User Configuration` → `Policies` → `Administrative Templates` → `Control Panel`.
  6. Located and double-clicked **Prohibit access to Control Panel and PC settings**. Changed the status to **Enabled**, clicked **Apply**, and saved.
* **Resolution Verification:** Switched to **VM 3 (Windows 11 Client)** under user account `ElliceP`. Opened Command Prompt and ran `gpupdate /force` to pull down the new policy rules. Attempted to open the Control Panel, and the system blocked it, displaying: *"This operation has been canceled due to restrictions in effect on this computer."*

### 🎫 Ticket #005: Client Network Connectivity & DNS Resolution Failure
* **User Issue:** The user on VM 3 states they cannot reach internal shared folders or authenticate using domain tools. Local command testing states the domain controller cannot be located.
* **Troubleshooting & Actions Taken:**
  1. Remotely walked the user through troubleshooting steps. Opened Command Prompt on **VM 3** and ran `ipconfig /all`.
  2. Analyzed the network dump. Discovered that the Client VM had an IP of `169.254.X.X` (APIPA address), meaning it was completely disconnected from the static network schema.
  3. Discovered that the network card settings inside VirtualBox had defaulted back to NAT instead of the internal network switch.
  4. Shut down VM 3, went to VirtualBox Settings → **Network** → set Adapter 1 back to **Internal Network** with the exact name matching VM 2 (`AD-Helpdesk-Net`).
  5. Booted the VM, opened `ncpa.cpl`, and verified that the manual static parameters (`IP: 192.168.10.50` / `DNS: 192.168.10.10`) were correctly initialized.
* **Resolution Verification:** Executed `nslookup helpdesk.local` from the client terminal. The machine immediately resolved back to the server address `192.168.10.10` with zero dropped packets.

---

## 5. Future Lab Roadmap & Expansions
To further enhance my systems administration capabilities, I plan to deploy the following additions to this environment:
* **DHCP Server Integration:** Install and configure the DHCP Server Role on VM 2 to automatically assign IP addresses dynamically within the `192.168.10.50 - 192.168.10.254` range, eliminating manual client IP configurations.
* **Enterprise File Shares (SMB):** Set up a centralized File Server with custom security permissions (NTFS/Share Permissions) to simulate corporate departmental drives.
* **Active Directory Server 2025 Upgrades:** Utilize the secondary server node slot to practice multi-domain controller replication scenarios and explore newer forest functional tier upgrades.

---

## 6. Key Takeaways & Applied Helpdesk Competencies
Through this project, I engineered and mastered fundamental infrastructure layers critical to enterprise desktop support roles:
* **Network Isolation Principles:** Understanding layer-2 mapping via VirtualBox internal configurations.
* **Identity & Access Management (IAM):** Creating security structures, OUs, and implementing mandatory initial login security parameters.
* **DNS Resolution Mechanics:** Troubleshooting network endpoints by forcing endpoints to interact with targeted host address mappings.

