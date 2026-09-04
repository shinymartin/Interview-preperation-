# Azure L1 Engineer – Atidan | Interview Preparation Q&A

## 1. Windows Server Administration

**Q1. What are the main responsibilities of a Windows Server administrator?**
A: Installing and configuring servers, managing users/groups via Active Directory, patch management, monitoring server health, managing storage/disks, configuring roles and features (DNS, DHCP, File/Print, IIS), backup and recovery, troubleshooting performance and service issues, and ensuring security compliance.

**Q2. What is the difference between a Workgroup and a Domain?**
A: A Workgroup is a peer-to-peer setup where each machine manages its own local accounts/security, suited for small networks. A Domain is a centralized model managed by Active Directory Domain Services (AD DS), where a Domain Controller authenticates and authorizes all users/computers, enabling centralized policy (GPO) and account management.

**Q3. How do you check which services are running on a Windows Server and restart a hung service?**
A: Use `services.msc` (GUI) or PowerShell: `Get-Service` to list, `Get-Service -Name <name> | Restart-Service`, or `net stop <service>` / `net start <service>`. For a hung service, check Event Viewer logs, verify dependencies, and if unresponsive, may need to kill the process via Task Manager/`Stop-Process` and restart.

**Q4. What is the boot process of Windows Server (brief)?**
A: Power-on self-test (POST) → Boot Manager (BOOTMGR) reads BCD → Windows Loader (winload.exe) loads kernel (ntoskrnl.exe) and drivers → Session Manager starts → Winlogon presents login screen → services start per startup type (Automatic/Manual/Disabled).

**Q5. How do you troubleshoot a server that is running slow?**
A: Check Task Manager/Resource Monitor for CPU, memory, disk I/O, and network usage; review Event Viewer (System/Application logs) for errors; check for pending updates or malware; check disk space and fragmentation; review scheduled tasks or runaway processes; check Performance Monitor (PerfMon) counters over time.

**Q6. What is the difference between NTFS and ReFS?**
A: NTFS is the traditional Windows file system supporting permissions, encryption (EFS), compression, quotas. ReFS (Resilient File System) is newer, designed for large-scale storage with better resiliency (auto-correction of certain corruption) but lacks some NTFS features like compression/EFS; commonly used with Storage Spaces.

**Q7. What are common Windows Server roles you should know?**
A: Active Directory Domain Services (AD DS), DNS Server, DHCP Server, File and Storage Services, Print and Document Services, Remote Desktop Services (RDS), IIS (Web Server), Hyper-V, Windows Server Update Services (WSUS).

**Q8. How do you check Windows Update status and apply patches via PowerShell?**
A: `Get-HotFix` to list installed updates; using the PSWindowsUpdate module: `Get-WindowsUpdate`, `Install-WindowsUpdate -AcceptAll -AutoReboot`. In enterprise environments, patching is often managed centrally via WSUS or Azure Update Management.

---

## 2. Active Directory, DNS, DHCP

**Q9. What is Active Directory and what does a Domain Controller do?**
A: Active Directory is Microsoft's directory service that stores information about users, computers, groups, and resources, and enforces security via authentication (Kerberos/NTLM) and authorization. A Domain Controller (DC) hosts the AD database (NTDS.dit), handles logon authentication, and replicates directory data to other DCs.

**Q10. What is the difference between an OU and a Group in AD?**
A: An Organizational Unit (OU) is a container used to organize objects (users, computers, groups) for applying Group Policy and delegating administrative control. A Group is used to assign permissions/access rights to multiple users at once (Security Groups) or for email distribution (Distribution Groups).

**Q11. What are FSMO roles?**
A: Flexible Single Master Operations roles are special DC responsibilities not handled by all DCs:
- Schema Master, Domain Naming Master (forest-wide, one each per forest)
- RID Master, PDC Emulator, Infrastructure Master (domain-wide, one each per domain)

**Q12. What is Group Policy (GPO) and how does it apply?**
A: GPOs are collections of settings (security, software deployment, scripts, restrictions) applied to users/computers in AD. Order of application: Local → Site → Domain → OU (LSDOU), with the last-applied policy generally taking precedence unless "Enforced" is set.

