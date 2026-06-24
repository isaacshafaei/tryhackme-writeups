# Windows Domains & Active Directory (AD)

## 🏢 The Scale Problem

* **Workgroup (Decentralized):** Managing computers individually works for small environments (e.g., 5 machines). Scales poorly as the organization grows.
* **Domain (Centralized):** Consolidates users, computers, and configurations into a single, unified network structure.

## 🗄️ Core Components

* **Active Directory (AD):** The centralized database/repository that stores all network data, including user credentials, computer objects, and access rights.
* **Domain Controller (DC):** The dedicated server running Active Directory services. It handles all network authentication and policy enforcement.

## 🏆 Key Benefits

* **Centralized Identity Management:** Users can log into any computer on the network using a single set of credentials checked by the DC.
* **Centralized Security Policies:** Administrators can deploy permissions and restrictions across the entire network instantly (e.g., blocking access to the Control Panel or restricting admin privileges).

---

## 🧭 Concept Map: Centralized vs. Decentralized Networks

---

## ❓ Knowledge Check Answers

* **In a Windows domain, credentials are stored in a centralised repository called...** `Active Directory`
* **The server in charge of running the Active Directory services is called...** `Domain Controller`
---
# Active Directory Objects & Management

## 📂 Core Objects (Security Principals)

Security Principals are entities that can authenticate and act upon network resources.

* **Users:** Represent human employees or background services (like IIS or MSSQL).
* **Machines:** Computers joined to the domain. They automatically get an account with a `$` suffix (e.g., `DC01$`). Machine passwords auto-rotate securely.

## ⚖️ Security Groups vs. Organizational Units (OUs)

Do not confuse the two; they serve entirely different purposes.

* **Security Groups (For Permissions):** Used to grant access rights to resources (like shared folders or printers). A user can be in **multiple** groups.
* *Example:* **Domain Admins** (full control over the domain), Backup Operators, Account Operators.


* **Organizational Units (For Policies):** Containers used to apply specific configurations or restrictions (Group Policies) to a department. A user can only be in **one** OU at a time.

## 🛠️ Management Tool

* **Active Directory Users and Computers (ADUC):** The built-in Windows GUI used on the Domain Controller to create, organize, and manage OUs, Users, and Groups.

---

### Knowledge Check Answers

* **Which group normally administrates all computers and resources in a domain?**
`Domain Admins`
* **What would be the name of the machine account associated with a machine named TOM-PC?**
`TOM-PC$`
* **What type of containers should we use to group all Quality Assurance users so that policies can be applied consistently to them?**
`Organizational Unit` (or `OU`)
---
---
# AD Management & Delegation

## 🗑️ Deleting Protected OUs

OUs are protected from accidental deletion by default.

* **To delete:** Go to **View > Advanced Features** in ADUC, right-click the target OU, select **Properties**, navigate to the **Object** tab, and uncheck **"Protect object from accidental deletion"**.

## 🤝 Delegation of Control

Delegation allows you to grant specific, limited administrative privileges (like resetting passwords) to non-admin users over specific OUs without giving them full Domain Admin rights.

* **To delegate:** Right-click the target OU and select **Delegate Control**, then follow the wizard to specify the user and the exact tasks they are allowed to perform.

## 💻 PowerShell Account Management

Delegated users (like Helpdesk staff) may not have access to the ADUC GUI and must often use PowerShell.

* **Reset a password:** `Set-ADAccountPassword <username> -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password')`
* **Force password change at next logon:** `Set-ADUser -Identity <username> -ChangePasswordAtLogon $true`

---

### Knowledge Check Answers

* **What was the flag found on Sophie's desktop?**
*(You will need to log into the TryHackMe lab machine via RDP using Sophie's new credentials to read the flag file on the desktop).*
* **The process of granting privileges to a user over some OU or other AD Object is called...**
`Delegation`
---

# AD Machine Organization

## 🖥️ The Default Container Problem

By default, newly joined machines (except DCs) are placed in the **Computers** container. Leaving them there makes applying specific Group Policies difficult, as servers and user endpoints require different rules.

## 🗂️ Device Segregation Strategy

Best practice dictates segregating machines into specific Organizational Units (OUs) based on their role:

* **Workstations:** Daily-use machines for regular users (laptops, PCs). Highly privileged accounts should *never* log into these.
* **Servers:** Machines providing network services. Require stricter security policies than workstations.
* **Domain Controllers (DCs):** The most sensitive servers that manage AD and store password hashes. (Windows isolates these in a default "Domain Controllers" OU automatically).

## 🛠️ Actionable Cleanup

To tidy the domain environment:

1. Create dedicated **Workstations** and **Servers** OUs at the domain root.
2. Move respective computer objects from the default `Computers` container into these new, role-specific OUs so you can apply targeted policies later.
---

