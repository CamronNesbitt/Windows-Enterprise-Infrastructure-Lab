# Windows-Enterprise-Infrastructure-Lab
A sandboxed enterprise network environment featuring a Windows Server 2019 Domain Controller and Windows 10 endpoints. Includes automated PowerShell user onboarding, Active Directory management, strict NTFS security configurations, and centralized DHCP/DNS routing.

# Enterprise Active Directory Infrastructure & Systems Automation Sandbox

## 1. Architectural Design & Topology
This section defines the sandboxed corporate network architecture engineered within Oracle VirtualBox to simulate real-world enterprise endpoint communication, identity provisioning, and centralized server roles.

### Environment Specification Matrix
| Node Name | Operating System | Network Card Configuration | IP Assignment Type | IP Address | Roles / Services Hosted |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DC1** | Windows Server 2019 | Internal Network (`mylab-net`) | Static | `192.168.20.10` | AD DS, DNS, DHCP Server |
| **PC1** | Windows 10 Enterprise | Internal Network (`mylab-net`) | Dynamic (DHCP Pool) | Variable Range | Domain Endpoint Client |

### Network Isolation Context
The entire system is locked within a private internal network switch daemon inside the hypervisor. This completely isolates the infrastructure from the local home network, providing a safe, compliant environment to test security policies, firewalls, and credential injections without production risk.

---

## 2. Automated Identity Provisioning (PowerShell Engine)
This section highlights the software automation layer of the infrastructure, focusing on removing manual administrator latency through modular scripting.

### The Production Vulnerability
Manually initializing enterprise user accounts via the standard Active Directory Users and Computers (ADUC) GUI is highly prone to syntactic data drift, inconsistent credential formats, missing department parameters, and extreme administrative overhead during bulk onboarding events.

### Algorithmic Solution Mechanics
A custom PowerShell automation pipeline script was authored to parse a raw corporate employee dataset (`HR_Onboard.csv`) containing string fields: `FirstName`, `LastName`, `Department`, `JobTitle`. 

The core scripting logic performs:
1. **Object Parsing:** An iterative loop processes each entry.
2. **String Manipulation:** Dynamically builds a standard corporate User Principal Name (UPN) using the `first.last` format (e.g., `jane.doe@mylab.local`).
3. **Cryptographic Conversions:** Transforms temporary alphanumeric password plaintext inputs into highly secure, encrypted system string data structures (`AsSecureString`).
4. **Organizational Path Evaluation:** Programmatically reads the `Department` parameter to build variable Distinguished Name target path pointers (`OU=$Department,DC=mylab,DC=local`), completely sorting users automatically into nested Active Directory containers.

### The Automation Source Code Blueprint
```powershell
# Automated Bulk User Creation Pipeline Script
Import-Module ActiveDirectory

$UsersFile = "C:\LabFiles\HR_Onboard.csv"
$UserData = Import-Csv -Path $UsersFile

foreach ($User in $UserData) {
    # Generate standard corporate properties
    $SAMAccountName = "$($User.FirstName).$($User.LastName)".ToLower()
    $PrincipalName  = "$SAMAccountName@mylab.local"
    $SecurePassword = ConvertTo-SecureString "TempPass2026!" -AsPlainText -Force
    
    # Establish target Organizational Unit path variables dynamically
    $TargetOU = "OU=$($User.Department),OU=Mylab Users,DC=mylab,DC=local"
    
    # Validation check: Ensure target container parameters exist before injection
    if (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$TargetOU'") {
        New-ADUser -Name "$($User.FirstName) $($User.LastName)" `
                   -SamAccountName $SAMAccountName `
                   -UserPrincipalName $PrincipalName `
                   -AccountPassword $SecurePassword `
                   -Path $TargetOU `
                   -Enabled $true `
                   -ChangePasswordAtLogon $true `
                   -Title $User.JobTitle `
                   -Department $User.Department
                   
        Write-Host "Successfully provisioned identity node: $PrincipalName inside $User.Department OU" -ForegroundColor Green
    } else {
        Write-Warning "Target structural container path not found for: $TargetOU"
    }
}
```
## 3. Data Boundary Enforcement & Share Security (NTFS & RBAC)
This section details the hardening of network storage systems, demonstrating how to enforce strict corporate data boundaries using nested permissions.