**Q13. What is DNS and why is it important?**
A: DNS (Domain Name System) resolves human-readable hostnames to IP addresses. In AD environments, DNS is critical because domain-joined computers locate Domain Controllers via DNS SRV records; AD replication and Kerberos authentication depend on correct DNS resolution.

**Q14. What are common DNS record types?**
A: A (IPv4 address), AAAA (IPv6 address), CNAME (alias), MX (mail exchange), PTR (reverse lookup), SRV (service location, used heavily by AD), NS (name server), TXT (text/verification records like SPF).

**Q15. How do you troubleshoot DNS resolution issues?**
A: Use `nslookup <hostname>`, `Resolve-DnsName <hostname>`, check DNS server logs, verify forwarders, flush client cache (`ipconfig /flushdns`), check that the correct DNS server is set on the NIC, verify DNS zone records exist and are not stale, and test with `Test-NetConnection -ComputerName <name> -Port 53`.

**Q16. What is DHCP and how does the DORA process work?**
A: DHCP automatically assigns IP addresses, subnet masks, gateways, and DNS servers to clients. DORA process: **D**iscover (client broadcasts) → **O**ffer (server offers IP) → **R**equest (client requests offered IP) → **A**cknowledge (server confirms lease).

**Q17. What is DHCP scope, lease, reservation, and exclusion?**
A: Scope = the range of IPs a DHCP server can assign for a subnet. Lease = duration a client can use an IP before renewal. Reservation = a specific IP always assigned to a specific MAC address. Exclusion = IPs within a scope range that DHCP will not assign (often reserved for static devices).

**Q18. What is DHCP failover/redundancy (80/20 rule)?**
A: To avoid a single point of failure, two DHCP servers can share scopes — traditionally split 80/20 (or configured via DHCP Failover feature in Server 2012+) so one server handles majority of leases and the other provides backup if the primary is down.

---

## 3. Networking Fundamentals

**Q19. Explain TCP/IP and the OSI model briefly.**
A: TCP/IP is the protocol suite that governs how data is transmitted across networks (Application, Transport, Internet, Link layers in the TCP/IP model). The OSI model has 7 layers: Physical, Data Link, Network, Transport, Session, Presentation, Application — used as a conceptual troubleshooting framework.

**Q20. What is subnetting and how do you calculate a subnet?**
A: Subnetting divides a larger network into smaller sub-networks using a subnet mask (CIDR notation, e.g., /24 = 255.255.255.0). Example: 192.168.1.0/24 has 256 addresses (254 usable hosts). A /26 subnet has 64 addresses (62 usable hosts).

**Q21. What is the difference between TCP and UDP?**
A: TCP is connection-oriented, reliable, ensures ordered delivery with acknowledgments (used for HTTP, RDP, SMB). UDP is connectionless, faster, no delivery guarantee (used for DNS queries, streaming, DHCP).

**Q22. What are common networking troubleshooting commands and what do they do?**
A:
- `ping` – tests basic reachability/latency to a host
- `tracert` (Windows) / `traceroute` – shows the path (hops) packets take to reach a destination, helps identify where latency/failure occurs
- `nslookup` – queries DNS to resolve hostnames/IPs
- `Test-NetConnection` (PowerShell) – tests connectivity to a host/port (replaces ping/telnet-like checks), e.g., `Test-NetConnection -ComputerName google.com -Port 443`
- `ipconfig /all` – shows IP configuration details
- `netstat -ano` – shows active connections and listening ports

**Q23. A user reports they cannot access an internal application by name but can by IP. What do you check?**
A: This points to a DNS issue — check the client's DNS server settings, run `nslookup <appname>`, verify the DNS record exists on the internal DNS server, check for stale cache (`ipconfig /flushdns`), and confirm the DNS zone is healthy.

**Q24. What is a default gateway and what happens if it's misconfigured?**
A: The default gateway is the router a device sends traffic to when the destination is outside its local subnet. If misconfigured, the device can communicate with hosts on its own subnet but cannot reach external networks/internet.

---

## 4. Azure Fundamentals (VMs, VNets, Subnets, NSGs, Storage)

**Q25. What is an Azure Resource Group?**
A: A logical container that holds related Azure resources (VMs, storage, networking) for a solution, used for lifecycle management, access control (RBAC), and billing organization. Deleting a resource group deletes all resources within it.

