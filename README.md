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
