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