**Q26. What is an Azure Virtual Network (VNet) and Subnet?**
A: A VNet is an isolated private network in Azure where you deploy resources like VMs; it enables resources to securely communicate with each other, the internet, and on-premises networks. A Subnet is a segmented range within a VNet's address space used to organize and secure resources separately (e.g., web tier vs. database tier).

**Q27. What is an NSG (Network Security Group) and how does it work?**
A: An NSG is a set of inbound/outbound security rules (like a firewall) that filter network traffic to Azure resources, applied at the subnet or NIC level. Rules are evaluated by priority (lower number = higher priority), specifying source/destination, port, protocol, and allow/deny action.

**Q28. What are the different Azure VM availability options?**
A: Availability Set (groups VMs across fault domains and update domains within a datacenter), Availability Zone (spreads VMs across physically separate datacenters within a region), and Virtual Machine Scale Sets (auto-scaling groups of identical VMs).

**Q29. What are the types of Azure Storage?**
A: Blob Storage (unstructured data — files, images, backups), File Storage (SMB/NFS file shares), Queue Storage (messaging between app components), Table Storage (NoSQL key-value data), Disk Storage (managed disks for VMs).

**Q30. What is the difference between a Managed Disk and Unmanaged Disk in Azure?**
A: Managed Disks are handled entirely by Azure (storage account management, replication, and availability handled automatically) — the recommended approach. Unmanaged Disks require the user to manage the underlying storage account (legacy approach, largely deprecated).

**Q31. What is Azure Monitor and Log Analytics?**
A: Azure Monitor is the platform for collecting, analyzing, and acting on telemetry (metrics, logs, alerts) from Azure and on-prem resources. Log Analytics is the workspace/query engine (using Kusto Query Language – KQL) used to analyze log data collected by Azure Monitor, often used with alert rules to notify on thresholds or anomalies.

**Q32. What is Azure Backup and how does it work for VMs?**
A: Azure Backup is a service that backs up VMs, files, and application data to a Recovery Services Vault. It uses a Backup Policy defining schedule and retention. Backups are application-consistent (using VSS on Windows) and can be restored at the VM level, disk level, or individual file level.

**Q33. What is a Recovery Services Vault?**
A: A storage entity in Azure that holds backup data and recovery points for VMs, SQL, and other supported workloads, and is also used for Azure Site Recovery (disaster recovery/replication).

**Q34. How do you troubleshoot an Azure VM that is unreachable via RDP?**
A: Check the VM's power/running state in the portal; verify NSG rules allow inbound RDP (port 3389) from the source IP; check the VM's public IP is assigned and correct; use "Boot diagnostics" to view the console screenshot; use "Serial Console" or "Run Command" (e.g., to restart RDP service or check firewall) if RDP itself is broken; check the guest OS firewall; verify the VM isn't out of disk space (common cause of RDP failure).

**Q35. What is the difference between a Basic and Standard Load Balancer/Public IP in Azure? (conceptual)**
A: Standard SKU offers zone redundancy, higher scale, secure-by-default (requires explicit NSG to allow traffic), and SLA-backed guarantees, whereas Basic SKU is simpler with fewer features and no SLA — Microsoft has been retiring Basic SKU in favor of Standard.

---

## 5. Microsoft Entra ID (Azure AD) & RBAC

**Q36. What is Microsoft Entra ID and how is it different from on-prem Active Directory?**
A: Microsoft Entra ID (formerly Azure AD) is a cloud-based identity and access management service used for authenticating users to cloud apps (Microsoft 365, Azure, SaaS apps) using protocols like OAuth2/SAML. Unlike on-prem AD, it doesn't use OUs/GPOs the same way and is primarily flat with security groups, and it supports Single Sign-On (SSO), Conditional Access, and MFA.

**Q37. What is Azure AD Connect / Entra Connect?**
A: A tool that synchronizes on-premises Active Directory identities with Microsoft Entra ID, enabling hybrid identity (same credentials work for both on-prem and cloud resources). Supports Password Hash Sync, Pass-through Authentication, or Federation (ADFS).

**Q38. What is Azure RBAC (Role-Based Access Control)?**
A: A system for granting granular access to Azure resources by assigning roles (e.g., Owner, Contributor, Reader, or custom roles) to users/groups/service principals at a specific scope (Management Group, Subscription, Resource Group, or Resource level).