### The Objective
Configure an SMB network file share (`Company_Share`) hosted on the server so that only validated IT department personnel can access or modify administrative documentation, preventing privilege escalation or data leaks from unauthorized users.

### Security Implementation Strategy
To prevent administrative sprawl and ensure scalable security, the following access control parameters were implemented:
* **Disabling File System Inheritance:** Stripped standard top-level directory inheritance rules from the local storage root (`C:\Company_Share`). This removed broad, default domain read/write permissions.
* **Role-Based Access Control (RBAC):** Created an explicit Active Directory Security Group named `IT_Folders`. Instead of assigning local permissions to individual users, the group object is assigned strict **Modify**, **Read & Execute**, and **List Folder Contents** NTFS permissions.
* **User Nesting:** Nested target IT user entities (provisioned via automation) inside the `IT_Folders` Security Group to grant instant, compliant workspace access.

### Permission Mapping Breakdown
| Principal | NTFS Permission | Share Permission | Functional Result |
| :--- | :--- | :--- | :--- |
| `IT_Folders` (Security Group) | Modify, Read & Execute | Full Control | Full capability to read, write, edit, and create folders. |
| `Domain Users` (All Personnel) | None (Explicitly Removed) | Read | Immediate "Access is Denied" security interception. |

---

## 4. Systematic Triage & Infrastructure Auditing
This section documents the diagnostic playbooks and auditing procedures established to maintain network availability and validate domain security baselines.

### Case Study A: Layer-3 Network Isolation Triage
* **The Symptom:** The client workstation (`PC1`) unexpectedly drops communication with the domain controller, stalling user login events and displaying a native network resource execution error.
* **The Diagnostic Pipeline:**
  1. Initiated a localized command-line analysis on the endpoint running `ipconfig /all`.
  2. Isolated an **Automatic Private IP Addressing (APIPA)** fault condition based on an auto-configured IP address return of `169.254.x.x`.
  3. Identified that the client network card was completely unassigned by the centralized pool scope.
  4. Traced the root cause to an unactivated scope configuration status on the Server DHCP daemon manager.
* **The Remediation:** Activated the target scope pool on the Domain Controller, returned to the client command interface, and executed dynamic network lease recycling commands (`ipconfig /release` and `ipconfig /renew`). This successfully re-established communication and restored domain synchronization.

### Case Study B: Session State & Credential Conflict Remediations
* **The Symptom:** When switching environments to test security group boundaries with alternate active user tokens, Windows throws a session execution collision error: *"The network folder specified is currently mapped using a different user name and password."*
* **The Diagnostic Pipeline:** Local operating systems natively cache credential access tokens to network path objects. When changing user profiles quickly, the active session string maintains a limbo state, blocking new access permissions from applying.
* **The Remediation:** Formulated a rapid terminal fix script to purge hidden, cached network shares and clear the execution cache without restarting the adapter card interface:

```cmd
:: Flush all active cached session tokens and force immediate connection teardowns
net use * /delete /y

```
###Case Study C: Centralized Security Auditing via Event Viewer
* **The Objective:** Validate identity security enforcement across the domain by mapping, capturing, and reading structural authentication failure logs without relying on end-user narrative.

* **The Diagnostic Pipeline:**
    1. Intentionally triggered an account lockout constraint by spamming mismatched authentication tokens at the client workstation interface.
    2. Transitioned to the Domain Controller administrative console and initialized the Windows Event Viewer.
    3. Navigated to the Windows Logs \ Security container and executed an explicit data filter query targeting Event ID 4740 (the native system receipt for a domain account lockout).  
    4. Successfully parsed the log payload to extract the exact structural metadata footprint: the literal timestamp, target account identifier, and the caller workstation hardware footprint (PC1) responsible for the alert.
* **The Auditing Code Blueprint:** Formulated an automated administrative query snippet using PowerShell to instantly grab the lockout details directly into the console shell:

```powershell
# Filter the Windows Security Event Log directly for Account Lockout Events (ID 4740)
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} | ForEach-Object {
    [PSCustomObject]@{
        Timestamp   = $_.TimeCreated
        TargetUser  = $_.Properties[0].Value
        CallerMachine = $_.Properties[1].Value
    }
} | Format-Table -AutoSize
