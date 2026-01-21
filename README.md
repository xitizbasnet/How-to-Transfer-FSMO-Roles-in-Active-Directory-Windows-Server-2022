## 📘 How to Transfer FSMO Roles in Active Directory | Windows Server 2022

In this guide, we will walk through the **steps required to view and transfer FSMO roles in Active Directory**.

By default, when **Active Directory Domain Services (AD DS)** is installed, **all five FSMO roles are assigned to the first domain controller in the forest root domain**. 

However, there are scenarios where an administrator must move one or more FSMO roles from the default holder domain controller to another domain controller.

➡️ When both the current FSMO role holder and the target domain controller are **online and operational**, the process is known as **FSMO role transfer**.

---

## 🔑 Understanding FSMO Roles

There are **five FSMO roles** in an Active Directory environment. These roles prevent conflicts in multi-domain controller environments by ensuring that specific tasks are handled by only one domain controller.

---

### 🌲 Forest-Wide FSMO Roles

These roles are **unique per Active Directory forest**.

#### 🧩 Schema Master

* Responsible for making changes to the **Active Directory schema**
* Required when upgrading the schema using commands such as:

  * `adprep /forestprep`

#### 🏷️ Domain Naming Master

* Ensures **unique domain and application partition names** within the forest
* Required when creating or removing domains
* 🔐 Requires **Enterprise Admin** privileges to manage

---

### 🏠 Domain-Wide FSMO Roles

These roles exist **once per domain**.

#### ⏱️ PDC Emulator

* Tracks account lockouts and password changes
* Acts as the **authoritative NTP time source**
* Provides backward compatibility for legacy Windows clients
* Used by **DFS root servers** to update namespace information

#### 🔗 Infrastructure Master

* Maintains and updates **cross-domain object references**
* The `adprep /domainprep` command runs on this role holder

#### 🆔 RID Master

* Allocates **Relative ID (RID) pools** (in blocks of 500)
* Ensures all security principals receive **unique SIDs**

🔐 To manage domain-wide FSMO roles, the administrator must be a member of the **Domain Admins** group.

---

## 🔍 Identifying Current FSMO Role Holders

Before transferring FSMO roles, always verify which domain controller currently holds each role.

### 🧪 Command-Line Verification

Run the following command from **Command Prompt or Windows PowerShell (Run as Administrator)**:

```powershell
netdom query fsmo
```

📌 In our environment:

* Forest Type: Single-domain Active Directory forest
* Domain Controllers:

  * `ADDC01` (First DC – initial FSMO holder)
  * `ADDC02` (Second DC)

Currently, **all five FSMO roles reside on ADDC01**.
  * Schema master `ADDC01`  
  * Domain naming master `ADDC01` 
  * PDC `ADDC01`  
  * RID pool manager `ADDC01`  
  * Infrastructure master `ADDC01`  

---

## 🗺️ Planning Considerations

Proper FSMO role placement is critical for **performance, availability, and resilience**.

⚠️ Note: This guide demonstrates FSMO transfers in a **test lab environment**.

FSMO roles can be transferred using:

* 🖱️ Graphical User Interface (GUI)
* ⌨️ Command-Line Interface (CLI – PowerShell)

➡️ PowerShell is generally the **fastest and most efficient method**, but both approaches are documented here.

🔑 Required Permissions:

* Enterprise Admins
* Domain Admins

---

## 🖱️ FSMO Role Transfer Using GUI

### 1️⃣ Transfer Domain-Wide FSMO Roles

Roles:

* RID Master
* PDC Emulator
* Infrastructure Master

📍 Tool: **Active Directory Users and Computers**

Steps:

1. Open **Server Manager**
2. Navigate to **Tools → Active Directory Users and Computers**
3. Right-click the domain(Example: adserver.local) → **Operations Masters**
4. Click on **RID** Tab
5. Ensure the destination DC i.e: **To transfer the operations master role to the following computer. Click Change**  (`ADDC02`) is selected
6. Click **Change** → **Yes** → **OK**

Repeat the process for all three(**PDC Emulator**) tabs.
1. Click on **PDC** Tab
2. Ensure the destination DC i.e: **To transfer the operations master role to the following computer. Click Change**  (`ADDC02`) is selected
3. Click **Change** → **Yes** → **OK**

Repeat the process for all three(**Infrastructure**) tabs.
1. Click on **Infrastructure** Tab
2. Ensure the destination DC i.e: **To transfer the operations master role to the following computer. Click Change**  (`ADDC02`) is selected
3. Click **Change** → **Yes** → **OK**

✅ Result: All domain-wide FSMO roles are transferred to `ADDC02`.

---

### 2️⃣ Transfer Schema Master Role

#### 🔧 Register Schema Management DLL

Open Powershell Run as Administrator:

```powershell
regsvr32 schmmgmt.dll
```

