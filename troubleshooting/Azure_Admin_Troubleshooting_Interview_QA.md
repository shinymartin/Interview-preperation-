# Azure Administrator – Troubleshooting Interview Q&A

A curated set of the most commonly asked Azure troubleshooting questions for an Azure Administrator interview round, organized by topic area.

---

## 1. Virtual Machines (VMs)

### Q1. A VM shows as "Running" in the portal but you cannot RDP/SSH into it. How do you troubleshoot?
**A.**
1. Check **Boot diagnostics** (screenshot of the VM console) to see if the OS actually booted.
2. Use **Azure Serial Console** to get direct access without network dependency.
3. Verify **NSG rules** (both subnet-level and NIC-level) allow inbound RDP (3389) / SSH (22) from your IP.
4. Check the **VM's OS firewall** (Windows Firewall / iptables) isn't blocking the port.
5. Confirm the VM has a **Public IP** or you're connecting via VPN/ExpressRoute/Bastion if using private IP.
6. Run **Network Watcher → IP flow verify** and **Connection troubleshoot** to pinpoint where traffic is being dropped.
7. Check if the **VM Agent (WaAppAgent/WALinuxAgent)** is running and healthy.
8. Look at **Azure Bastion** as an alternative access path if NSG/public IP is the issue.

### Q2. VM deployment fails with "Allocation Failed" or "OS Provisioning Timeout". What do you do?
**A.**
- **Allocation Failed**: Usually a capacity issue in that region/zone for the chosen VM size. Try a different size, availability zone, or region; or use a **Proximity Placement Group** alternative.
- **OS Provisioning Timeout**: Often caused by issues with the VM Agent not communicating, unattended install script failing, or a corrupted custom image. Check boot diagnostics/serial console logs for the specific stage where provisioning failed.

### Q3. A VM is running slow / high CPU. How do you diagnose?
**A.**
1. Check **Azure Monitor Metrics** (CPU %, Disk IOPS, Network In/Out) for the VM.
2. Enable/check **Azure Monitor for VMs (VM Insights)** for deeper performance data.
3. RDP/SSH in (if possible) and check Task Manager / `top`/`htop` for the offending process.
4. Check **Disk metrics** — if using Standard HDD, IOPS throttling could be the bottleneck; consider Premium SSD.
5. Review **VM size** — it may be undersized for the workload; consider resizing.
6. Check for **noisy neighbor** issues (rare, but proximity placement/host issues can occasionally play a role).

### Q4. How do you troubleshoot a VM that keeps unexpectedly rebooting?
**A.**
- Check **Activity Log** for platform-initiated reboots (host maintenance/patching).
- Check **Event Viewer** (Windows) or `/var/log` (Linux) for OS-level crash/reboot causes.
- Verify if **Auto-shutdown** policy is configured.
- Check for **Scheduled Events** (Azure Metadata Service) indicating planned maintenance.
- Rule out memory/disk exhaustion causing OS-level crashes.

---

## 2. Networking

### Q5. Two VMs in peered VNets cannot communicate. What are the checks?
**A.**
1. Confirm **VNet peering status** is "Connected" on both sides (not just "Initiated").
2. Check **"Allow forwarded traffic"** and **"Allow gateway transit"** settings if relevant.
3. Verify **NSGs** on both subnets/NICs allow the traffic.
4. Check for **overlapping address spaces** (peering fails/works oddly if CIDRs overlap).
5. Verify **UDRs (User Defined Routes)** aren't sending traffic to a black hole (e.g., forced tunneling through an NVA/firewall that's misconfigured).
6. Use **Network Watcher's Connection Troubleshoot** and **Topology** tools.

### Q6. A user reports "This site can't be reached" for an app hosted on Azure. How do you approach it?
**A.**
1. Confirm DNS resolution — is the domain resolving to the correct Azure resource (Public IP / App Service / Front Door)?
2. Check if it's a **regional/global outage** via **Azure Status page**.
3. Test connectivity with `nmtstat`, `Test-NetConnection`, or `curl` from different networks.
4. Check **NSG**, **Azure Firewall**, or **WAF** rules for the resource blocking traffic.
5. If using **App Service**, check **health check** and **App Service Plan** status.
6. Check **Load Balancer/Application Gateway backend pool health probes** — an unhealthy backend removes it from rotation.