**Q39. What is the difference between Owner, Contributor, and Reader roles?**
A: Owner has full access including the ability to assign roles to others. Contributor can create/manage resources but cannot grant access to others. Reader can only view resources, not modify them.

**Q40. What is Conditional Access and MFA?**
A: Conditional Access is a policy engine in Entra ID that grants/blocks/requires additional verification for access based on conditions (user, location, device, risk level). MFA (Multi-Factor Authentication) requires a second verification factor (e.g., authenticator app, SMS) beyond just a password, significantly reducing account compromise risk.

---

## 6. PowerShell Basics

**Q41. Why is PowerShell important for an L1 Azure/Windows engineer?**
A: It enables automation of repetitive tasks (user creation, service checks, reporting), bulk operations (e.g., managing hundreds of AD accounts), and is the primary CLI for managing both Windows Server and Azure resources (via Az module).

**Q42. What is the basic syntax structure of a PowerShell cmdlet?**
A: Verb-Noun format, e.g., `Get-Service`, `Set-ADUser`, `New-AzResourceGroup`, `Restart-Computer`. Verbs indicate the action (Get, Set, New, Remove, Restart) and Nouns indicate the target.

**Q43. How do you connect to Azure using PowerShell?**
A: `Connect-AzAccount` to sign in, `Get-AzSubscription` to list subscriptions, `Set-AzContext -Subscription "<name>"` to select one, then run cmdlets like `Get-AzVM`, `New-AzResourceGroup`, `Start-AzVM`, `Stop-AzVM`.

**Q44. Write a simple PowerShell command to check disk space on a server.**
A: `Get-PSDrive -PSProvider FileSystem` or `Get-WmiObject Win32_LogicalDisk | Select-Object DeviceID, @{N='FreeGB';E={[math]::Round($_.FreeSpace/1GB,2)}}`

**Q45. How would you export a list of all AD users to a CSV file?**
A: `Get-ADUser -Filter * -Properties * | Select-Object Name, SamAccountName, EmailAddress | Export-Csv -Path "C:\Users.csv" -NoTypeInformation`

**Q46. What is the difference between a PowerShell script (.ps1) and a module?**
A: A script is a single file containing a sequence of commands to perform a task. A module is a packaged collection of related cmdlets/functions (e.g., ActiveDirectory module, Az module) that can be imported to extend PowerShell's capabilities.

---

## 7. ITIL, Incident & Change Management

**Q47. What is ITIL and why does it matter in operations roles?**
A: ITIL (Information Technology Infrastructure Library) is a framework of best practices for IT service management, covering processes like Incident, Problem, Change, and Service Request Management — it ensures consistent, predictable handling of IT operations and minimizes business disruption.

**Q48. What is the difference between Incident Management and Problem Management?**
A: Incident Management focuses on restoring normal service as quickly as possible after a disruption (a fire-fighting/reactive process). Problem Management focuses on identifying and addressing the root cause of recurring incidents to prevent them from happening again (proactive).

**Q49. What is the difference between a Standard, Normal, and Emergency Change?**
A: Standard Change is pre-approved, low-risk, and repeatable (e.g., routine patching). Normal Change requires assessment and approval through the Change Advisory Board (CAB) before implementation. Emergency Change is implemented urgently to resolve a major incident, often with expedited/retrospective approval.