#### 📦 Load Schema Snap-in

1. Run `mmc`
2. **File → Add/Remove Snap-in**
3. Select **Active Directory Schema**
4. Add
5. Click: **OK**

Steps to transfer:

* Right Click on : **Active Directory Schema `ADDC02`**
* Open **Operations Master**
* Click **Change** → **Yes** → **OK**

✅ Schema Master role transferred successfully.

---

### 3️⃣ Change Master Role

📍 Tool: **Active Directory Schema `ADDC02`**

Steps:

1. Active Directory Schema `ADDC02`
2. Right-click root → **Operations Master**
3. Here both the option called: **Current schema master(online)** and **To transfer the schema master role to the targeted schema FSMO holder below:** `ADDC01` 
4. Click **Close** 

---

### 4 Transfer Domain Naming Master Role

📍 Tool: **Active Directory Schema `ADDC02`**

Steps:

1. Active Directory Schema `ADDC02`
2. Right-click  → **Change Active Directory Domain COntroller**
3. Select `ADDC02` 
4. Click **Ok** → **OK**

---

### 5 Change Master Role

📍 Tool: **Active Directory Schema `ADDC02`**

Steps:

1. Active Directory Schema `ADDC02`
2. Right-click root → **Operations Master**
3. Here the option called: **Current schema master(online)** will be `ADDC01` and **To transfer the schema master role to the targeted schema FSMO holder below:** `ADDC02` 
4. Click **Change** → **Yes**  → **OK**
5. After changes the result will be: **Current schema master(online)** will be `ADDC02` and **To transfer the schema master role to the targeted schema FSMO holder below:** `ADDC02`
6. Click **Close**
7. Close the **mmc** window
8. Save console setting to console: **No**

---
### 6 Transfer Domain Naming Master Role

📍 Tool: **Active Directory Domains and Trusts**

Steps:

1. Server Manager → Tools → Active Directory Domains and Trusts
2. Right-click **Active Directory Schema `ADDC02`** → **Operations Master**
3. 3. Here the option called: **Current schema master(online)** will be `ADDC01` and **To transfer the schema master role to the targeted schema FSMO holder below:** `ADDC02` 
4. Click **Change** → **Yes** → **OK**

---

### 🔎 Verification

```powershell
netdom query fsmo
```

✅ Confirm `ADDC02` holds all five FSMO roles.
 * Schema master `ADDC02`  
  * Domain naming master `ADDC02` 
  * PDC `ADDC02`  
  * RID pool manager `ADDC02`  
  * Infrastructure master `ADDC02`

---

## ⌨️ FSMO Role Transfer Using PowerShell (CLI)

PowerShell allows **single or bulk FSMO transfers** using one cmdlet:

Run **Powershell** as an administrator

```powershell
Move-ADDirectoryServerOperationMasterRole
```

### 📊 FSMO Role Reference Table

| Role                  | Number |
| --------------------- | ------ |
| PDC Emulator          | 0      |
| RID Master            | 1      |
| Infrastructure Master | 2      |
| Schema Master         | 3      |
| Domain Naming Master  | 4      |

---

### Example: Transfer RID Master

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity ADDC01 -OperationMasterRole RIDMaster
```

Type **Y** to confirm.

Verify:

```powershell
netdom query fsmo
```

✅ Confirm `RID` roles.
 * Schema master `ADDC02`  
  * Domain naming master `ADDC02` 
  * PDC `ADDC02`  
  * RID pool manager **`ADDC01`**  
  * Infrastructure master `ADDC02`

---

### Example: Transfer PDC Emulator

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity ADDC01 -OperationMasterRole 0
```

Verify:

```powershell
netdom query fsmo
```

✅ Confirm PDC roles.
 * Schema master `ADDC02`  
  * Domain naming master `ADDC02` 
  * PDC **`ADDC01`**  
  * RID pool manager `ADDC02`  
  * Infrastructure master `ADDC02`
---

### 🚀 Transfer All FSMO Roles at Once

```powershell
Move-ADDirectoryServerOperationMasterRole -Identity ADDC01 -OperationMasterRole 0,1,2,3,4
```

Type **A** to accept all prompts.

Verify:

```powershell
netdom query fsmo
```

✅ Confirm `ADDC02` holds all five FSMO roles.
 * Schema master `ADDC01`  
  * Domain naming master `ADDC01` 
  * PDC `ADDC01`  
  * RID pool manager `ADDC01`  
  * Infrastructure master `ADDC01`
---

## ✅ Conclusion

You have learned how to **identify, plan, and transfer FSMO roles** in Active Directory using both **GUI tools and PowerShell**.

✔️ PowerShell provides the **fastest and most scalable approach** for FSMO management.


Thank you for reviewing this guide. If you have questions or suggestions, please document them in the repository issues section.

---