### Q7. NSG rules look correct, but traffic is still blocked. What's next?
**A.**
- Use **Effective Security Rules** (on the NIC) to see the combined result of subnet + NIC NSGs — rule precedence/order issues are common.
- Check for **Azure Firewall** or **third-party NVA** sitting in the path with its own deny rules.
- Check **Application Security Groups (ASGs)** are correctly assigned.
- Verify there isn't a **higher-priority Deny rule** overriding your Allow rule (lowest number = highest priority).
- Confirm **Service Endpoints/Private Endpoints** configuration if targeting PaaS services.

### Q8. How do you troubleshoot a Site-to-Site VPN connection that shows "Not Connected"?
**A.**
1. Check **Gateway status** and **connection status** in the portal.
2. Verify **shared key (PSK)** matches on both ends.
3. Confirm **on-premises public IP** hasn't changed.
4. Check **IKE/IPsec parameters** (encryption, integrity, DH group, SA lifetimes) match between Azure and on-prem device.
5. Review **VPN diagnostic logs** and enable **packet capture** on the Azure Gateway if needed.
6. Confirm **on-prem firewall** allows UDP 500/4500 and ESP protocol outbound.

---

## 3. Storage

### Q9. Users report "403 Forbidden" when accessing a Storage Account/Blob. Troubleshooting steps?
**A.**
1. Check **Storage Account Firewall/Network rules** — is the client's IP or VNet allowed?
2. Verify **SAS token** hasn't expired or has correct permissions/scope.
3. Check **Azure AD RBAC roles** (e.g., Storage Blob Data Reader/Contributor) if using Azure AD auth.
4. Confirm **container/blob access level** (Private vs Blob vs Container).
5. Check if **Private Endpoint** is required and DNS is resolving to the private IP correctly.

### Q10. A managed disk shows as "unattached" but you can't delete it. Why, and how do you fix it?
**A.**
- It may still be referenced by a **snapshot**, in a **VM's disk swap operation**, or have a **resource lock** applied.
- Check for **Azure Backup** dependencies (recovery points referencing the disk).
- Remove any **locks** (Delete/ReadOnly) at resource/resource-group level, then retry deletion.

---

## 4. Identity & Access (Azure AD / Entra ID)

### Q11. A user can't log in and gets "AADSTS50126" or similar sign-in errors. How do you troubleshoot?
**A.**
1. Check the exact **AADSTS error code** — each maps to a specific cause (e.g., 50126 = invalid credentials, 50053 = account locked, 50076 = MFA required).
2. Review **Azure AD Sign-in logs** for the failed attempt details.
3. Check **Conditional Access policies** that may be blocking sign-in (location, device compliance, MFA).
4. Verify the **user account status** (enabled/disabled, licensed correctly).
5. Check for **password expiration** or **hybrid sync issues** (Azure AD Connect) if it's a synced on-prem account.

### Q12. RBAC role assigned to a user isn't taking effect. What do you check?
**A.**
- Confirm the **scope** of assignment (Management Group/Subscription/Resource Group/Resource) — assigned at wrong scope is a common issue.
- Check for a **Deny assignment** (rare, usually from Blueprints/Managed Apps) overriding the role.
- RBAC changes can take a few minutes to propagate — verify with **"Check access"** blade.
- Confirm it's an **Azure RBAC** role and not confused with a **classic admin role** or **Azure AD role** (different systems).

### Q13. Azure AD Connect sync isn't syncing new on-prem users to Azure AD. Troubleshooting steps?
**A.**
1. Check **Azure AD Connect Health** for sync errors.
2. Verify the **sync scheduler** is running (`Get-ADSyncScheduler`).
3. Check for **OU filtering** — the new user might be in an OU excluded from sync.
4. Look for **duplicate attribute errors** (e.g., duplicate UPN/proxyAddress).
5. Run a **manual delta sync** (`Start-ADSyncSyncCycle -PolicyType Delta`) and review the **Synchronization Service Manager** for errors.

---

## 5. App Services & PaaS