**Q50. How do you prioritize and escalate an incident as an L1 engineer?**
A: Priority is typically determined by Impact × Urgency (per the organization's priority matrix). L1 handles initial triage, applies known fixes/runbooks, and if unable to resolve within SLA or if it's beyond scope/access level, escalates to L2/L3 with full details (steps taken, logs, error messages, business impact) to avoid delay and duplication of effort.

**Q51. What is an SLA and why is it important?**
A: A Service Level Agreement defines the agreed response and resolution times based on incident priority/severity, ensuring accountability and setting client expectations for support quality.

---

## 8. Troubleshooting Scenario Questions

**Q52. A user says they can't log in to their domain-joined PC — how do you troubleshoot?**
A: Verify the account isn't locked/disabled/expired in AD (`Get-ADUser -Identity <user> -Properties LockedOut,Enabled`); check password expiry; confirm the PC can reach a Domain Controller (network/DNS issue); check time sync (Kerberos fails if clock skew >5 min); check Event Viewer Security logs for the failure reason (e.g., Event ID 4625).

**Q53. An Azure VM shows "high CPU" alert — walk through your troubleshooting steps.**
A: Log into the VM (RDP or Run Command if RDP fails); check Task Manager/Resource Monitor for the process consuming CPU; check for runaway processes, malware, or scheduled tasks; review Azure Monitor metrics for CPU trend over time; check if it's an application issue (e.g., unoptimized query) vs. undersized VM (may need to resize the VM SKU); document findings and escalate if root cause is application-level (L2/L3 or app team).

**Q54. A newly created VNet's VM cannot reach the internet. What could be wrong?**
A: Check the NSG for outbound deny rules; verify there's no forced tunneling/UDR (User Defined Route) sending traffic incorrectly; check if a firewall/NVA is intended to handle outbound traffic; confirm the VM has a Public IP or is behind a NAT Gateway/Load Balancer for outbound connectivity, since VMs without any of these have no default outbound path in some configurations.

**Q55. How would you troubleshoot slow file access on a file server?**
A: Check server resource utilization (CPU, memory, disk I/O); check network latency between client and server (ping, tracert); check for SMB protocol issues or version mismatches; review antivirus scanning impact; check Event Viewer for storage/controller errors; verify no bandwidth-heavy processes/backups are running concurrently.

---

## 9. Behavioral / General Questions

**Q56. Tell me about a time you resolved a critical incident under pressure.**
A: (Use the STAR method — Situation, Task, Action, Result. Focus on: how you triaged, communicated with stakeholders, followed escalation procedures, and what the resolution/outcome was, plus what you learned or improved afterward.)

**Q57. How do you handle a situation where you don't know the answer to an issue a user is reporting?**
A: Acknowledge the issue, gather all relevant details/logs, check internal knowledge base/runbooks, attempt safe diagnostic steps within your access level, and escalate promptly to L2/L3 with clear documentation rather than guessing or making unauthorized changes.

**Q58. Why do you want to work as an Azure L1 Engineer, and where do you see this role taking you?**
A: (Tailor this to your background — emphasize interest in cloud infrastructure, hands-on Windows Server/Azure experience, willingness to learn certifications like AZ-104, and long-term interest in growing into L2/Cloud Engineer/Administrator roles.)

**Q59. How do you keep your technical knowledge updated?**
A: Mention Microsoft Learn, hands-on labs/sandbox environments, pursuing certifications (AZ-900/AZ-104), following Azure updates blog, and practicing in a home lab or free-tier Azure subscription.

---

## 10. Good-to-Have Topic Quick Notes

- **AZ-900**: Azure Fundamentals — covers cloud concepts, core Azure services, pricing/support.
- **AZ-104**: Azure Administrator Associate — covers identity, governance, storage, compute, networking, monitoring in depth.
- **Azure Site Recovery (ASR)**: Disaster recovery service that replicates VMs to another Azure region (or on-prem to Azure) for business continuity.
- **Intune**: Microsoft's cloud-based endpoint/device management (MDM/MAM) solution for managing PCs and mobile devices.
- **Microsoft Defender**: Security suite (Defender for Endpoint, Defender for Cloud) providing threat protection across endpoints and cloud resources.
- **Terraform/Bicep**: Infrastructure-as-Code (IaC) tools used to provision Azure resources declaratively/repeatably instead of manual portal clicks.
- **Hybrid environment**: A setup combining on-premises infrastructure with Azure cloud resources, often connected via VPN Gateway or ExpressRoute, with identity synced via Entra Connect.

---

## Tips for the Interview
1. Be ready to explain concepts in your own words with a real (even small/lab) example — interviewers value practical clarity over textbook definitions.
2. For every "What is X" question, try to also mention "when/why you'd use it" and a related troubleshooting command if applicable.
3. If you haven't used a "Good to Have" tool hands-on, be honest but show conceptual understanding and willingness to learn.
4. Keep answers structured and concise — L1 roles value clear communication as much as technical accuracy.
5. Practice explaining a past troubleshooting incident using the STAR method in under 2 minutes.

Good luck with your interview at Atidan!