### Q14. An App Service is returning HTTP 502/503 errors. How do you troubleshoot?
**A.**
1. Check **App Service diagnostics** ("Diagnose and solve problems" blade) — has built-in detectors for 5xx errors.
2. Review **Application/Web Server logs** (enable logging if not already).
3. Check **App Service Plan** metrics — CPU/Memory quota exhaustion causes 503s.
4. Verify **health check** configuration and startup command/deployment success.
5. Check for **cold start** issues or **scaling** delays under load.
6. If using deployment slots, check if a **recent deployment/swap** introduced the issue — consider swapping back.

### Q15. How do you troubleshoot slow performance in an Azure SQL Database?
**A.**
1. Use **Query Performance Insight** to identify top resource-consuming queries.
2. Check **DTU/vCore utilization** metrics — may need to scale up or optimize queries.
3. Review **Intelligent Insights** for automatically detected performance issues.
4. Check for **missing indexes** using recommendations in the portal.
5. Look at **blocking/deadlocks** via Dynamic Management Views (DMVs).

---

## 6. General / Conceptual Troubleshooting

### Q16. What is your general approach/methodology when troubleshooting an unfamiliar Azure issue?
**A.** A good structured answer:
1. **Reproduce & scope** the issue — is it one resource, one region, all users, intermittent?
2. **Check Azure Service Health** to rule out a platform-wide outage.
3. **Review Activity Log** for recent changes (deployments, policy, config changes) that correlate with the issue's start time.
4. **Check resource-specific diagnostics** ("Diagnose and solve problems" blade — available on most resources).
5. **Use Azure Monitor/Log Analytics** to correlate metrics and logs.
6. **Isolate variables** — test with a minimal reproduction (new resource, different network path, etc.).
7. **Escalate** with a Microsoft Support ticket if it's a platform-level issue, providing Correlation ID/Request ID from error messages.

### Q17. What tools do you rely on most for Azure troubleshooting?
**A.**
- **Azure Monitor** (Metrics, Logs, Alerts)
- **Log Analytics / KQL queries**
- **Network Watcher** (Connection Troubleshoot, IP Flow Verify, Packet Capture, NSG Flow Logs)
- **Azure Service Health / Resource Health**
- **Activity Log**
- **"Diagnose and solve problems" blade** (built into most resources)
- **Azure Advisor** (for proactive recommendations)
- **Serial Console / Boot Diagnostics** (for VMs)

### Q18. How do you troubleshoot a failed ARM/Bicep template deployment?
**A.**
1. Check the **deployment error details** in Deployments blade — it usually gives a specific error code/message.
2. Common issues: **naming conflicts**, **quota limits**, **invalid parameter values**, **dependency ordering** (missing `dependsOn`).
3. Use **What-If** deployment before actual deployment to preview changes.
4. Validate template syntax with `az deployment group validate`.
5. Check **resource provider registration** status if deploying a new resource type for the first time in the subscription.

### Q19. A resource is stuck in "Deleting" or "Updating" state. What do you do?
**A.**
- Wait — some operations genuinely take time (especially networking resources).
- Check **Activity Log** for the operation's actual status/error.
- Look for **dependent resources or locks** blocking the operation.
- As a last resort, raise a **support ticket** — Microsoft can sometimes force-clear a stuck provisioning state.

### Q20. How do you handle and troubleshoot Azure subscription/billing-related access issues (e.g., "Your subscription is disabled")?
**A.**
- Check the **subscription status** in the portal (disabled subscriptions are often due to unpaid invoices, spending limit reached on free/pay-as-you-go, or expired free trial).
- Verify with the **Account Administrator** on billing status.
- Re-enable via **Cost Management + Billing** blade for spending-limit-based deactivation.

---

## Quick Reference: Key Diagnostic Tools by Category

| Category | Primary Tools |
|---|---|
| VM Connectivity | Boot Diagnostics, Serial Console, Network Watcher |
| Network | NSG Flow Logs, Connection Troubleshoot, Effective Routes/Rules |
| Identity | Sign-in Logs, Audit Logs, Azure AD Connect Health |
| Storage | Storage Analytics Logs, Metrics |
| App Service/PaaS | Diagnose and Solve Problems, App Service Logs |
| General | Azure Monitor, Log Analytics (KQL), Activity Log, Service Health |

---

*Tip for the interview: When answering troubleshooting questions, structure your response as a methodology (Identify → Isolate → Diagnose → Resolve → Prevent) rather than just listing steps — interviewers often care as much about your approach as the specific answer.*
