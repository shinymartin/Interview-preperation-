# Azure Cloud & Infrastructure Engineer — Interview Preparation Guide

> A complete, self-study question bank built from the skills, certifications, and projects on Dinto Martin's CV.
> Read this end-to-end and you should be ready to handle basic, intermediate, and advanced questions in real interviews.

---

## 📊 Total Questions & Answers: **154**

### Topic Breakdown
| # | Topic | Questions |
|---|-------|-----------|
| 1 | Cloud & Azure Fundamentals | Q1 – Q12 (12) |
| 2 | Azure Administration (Compute, Storage, App Service) | Q13 – Q26 (14) |
| 3 | Azure Networking | Q27 – Q43 (17) |
| 4 | Identity & Access Management (Entra ID, RBAC) | Q44 – Q57 (14) |
| 5 | Cloud Security & Governance (Policy, Defender, Key Vault) | Q58 – Q73 (16) |
| 6 | Infrastructure as Code — Terraform | Q74 – Q92 (19) |
| 7 | CI/CD & DevOps (GitHub Actions, OIDC) | Q93 – Q104 (12) |
| 8 | Windows Server & Active Directory | Q105 – Q116 (12) |
| 9 | Monitoring, Backup & Disaster Recovery | Q117 – Q125 (9) |
| 10 | AWS Essentials | Q126 – Q132 (7) |
| 11 | Scripting (PowerShell & Bash) | Q133 – Q139 (7) |
| 12 | Solutions Architecture (AZ-305 mindset) | Q140 – Q146 (7) |
| 13 | Scenario-Based / Real-World Troubleshooting | Q147 – Q154 (8) |

> **Note on jargon:** Whenever a short form or technical term first appears, its full meaning is given in parentheses, e.g. RBAC (Role-Based Access Control).

---

## 1. Cloud & Azure Fundamentals

### Q1. What is cloud computing, and what are the main service models?
**Answer:** Cloud computing means renting computing resources (servers, storage, networking, software) over the internet from a provider instead of buying and running your own hardware. You pay only for what you use, and you can scale up or down on demand.

There are three main service models:
- **IaaS (Infrastructure as a Service):** You rent the raw infrastructure — virtual machines, storage, networks — and you manage the operating system and everything above it. Example: an Azure Virtual Machine.
- **PaaS (Platform as a Service):** You rent a managed platform where the provider handles the OS, patching, and runtime, and you just deploy your code or data. Example: Azure App Service, Azure SQL Database.
- **SaaS (Software as a Service):** You consume ready-made software over the internet and manage nothing underneath. Example: Microsoft 365, Outlook on the web.

A simple way to remember it: with IaaS you manage the most, with SaaS you manage the least.

### Q2. What is the shared responsibility model in the cloud?
**Answer:** The shared responsibility model defines who secures what — the cloud provider (Microsoft) versus you (the customer). The split depends on the service type.

- The provider is **always** responsible for the physical security of data centres, the host hardware, and the underlying network.
- **You are always** responsible for your data, your accounts/identities, and access management.
- The middle layers (operating system, network controls, applications) shift between you and the provider depending on whether you use IaaS, PaaS, or SaaS.

For example, with an IaaS VM you patch the OS; with a PaaS database, Microsoft patches it. In an interview, the key line is: *"Security of the cloud is Microsoft's job; security in the cloud is mine."*

### Q3. Explain the Azure resource hierarchy (Management Group → Subscription → Resource Group → Resource).
**Answer:** Azure organises everything into a four-level hierarchy, from broadest to narrowest:
1. **Management Group:** A container for multiple subscriptions. Used to apply governance (policies, access) across many subscriptions at once. Useful in large enterprises.
2. **Subscription:** A billing and management boundary. Resources are billed per subscription, and it acts as a scope for limits/quotas and access control.
3. **Resource Group (RG):** A logical container that holds related resources (for one application or environment). Resources in an RG usually share the same lifecycle — you can deploy or delete them together.
4. **Resource:** The actual thing you create — a VM, storage account, virtual network, etc.

Permissions and policies set at a higher level are **inherited** downward. So a policy applied at a management group flows down to all subscriptions, resource groups, and resources beneath it.

### Q4. What is an Azure Region, an Availability Zone, and a Region Pair?
**Answer:**
- **Region:** A geographic area containing one or more data centres (e.g., Central India, East US). You choose a region close to your users for low latency and to meet data-residency rules.
- **Availability Zone (AZ):** Physically separate data centres *within* a single region, each with independent power, cooling, and networking. Spreading VMs across zones protects you from a single data-centre failure.
- **Region Pair:** Each region is paired with another in the same geography (e.g., Central India ↔ South India). Microsoft staggers updates across the pair and replicates some services for disaster recovery (DR), so both don't go down at once.

**Practical use:** For high availability inside a region, use Availability Zones. For disaster recovery across regions, use the region pair.

### Q5. What is the difference between high availability, scalability, and disaster recovery?
**Answer:**
- **High Availability (HA):** Keeping a system running with minimal downtime, usually by removing single points of failure (e.g., multiple VMs behind a load balancer across zones).
- **Scalability:** The ability to handle more (or less) load by adding/removing resources. *Vertical scaling* = bigger machine; *horizontal scaling* = more machines.
- **Disaster Recovery (DR):** The plan and tooling to recover after a major failure (region outage, data loss). It's measured by **RPO (Recovery Point Objective** — how much data you can afford to lose) and **RTO (Recovery Time Objective** — how quickly you must be back up).

HA prevents small failures from causing downtime; DR deals with large-scale disasters; scalability handles changing demand.

### Q6. What is the difference between Azure Resource Manager (ARM) and the classic deployment model?
**Answer:** **ARM (Azure Resource Manager)** is the modern deployment and management layer for Azure — every request to create, update, or delete a resource goes through it. It introduced resource groups, tags, RBAC, templates (declarative JSON), and locks.

The **classic model** was the old pre-2014 system without resource groups or fine-grained RBAC. It's effectively deprecated. Today, virtually everything uses ARM. When you write Terraform or run an Azure CLI command, it ultimately calls the ARM API. Key benefits of ARM are consistent management, grouping of resources, role-based access, and repeatable template-based deployments.

### Q7. What are Azure Tags and why are they important?
**Answer:** Tags are key–value metadata labels you attach to resources, resource groups, or subscriptions (e.g., `Environment = Production`, `CostCenter = Finance`, `Owner = DintoMartin`).

They're important for:
- **Cost management:** Group and report spend by department, project, or environment.
- **Governance:** Azure Policy can require tags, so untagged resources are denied or flagged.
- **Automation & filtering:** Scripts and queries can target resources by tag (e.g., shut down all `Environment = Dev` VMs at night).

A common interview point: tags do **not** automatically inherit from resource group to resources — you must apply or enforce them (often via Azure Policy).

### Q8. What is the difference between a resource lock of type "CanNotDelete" and "ReadOnly"?
**Answer:** Resource locks protect against accidental changes and override user permissions.
- **CanNotDelete (Delete lock):** Authorized users can still read and modify the resource, but **no one can delete it** until the lock is removed.
- **ReadOnly:** Users can only read the resource — **no modifications or deletions** are allowed. This is stricter and can sometimes break operations that need write access (e.g., it may block certain management actions).

Locks are inherited by child resources. A real-world use: place a CanNotDelete lock on a production resource group so an engineer can't wipe it by mistake.

### Q9. What is the difference between CapEx and OpEx, and how does cloud change this?
**Answer:**
- **CapEx (Capital Expenditure):** Large upfront spending on assets you own — buying physical servers, data-centre hardware. You depreciate it over years.
- **OpEx (Operational Expenditure):** Ongoing pay-as-you-go spending — like a monthly cloud bill.

The cloud shifts IT spending from CapEx to OpEx: instead of buying servers, you rent them and pay monthly. This frees up capital, reduces upfront risk, and lets you scale costs with actual usage. This is a common cost-optimization talking point.

### Q10. What is an SLA and how do composite SLAs work?
**Answer:** An **SLA (Service Level Agreement)** is the provider's commitment for uptime/availability of a service, e.g., 99.9%. If they miss it, you may get service credits.

A **composite SLA** is the combined SLA when an application depends on multiple services chained together. You **multiply** the individual SLAs. For example, a web app (99.95%) talking to a database (99.99%) has a composite SLA of 0.9995 × 0.9999 ≈ 99.94% — *lower* than either component. Adding more dependencies in series lowers overall availability; adding redundancy (parallel paths) raises it. This shows interviewers you understand reliability design, not just numbers.

### Q11. What is the difference between a "lift-and-shift" migration and cloud-native?
**Answer:**
- **Lift-and-shift (rehosting):** Moving existing on-premises workloads to the cloud as-is, typically onto IaaS VMs, with little or no redesign. It's fast and low-risk but doesn't fully use cloud benefits.
- **Cloud-native:** Designing or re-architecting applications to use managed services (PaaS), auto-scaling, containers, serverless, and automation. More effort, but better cost, scalability, and resilience.

In practice, organisations often lift-and-shift first to exit a data centre quickly, then modernise gradually (the "5 Rs": Rehost, Refactor, Rearchitect, Rebuild, Replace).

### Q12. What is the Microsoft Cloud Adoption Framework (CAF), and the Well-Architected Framework (WAF)?
**Answer:**
- **CAF (Cloud Adoption Framework):** Microsoft's guidance for the *whole journey* of adopting cloud — strategy, planning, landing zones, governance, and operations. It answers "how do we move to and run on Azure as an organisation."
- **WAF (Well-Architected Framework):** Guidance for designing *individual workloads* well, across five pillars: **Reliability, Security, Cost Optimization, Operational Excellence, and Performance Efficiency.**

Interview tip: CAF = organisation-level adoption; WAF = workload-level design quality. Mentioning the five WAF pillars signals architectural maturity (relevant to your AZ-305).

---

## 2. Azure Administration (Compute, Storage, App Service)

### Q13. What are the different Azure compute options and when would you use each?
**Answer:**
- **Virtual Machines (VMs / IaaS):** Full control over the OS; best for lift-and-shift, legacy apps, or when you need specific OS configuration.
- **Azure App Service (PaaS):** Managed hosting for web apps and APIs; no OS to manage; great for standard web workloads.
- **Azure Functions (Serverless):** Run small pieces of code on events without managing servers; pay per execution; ideal for event-driven, bursty tasks.
- **Azure Kubernetes Service (AKS):** Managed Kubernetes for running containers at scale.
- **Azure Container Instances (ACI):** Run a single container quickly without orchestration overhead.

Rule of thumb: prefer PaaS/serverless for less management overhead; use VMs when you need OS-level control.

### Q14. What is a VM Scale Set (VMSS) and how does it differ from an Availability Set?
**Answer:**
- **VM Scale Set (VMSS):** A group of identical VMs that can **automatically scale** in or out based on demand (CPU, schedule, etc.). Built for elasticity and large-scale stateless workloads, and it can spread instances across availability zones.
- **Availability Set:** A logical grouping that places VMs across **fault domains** (separate hardware/power/network racks) and **update domains** (groups patched at different times) within a single data centre. It improves availability but does **not** auto-scale.

In short: Availability Set = resilience for a fixed set of VMs within one data centre; Scale Set = resilience **plus** automatic scaling.

### Q15. Explain fault domains and update domains.
**Answer:**
- **Fault Domain (FD):** A set of hardware (rack) sharing a common power source and network switch. If that rack fails, all VMs in that fault domain go down. Spreading VMs across fault domains protects against hardware failure.
- **Update Domain (UD):** A group of VMs and hardware that can be rebooted together during planned maintenance. Azure updates one update domain at a time, so not everything reboots at once.

Together, an Availability Set distributes your VMs across multiple FDs and UDs so that neither a hardware failure nor planned maintenance takes down your whole application.

### Q16. What are the Azure managed disk types and how do you choose?
**Answer:** Azure managed disks come in performance tiers:
- **Ultra Disk:** Highest performance/lowest latency; for the most demanding databases (e.g., SAP HANA, top-tier SQL).
- **Premium SSD (Solid State Drive):** High performance for production workloads and most databases.
- **Standard SSD:** Balanced cost/performance for web servers and lightly used production apps.
- **Standard HDD (Hard Disk Drive):** Cheapest; for backups, dev/test, and infrequently accessed data.

Choose based on the required **IOPS (Input/Output Operations Per Second)** and **throughput**, balanced against cost. Managed disks also handle replication and availability for you, versus unmanaged disks where you manage the storage accounts.

### Q17. What are the Azure Storage account types and redundancy options?
**Answer:** A storage account holds blobs, files, queues, and tables. Key redundancy options (how many copies and where):
- **LRS (Locally Redundant Storage):** 3 copies within a single data centre. Cheapest; protects against disk/rack failure only.
- **ZRS (Zone-Redundant Storage):** 3 copies across availability zones in one region. Protects against a data-centre (zone) failure.
- **GRS (Geo-Redundant Storage):** LRS in the primary region **plus** copies in the paired region. Protects against a full regional outage.
- **GZRS (Geo-Zone-Redundant Storage):** ZRS in primary + replication to the paired region. Highest durability.
- **RA-GRS / RA-GZRS (Read-Access variants):** Same as above but you can **read** the secondary region copy.

Choose based on how much resilience you need versus cost.

### Q18. What are Azure Blob storage access tiers?
**Answer:** Blob storage offers tiers that trade storage cost against access cost:
- **Hot:** Highest storage cost, lowest access cost — for frequently accessed data.
- **Cool:** Lower storage cost, higher access cost — for data accessed infrequently (kept ~30+ days).
- **Cold:** Even cheaper storage, higher access cost — for rarely accessed data (~90+ days).
- **Archive:** Cheapest storage, but data is offline and must be "rehydrated" (can take hours) before reading — for long-term backups/compliance.

You can use **lifecycle management policies** to automatically move blobs from Hot → Cool → Archive as they age, saving money.

### Q19. What is Azure App Service and what are deployment slots?
**Answer:** **Azure App Service** is a PaaS for hosting web apps, REST APIs, and mobile backends without managing servers. It handles patching, scaling, and load balancing.

**Deployment slots** are live, separate instances of your app (e.g., a `staging` slot alongside `production`) that share the same App Service Plan. You deploy a new version to `staging`, test it on a real environment, then **swap** it into production. The swap is near-instant and warms up the app first, enabling **zero-downtime deployments** and easy rollback (just swap back). This is a strong real-world DevOps talking point.

### Q20. What is an App Service Plan and how does it affect cost and scaling?
**Answer:** An **App Service Plan** defines the compute resources (region, VM size, instance count, pricing tier) that your App Service apps run on. Multiple apps can share one plan.

- The **tier** (Free, Shared, Basic, Standard, Premium, Isolated) determines features like custom domains, deployment slots, auto-scale, and VNet integration.
- You **scale up** by choosing a bigger tier (more CPU/RAM) and **scale out** by adding more instances.
- Cost is driven by the plan, not the number of apps — so consolidating small apps onto one plan can save money, but be careful not to over-subscribe and starve them of resources.

### Q21. How does autoscaling work in Azure?
**Answer:** Autoscaling automatically adjusts the number of instances based on rules. There are two types:
- **Scale based on a metric:** e.g., add an instance when average CPU > 70% for 10 minutes, remove one when CPU < 30%.
- **Scale based on a schedule:** e.g., run 10 instances during business hours, 2 at night.

Best practices include scaling out **before** you scale in (to avoid flapping), setting min/max instance limits, and using a cooldown period so it doesn't react to brief spikes. Autoscale applies to VM Scale Sets and App Service (Standard tier and above).

### Q22. What is the difference between scaling up and scaling out?
**Answer:**
- **Scaling up (vertical scaling):** Increasing the power of a single instance — more CPU, RAM, or a bigger VM size. Simple, but has an upper limit and usually requires a restart/downtime.
- **Scaling out (horizontal scaling):** Adding more instances and distributing load across them (e.g., behind a load balancer). No hard ceiling and supports high availability, but the app must be designed to run statelessly across multiple instances.

Cloud-native design favours scaling out because it's more elastic and resilient.

### Q23. How do you manage and optimize Azure costs?
**Answer:** Several layered techniques:
- **Right-sizing:** Match VM/disk sizes to actual usage; downsize over-provisioned resources.
- **Reservations & Savings Plans:** Commit to 1- or 3-year usage for big discounts on steady workloads.
- **Azure Hybrid Benefit:** Reuse existing Windows Server/SQL licenses to cut VM costs.
- **Auto-shutdown:** Stop dev/test VMs at night/weekends.
- **Storage tiering & lifecycle policies:** Move cold data to cheaper tiers.
- **Tags + Cost Management + Budgets/Alerts:** Track spend by project and get alerted before overspending.
- **Azure Advisor:** Gives cost (and other) recommendations automatically.

On your CV you mention cost management — be ready to give a concrete example, such as using tags and budgets to attribute and control spend.

### Q24. What is the difference between deallocating and stopping a VM?
**Answer:** This is a classic cost gotcha:
- **Stopped (from inside the OS, e.g., shutdown):** The VM is off but **still allocated** hardware, so **you keep paying** compute charges.
- **Stopped (deallocated) (from the Azure portal/CLI "Stop"):** The VM releases its compute resources, so you **stop paying for compute** (you still pay for the disks). Its dynamic public IP may change when restarted.

So to actually save money, you must **deallocate**, not just shut down inside the OS.

### Q25. How does Azure Backup work for VMs?
**Answer:** **Azure Backup** uses a **Recovery Services Vault** to store backups. For VMs:
- You define a **backup policy** (schedule + retention — e.g., daily backups kept 30 days, monthly kept a year).
- Azure takes **application-consistent snapshots** (it quiesces the app where possible so the backup is usable for restore), then transfers them to the vault.
- You can restore the whole VM, restore to a new VM, or do **file-level recovery** (mount the recovery point and copy individual files).

Backups are incremental after the first one (so only changes are stored), and the data can be encrypted and geo-redundant. This directly supports the DR work mentioned on your CV.

### Q26. What is the difference between a snapshot and a backup?
**Answer:**
- **Snapshot:** A point-in-time copy of a disk, usually stored in the same region/account. It's fast to create and good for short-term protection or cloning a disk, but it's not managed by retention policies and isn't ideal for long-term or geo-redundant protection on its own.
- **Backup:** A managed, policy-driven copy (e.g., via Azure Backup) with scheduling, retention, often geo-redundancy, and application-consistency. Better for compliance, long-term retention, and reliable recovery.

In short: snapshots are quick and manual; backups are managed, scheduled, and built for recovery at scale.

---

## 3. Azure Networking

### Q27. What is an Azure Virtual Network (VNet) and a subnet?
**Answer:** A **VNet (Virtual Network)** is your own private, isolated network in Azure, defined by an address space (e.g., `10.0.0.0/16`). Resources in a VNet can talk to each other privately.

A **subnet** is a subdivision of that address space (e.g., `10.0.1.0/24`) used to group and isolate resources — for example, a web subnet, an app subnet, and a database subnet. Subnets let you apply different security rules and route tables to different tiers, which is the basis of network segmentation. Some Azure services require their own **dedicated/delegated subnets** (e.g., Azure Firewall, Bastion, VPN Gateway).

### Q28. What is CIDR notation? Explain with an example.
**Answer:** **CIDR (Classless Inter-Domain Routing)** notation describes an IP address range using the address plus a slash and a prefix length, e.g., `10.0.0.0/24`.

The number after the slash is how many bits are "fixed" for the network portion. The remaining bits are available for hosts:
- `/24` = 256 addresses (10.0.0.0 – 10.0.0.255)
- `/16` = 65,536 addresses
- Smaller number after the slash = bigger range.

In Azure, remember the provider **reserves 5 addresses per subnet** (the first 4 and the last), so a `/24` gives you 251 usable IPs, not 256. Planning non-overlapping CIDR ranges is critical because you can't peer or connect networks with overlapping address spaces.

### Q29. What is an NSG (Network Security Group) and how do its rules work?
**Answer:** An **NSG (Network Security Group)** is a basic, stateful firewall that filters traffic to/from resources using **allow/deny rules**. You can attach it to a subnet or a NIC (network interface card).

Each rule has: priority (lower number = evaluated first), source, destination, port, protocol, direction (inbound/outbound), and action (allow/deny). Azure evaluates rules **in priority order and stops at the first match**.

Because NSGs are **stateful**, if you allow an inbound request, the response traffic is automatically allowed back out (you don't need a matching outbound rule). There are default rules (e.g., allow VNet-to-VNet, deny all inbound from the internet) that you can override with higher-priority custom rules.

### Q30. What is the difference between an NSG and Azure Firewall?
**Answer:**
- **NSG:** A free, basic, distributed **layer-3/4 (IP/port)** packet filter attached to subnets/NICs. Good for simple allow/deny between subnets. No application-layer inspection, no logging by default of full traffic, no FQDN filtering.
- **Azure Firewall:** A managed, **stateful, centralized network security service** with more advanced features: application rules (filter by **FQDN — Fully Qualified Domain Name**), network rules, NAT rules, threat intelligence, full logging, and high availability. It's a chargeable PaaS service.

Rule of thumb: use NSGs for cheap, granular subnet-level filtering, and Azure Firewall as a central choke point for inspecting and controlling all traffic — exactly what your hub-and-spoke project does.

### Q31. Explain hub-and-spoke network topology and why it's used. (Tied to your project.)
**Answer:** **Hub-and-spoke** is a network design where a central **hub** VNet hosts shared services and the **spoke** VNets host individual workloads/environments, all connected to the hub via VNet peering.

- The **hub** contains shared, security-sensitive components: Azure Firewall, VPN/ExpressRoute Gateway, Bastion, and shared DNS.
- The **spokes** are isolated workloads (e.g., prod, dev, app teams) that route through the hub.

**Why use it:**
- **Centralized security & inspection:** All egress (outbound) and inter-spoke traffic is forced through the hub's Azure Firewall, so you inspect/control it in one place.
- **Cost & management efficiency:** Shared services (firewall, gateway, Bastion) live once in the hub instead of being duplicated in every spoke.
- **Isolation:** Spokes can't talk to each other directly unless routed via the hub, reducing blast radius.

In your project, **User-Defined Routes (UDRs)** force traffic to the firewall, achieving centralized inspection.

### Q32. What is a User-Defined Route (UDR) and how did you use it? (Tied to your project.)
**Answer:** A **UDR (User-Defined Route)** is a custom routing rule you place in a **route table** attached to a subnet, overriding Azure's default system routes. Each route says "for this destination prefix, send traffic to this next hop."

In a hub-and-spoke, you create a UDR with destination `0.0.0.0/0` (all traffic) and **next hop = the Azure Firewall's private IP**. Attaching this route to the spoke subnets forces **all outbound and inter-spoke traffic through the firewall** for inspection, instead of going directly to the internet or peer. This is exactly the "centralized inspection" pattern on your CV. The next-hop type for the firewall is `VirtualAppliance`.

### Q33. What is VNet Peering and what are its key characteristics?
**Answer:** **VNet Peering** connects two virtual networks so resources in them communicate privately over Microsoft's backbone (not the public internet), as if on one network. Key points:
- **Low latency, high bandwidth** private connectivity.
- **Non-transitive by default:** If A peers with B and B peers with C, A cannot reach C automatically — you'd need a route through B (e.g., via a firewall/NVA) or a direct peering. This is why hub-and-spoke needs UDRs/firewall for spoke-to-spoke traffic.
- Networks **must not have overlapping address spaces**.
- Can be **regional** (same region) or **global** (across regions).
- You can control **gateway transit** (let spokes use the hub's VPN gateway) and traffic forwarding.

### Q34. What are Private Endpoints and Private Link? (Tied to your project.)
**Answer:** A **Private Endpoint** is a network interface with a private IP address from your VNet that connects privately to an Azure PaaS service (like Storage, SQL, Key Vault) via **Azure Private Link**. The service then becomes reachable over your private network instead of its public endpoint.

Benefits:
- Traffic to the PaaS service stays on the **private network/Microsoft backbone**, never traversing the public internet.
- You can **disable public access** on the service so it's only reachable through the private endpoint — exactly what your project did ("secured PaaS with private endpoints… disabled public access").
- It reduces the attack surface and helps meet compliance.

You typically pair private endpoints with **Private DNS zones** so the service's name resolves to the private IP (see next question).

### Q35. Why do you need Private DNS zones with Private Endpoints? (Tied to your project.)
**Answer:** When you create a private endpoint, the PaaS service must resolve to its **private IP** instead of its public IP. By default, the public DNS name (e.g., `mystorage.blob.core.windows.net`) still resolves to a public address.

A **Private DNS zone** (e.g., `privatelink.blob.core.windows.net`) linked to your VNet overrides this, so the service name resolves to the private endpoint's private IP for clients inside the network. Without it, your applications would still try to reach the public endpoint (and fail, since you disabled public access). This DNS integration is what makes private endpoints actually usable — a detail interviewers love to probe.

### Q36. What is Azure Bastion and what problem does it solve? (Tied to your project.)
**Answer:** **Azure Bastion** is a managed PaaS service that gives secure **RDP (Remote Desktop Protocol)** and **SSH (Secure Shell)** access to your VMs **directly through the Azure portal over TLS**, without exposing the VMs to the public internet.

The problem it solves: traditionally you'd put a **public IP** on a VM and open port 3389/22, which is a huge attack surface (constant brute-force attempts). With Bastion:
- VMs need **no public IP** and no open inbound RDP/SSH from the internet.
- Access happens over HTTPS (port 443) through the browser.
- It reduces exposure to port-scanning and brute-force attacks.

Your project explicitly used Bastion for admin access "with no public IPs," which is a strong security practice.

### Q37. What is the difference between Azure Load Balancer and Application Gateway?
**Answer:**
- **Azure Load Balancer:** A **layer-4 (transport — TCP/UDP)** load balancer. It distributes traffic based on IP and port, very fast, but doesn't understand HTTP. Good for general TCP/UDP workloads.
- **Application Gateway:** A **layer-7 (application — HTTP/HTTPS)** load balancer. It understands URLs and can do **path-based routing** (e.g., `/images` → one pool, `/api` → another), **SSL/TLS termination**, cookie-based session affinity, and includes an optional **WAF (Web Application Firewall)** to block common web attacks (SQL injection, XSS).

Rule of thumb: layer-4 + non-HTTP → Load Balancer; HTTP/HTTPS with smart routing and web security → Application Gateway.

### Q38. What is Azure Front Door and how does it differ from Application Gateway?
**Answer:** **Azure Front Door** is a **global**, layer-7 entry point and **CDN (Content Delivery Network)**-like service that routes user traffic to the closest/healthiest backend across regions, with SSL offload, caching, and WAF.

The key difference:
- **Application Gateway is regional** — it load-balances within one region.
- **Front Door is global** — it routes across regions and brings users in at the nearest Microsoft edge location for lower latency and global failover.

A common architecture uses Front Door at the global edge and Application Gateway within each region.

### Q39. What is a VPN Gateway and the difference between site-to-site and point-to-site?
**Answer:** A **VPN Gateway** is a service that creates encrypted tunnels between Azure and other networks over the internet.
- **Site-to-Site (S2S) VPN:** Connects an **entire on-premises network** to an Azure VNet via your on-prem VPN device. Used to extend a corporate network to the cloud.
- **Point-to-Site (P2S) VPN:** Connects an **individual device/client** (like an admin's laptop) to the Azure VNet. Used for remote workers needing access without a full site connection.

There's also **ExpressRoute** for a private, dedicated connection (next question).

### Q40. What is ExpressRoute and when would you choose it over a VPN?
**Answer:** **ExpressRoute** is a **private, dedicated connection** between your on-premises network and Azure through a connectivity provider — it does **not** go over the public internet.

Choose ExpressRoute over a VPN when you need:
- **Higher, more predictable bandwidth** and consistent low latency.
- **Better reliability and security** (private circuit, not internet-based).
- To meet compliance that prohibits public-internet transit.

The trade-off is higher cost and longer setup time. VPN is cheaper and quicker but rides the public internet with variable performance.

### Q41. What is a Service Endpoint and how does it differ from a Private Endpoint?
**Answer:** Both secure access to PaaS services, but differently:
- **Service Endpoint:** Extends your VNet identity to the Azure service over the **Microsoft backbone**, and you restrict the service's firewall to allow only that subnet. However, the service still uses its **public IP**, and access is at the subnet level.
- **Private Endpoint:** Gives the service a **private IP inside your VNet** (via Private Link). The service can be made fully private with no public access, and it works across peered networks and on-premises.

Private Endpoints are more secure and flexible (truly private IP, granular, works on-prem). Service Endpoints are simpler/free but keep the public IP and are subnet-scoped. Modern designs favour Private Endpoints.

### Q42. How does name resolution / DNS work in Azure?
**Answer:** Azure provides several DNS options:
- **Azure-provided default DNS:** Automatic name resolution within a VNet, no setup needed, but limited (no custom records, no cross-VNet names).
- **Azure Private DNS zones:** You manage custom private domains and records resolvable within linked VNets (used heavily with private endpoints).
- **Azure DNS (public):** Hosts your public domains' DNS records.
- **Custom DNS servers:** You can point a VNet to your own DNS servers (e.g., domain controllers) for hybrid name resolution.

A frequent real-world issue is DNS misconfiguration breaking private endpoint resolution — knowing the DNS layer is essential.

### Q43. What is NAT and what is an Azure NAT Gateway?
**Answer:** **NAT (Network Address Translation)** maps private IP addresses to public ones so internal resources can reach the internet without each having its own public IP.

**Azure NAT Gateway** is a managed service that provides **outbound internet connectivity** for a subnet using a stable set of public IPs. Benefits:
- VMs get internet **egress** without needing public IPs (more secure).
- It provides a **predictable outbound IP** (useful for allow-listing on third-party services).
- It scales automatically and avoids **SNAT (Source NAT) port exhaustion** issues that can occur with load balancer outbound rules.

It's the recommended modern way to give private resources controlled outbound access.

---

## 4. Identity & Access Management (Entra ID, RBAC)

### Q44. What is Microsoft Entra ID (formerly Azure AD)?
**Answer:** **Microsoft Entra ID** (previously **Azure AD — Azure Active Directory**) is Microsoft's cloud-based **identity and access management (IAM)** service. It stores users, groups, and applications, and handles **authentication** (proving who you are) and **authorization** (what you can access) for cloud resources and SaaS apps like Microsoft 365.

Important distinction: it's **not** the same as on-premises **Active Directory (AD DS — Active Directory Domain Services)**. Traditional AD uses protocols like **Kerberos/LDAP** for on-prem domains, while Entra ID is built for the cloud using **modern protocols** (OAuth 2.0, OpenID Connect, SAML) and has no concept of OUs/Group Policy in the same way. They are often synced together with Entra Connect.

### Q45. What is the difference between authentication and authorization?
**Answer:**
- **Authentication (AuthN):** Verifying **who you are** — proving your identity, e.g., entering your password and an MFA code.
- **Authorization (AuthZ):** Determining **what you're allowed to do** — your permissions, e.g., whether you can read or delete a storage account.

Order matters: you authenticate first, then the system authorizes your actions. In Azure, authentication is handled by Entra ID; authorization is handled by **RBAC** and policies.

### Q46. Explain Azure RBAC and its key components.
**Answer:** **RBAC (Role-Based Access Control)** grants access in Azure by assigning roles to identities at a scope. It has three parts (the "who, what, where"):
- **Security principal (who):** The identity — a user, group, service principal, or managed identity.
- **Role definition (what):** A collection of permissions (Actions it can do, NotActions it can't, plus data actions). Examples: Owner, Contributor, Reader, or specific roles like "Virtual Machine Contributor."
- **Scope (where):** The level it applies to — management group, subscription, resource group, or a single resource.

A **role assignment** binds these together: *this principal has this role at this scope*. Assignments **inherit downward**, so a role at the subscription applies to all its resource groups and resources. This directly maps to your CV's RBAC work.

### Q47. What's the difference between Owner, Contributor, Reader, and User Access Administrator?
**Answer:**
- **Reader:** Can **view** resources but not change anything.
- **Contributor:** Can **create and manage** resources but **cannot grant access** to others (can't manage role assignments).
- **Owner:** Full access **including** managing access (assigning roles) — the most powerful built-in role.
- **User Access Administrator:** Can **manage user access/role assignments** but not the resources themselves.

The key interview nuance: **Contributor cannot assign roles** — that ability is reserved for Owner and User Access Administrator. Follow least privilege: give Contributor, not Owner, unless someone truly needs to manage access.

### Q48. What is the principle of least privilege and how do you apply it in Azure?
**Answer:** **Least privilege** means giving each identity only the minimum permissions needed to do its job — no more. This limits the damage if an account is compromised or makes a mistake.

In Azure you apply it by:
- Assigning **specific built-in roles** (or custom roles) instead of Owner/Contributor everywhere.
- Scoping roles **narrowly** (resource group or resource, not the whole subscription).
- Using **groups** for assignments instead of individuals (easier to manage).
- Using **PIM (Privileged Identity Management)** for **just-in-time**, time-bound elevation instead of standing admin access.
- Using **managed identities** for apps instead of stored credentials.

### Q49. What are Managed Identities and what's the difference between system-assigned and user-assigned? (Tied to your project.)
**Answer:** A **Managed Identity** is an identity in Entra ID that Azure automatically manages for a resource, so the resource can authenticate to other Azure services **without storing any credentials/secrets** in code. Azure handles the credential rotation behind the scenes.

- **System-assigned:** Tied to a single resource (e.g., one VM or one policy assignment). It's created and deleted **with that resource** — a 1:1 lifecycle. Your project used a **system-assigned managed identity** for Azure Policy auto-remediation.
- **User-assigned:** A standalone identity you create once and **assign to multiple resources**. It lives independently of any single resource — good when several resources need the same identity.

This eliminates hard-coded secrets, which is a major security win and a strong talking point.

### Q50. What is a Service Principal and how does it relate to an App Registration?
**Answer:** When you register an application in Entra ID (**App Registration**), Azure creates an identity for it. The **Service Principal** is the local representation/instance of that application **in your tenant** — it's what actually gets permissions (via RBAC) and is used for authentication.

Analogy: the App Registration is the global "blueprint" of the app; the service principal is the concrete "instance" in your directory that you assign roles to. Service principals are used by automation, CI/CD pipelines, and scripts to authenticate non-interactively. Managed identities are essentially a special, Azure-managed type of service principal where you don't handle the credentials yourself.

### Q51. What is Conditional Access and give a real-world example.
**Answer:** **Conditional Access** is an Entra ID feature that enforces access policies based on **signals/conditions** ("if-then" rules). If certain conditions are met, it requires certain controls or blocks access.

Signals include: user/group, location/IP, device state, application, and sign-in risk. Controls include: require **MFA (Multi-Factor Authentication)**, require a compliant device, or block access.

Real-world examples:
- "If a user signs in from **outside the corporate network**, require MFA."
- "**Block** sign-ins from countries we don't operate in."
- "Require a managed, compliant device to access finance apps."

It's a cornerstone of **Zero Trust** ("never trust, always verify").

### Q52. What is MFA and why is it important?
**Answer:** **MFA (Multi-Factor Authentication)** requires two or more verification factors to sign in, from different categories:
- **Something you know** (password/PIN),
- **Something you have** (phone/authenticator app/token),
- **Something you are** (fingerprint/face).

It's important because passwords alone are easily stolen (phishing, breaches, reuse). With MFA, a stolen password isn't enough — the attacker also needs the second factor. Microsoft data shows MFA blocks the vast majority of account-compromise attacks, which is why it's a baseline security control and often enforced via Conditional Access. This maps directly to your SC-300 identity certification.

### Q53. What is PIM (Privileged Identity Management)?
**Answer:** **PIM (Privileged Identity Management)** provides **just-in-time (JIT)**, time-bound access to privileged roles, instead of users holding standing admin rights all the time.

How it works: a user is made **eligible** for a role rather than permanently assigned. When they need it, they **activate** the role (optionally with MFA, justification, and approval) for a limited time, after which it expires automatically.

Benefits: reduces the attack surface (fewer standing admins), provides an **audit trail** of who elevated when and why, supports approval workflows and access reviews. It's a key least-privilege control for sensitive roles like Global Administrator.

### Q54. What is the difference between security groups and Microsoft 365 groups?
**Answer:**
- **Security groups:** Used to manage **access to resources** — you assign permissions/roles (RBAC, app access, SharePoint) to the group, and members inherit them. Can contain users, devices, and other groups.
- **Microsoft 365 groups:** Used for **collaboration** — they come with shared resources like a mailbox, calendar, SharePoint site, and Teams. They're more about teamwork than access control.

For Azure RBAC assignments, you typically use **security groups** (often role-assignable groups), which makes managing access at scale much easier than assigning individuals.

### Q55. What is Single Sign-On (SSO) and how does it work?
**Answer:** **SSO (Single Sign-On)** lets users authenticate **once** and then access multiple applications without logging in again to each one.

It works through a trusted **identity provider (IdP)** like Entra ID using protocols such as **SAML (Security Assertion Markup Language)**, **OAuth 2.0**, and **OpenID Connect (OIDC)**. After the user signs in to the IdP, it issues a **token** that apps trust to grant access. Benefits: better user experience, fewer passwords (so less phishing/reuse risk), and centralized control — you can enforce MFA and Conditional Access in one place and disable a user once to cut access everywhere.

### Q56. What is the difference between OAuth 2.0, OpenID Connect, and SAML?
**Answer:**
- **OAuth 2.0:** An **authorization** framework — it lets an app get delegated access (a token) to act on a user's behalf, without sharing the password. It's about *what an app can access*, not proving identity.
- **OpenID Connect (OIDC):** An **authentication** layer built **on top of OAuth 2.0**. It adds an **ID token** that proves who the user is. Modern web/mobile apps use OIDC for login.
- **SAML (Security Assertion Markup Language):** An older XML-based standard for SSO and authentication, common in enterprise apps.

Quick memory aid: OAuth = authorization, OIDC = authentication (built on OAuth), SAML = older enterprise SSO. Your GitHub-to-Azure OIDC pipeline uses these concepts (see CI/CD section).

### Q57. What is Entra Connect (formerly Azure AD Connect) and hybrid identity?
**Answer:** **Entra Connect** is the tool that **synchronizes** on-premises Active Directory identities to Microsoft Entra ID, enabling **hybrid identity** — users have one identity that works both on-prem and in the cloud.

Sync/authentication options include:
- **Password Hash Sync (PHS):** A hash of the on-prem password hash is synced to the cloud; simplest and most resilient.
- **Pass-through Authentication (PTA):** Passwords are validated against on-prem AD in real time.
- **Federation (AD FS):** Authentication is handed off to an on-prem federation service.

This is common in enterprises (like the kind you'd support at Accenture) that aren't fully cloud-only and need seamless access across both worlds.

---

## 5. Cloud Security & Governance (Policy, Defender, Key Vault)

### Q58. What is Azure Policy and how does it work? (Tied to your project.)
**Answer:** **Azure Policy** enforces organisational rules and standards on your resources at scale. You define rules (policy definitions); Azure continuously **evaluates** resources against them and reports **compliance**.

Components:
- **Policy definition:** The rule itself (e.g., "deny resources without an Environment tag").
- **Initiative (policy set):** A bundle of related policy definitions managed together (your project bundled custom policies into a reusable initiative).
- **Assignment:** Applying a definition/initiative to a scope (management group, subscription, resource group).
- **Parameters:** Make definitions reusable (e.g., pass the list of allowed regions).

It governs the **configuration** of resources (it does *not* control identity access — that's RBAC's job). This is exactly your "Policy-as-Code governance" project.

### Q59. What are the main Azure Policy effects (Deny, Audit, Append, DeployIfNotExists, etc.)? (Tied to your project.)
**Answer:** The **effect** determines what happens when a resource matches a policy:
- **Deny:** Blocks the resource creation/update if non-compliant (e.g., deny public IPs).
- **Audit:** Allows it but **flags** it as non-compliant for reporting (no blocking).
- **Append:** Adds fields/values to a resource (e.g., add a tag).
- **Modify:** Adds, updates, or removes properties/tags (supports remediation).
- **DeployIfNotExists (DINE):** Deploys related resources/configuration if missing (e.g., deploy a diagnostic setting).
- **AuditIfNotExists (AINE):** Audits if a related resource is missing.
- **Disabled:** Turns the policy off.

Your project used **parameterized Deny/Audit effects** — meaning the same policy could be set to either block or just report, depending on the environment. DINE/Modify enable **auto-remediation** (next question).

### Q60. How does Azure Policy auto-remediation work? (Tied to your project.)
**Answer:** Auto-remediation **fixes existing non-compliant resources** automatically, rather than only blocking new ones. It uses **DeployIfNotExists** or **Modify** effects, which need a **managed identity** to make the changes.

The flow:
1. The policy (with a DINE/Modify effect) is assigned and given a **managed identity** with the right RBAC role scoped to the target.
2. For pre-existing non-compliant resources, you trigger a **remediation task**.
3. The managed identity applies the fix (e.g., adds the required tag, deploys the missing diagnostic setting).

Your project used a **system-assigned managed identity with scoped RBAC** to drive the subscription to **100% compliance** — a great concrete example of policy-as-code with self-healing.

### Q61. What is the difference between Azure Policy and RBAC?
**Answer:** People often confuse these:
- **RBAC (Role-Based Access Control):** Controls **who can do what** — *access/permissions* on resources (can this user create or delete a VM?).
- **Azure Policy:** Controls **what the resources can be** — *configuration/compliance* of resources (can a VM be created without encryption, or in a banned region?).

In short: **RBAC = who has access; Policy = what's allowed to exist and how it's configured.** They work together — RBAC says you *can* create a resource, but Policy can still deny it if it violates a rule.

### Q62. What is Microsoft Defender for Cloud and what is Secure Score?
**Answer:** **Microsoft Defender for Cloud** is Azure's **CSPM (Cloud Security Posture Management)** and **CWPP (Cloud Workload Protection Platform)** tool. It:
- Continuously **assesses** your resources against security best practices and gives recommendations.
- Provides **threat protection** for VMs, storage, SQL, containers, etc. (the paid "Defender plans").
- Helps with **regulatory compliance** dashboards.

**Secure Score** is a measurable percentage of how well your environment follows security recommendations. Each recommendation you fix raises the score. It gives you a prioritized, quantifiable way to improve security posture — a useful metric to mention when discussing hardening (which is on your CV).

### Q63. What is Azure Key Vault and what does it store? (Tied to your project.)
**Answer:** **Azure Key Vault** is a managed service for securely storing and controlling access to **secrets, keys, and certificates**:
- **Secrets:** Passwords, connection strings, API keys.
- **Keys:** Cryptographic keys for encryption/decryption and signing.
- **Certificates:** TLS/SSL certificates, with lifecycle management.

Benefits: centralized secret management, access controlled via RBAC or access policies, full **audit logging**, integration with managed identities (so apps fetch secrets without hard-coding them), soft-delete/purge protection, and the option of **HSM (Hardware Security Module)**-backed keys for higher assurance. In your hub-and-spoke project, Key Vault was secured behind a **private endpoint** with public access disabled.

### Q64. What is the difference between Key Vault access policies and RBAC for Key Vault?
**Answer:** Key Vault supports two permission models:
- **Access policies (legacy):** Permissions are set **per vault**, listing which principal can do which operations (get/list secrets, etc.). It's vault-wide and not as granular or centrally manageable.
- **Azure RBAC for Key Vault (recommended):** Uses standard Azure roles (e.g., "Key Vault Secrets User") scoped at management group/subscription/RG/vault level, integrated with the same RBAC system as the rest of Azure, with better auditing and least-privilege control.

Microsoft recommends the **RBAC model** for consistency and granularity. Knowing both — and that RBAC is preferred — shows current best-practice awareness.

### Q65. What is encryption at rest and in transit, and how does Azure provide it?
**Answer:**
- **Encryption at rest:** Protecting stored data so it's unreadable if disks are stolen. Azure encrypts storage and managed disks by default (often called **SSE — Storage Service Encryption**), using platform-managed keys, or you can use **customer-managed keys (CMK)** in Key Vault for more control.
- **Encryption in transit:** Protecting data moving across networks using **TLS (Transport Layer Security)** / HTTPS so it can't be intercepted.

For VM disks you can also use **Azure Disk Encryption (ADE)** (BitLocker/dm-crypt) or platform-managed disk encryption. Mentioning CMK vs platform-managed keys shows depth — CMK gives you control over key rotation and the ability to revoke access.

### Q66. What is the difference between platform-managed keys and customer-managed keys (CMK)?
**Answer:**
- **Platform-managed keys (PMK):** Microsoft creates, stores, and rotates the encryption keys for you. Simplest, zero effort, and secure by default.
- **Customer-managed keys (CMK):** You provide and control the keys (stored in Key Vault). You manage rotation, and you can **revoke** access to render data inaccessible — useful for strict compliance or "right to be forgotten" scenarios.

Trade-off: CMK gives control and compliance benefits but adds operational responsibility (if you lose/revoke the key, data becomes unrecoverable). Choose CMK when regulations or internal policy demand key control; otherwise PMK is fine.

### Q67. What does "hardening" a server mean and how do you do it? (Tied to your CV experience.)
**Answer:** **Hardening** means reducing a system's attack surface by configuring it securely — removing or disabling anything unnecessary and enforcing secure settings. On your CV you did "post-build hardening" and "baseline enforcement." Typical steps:
- **Disable unused services, ports, and features.**
- **Apply security baselines** (e.g., **CIS — Center for Internet Security** benchmarks, or Microsoft security baselines via Group Policy/Intune).
- **Enforce strong authentication** and least-privilege accounts; remove default/local admin where possible.
- **Patch promptly** and keep software updated.
- **Enable logging/monitoring** and endpoint protection (Defender).
- **Encrypt disks** and secure remote access (e.g., Bastion instead of public RDP).

The goal is a known-good, consistent secure configuration applied to every newly provisioned server.

### Q68. What is a security baseline and why enforce it? (Tied to your CV experience.)
**Answer:** A **security baseline** is a standardized set of secure configuration settings that every system of a given type must meet (password policies, disabled protocols, audit settings, firewall rules, etc.). Examples include Microsoft security baselines and **CIS benchmarks**.

Why enforce it:
- **Consistency:** Every server is configured the same secure way, so there's no weak link.
- **Compliance:** Demonstrates due diligence to auditors and meets regulatory standards.
- **Reduced risk:** Closes common misconfigurations attackers exploit.

You enforce baselines through Group Policy, Intune, Azure Policy guest configuration, or configuration-management tooling, and you continuously check for drift. On your CV, "baseline enforcement on newly provisioned servers" is exactly this practice.

### Q69. What is patch management and why is it critical? (Tied to your CV experience.)
**Answer:** **Patch management** is the process of identifying, testing, deploying, and verifying software/OS updates and security fixes. It's critical because unpatched systems are the most common entry point for attackers (many breaches exploit known vulnerabilities that already had fixes).

A good process:
- **Inventory** what needs patching.
- **Categorize/prioritize** (critical security patches first).
- **Test** patches in a non-production ring to avoid breaking apps.
- **Deploy in waves/maintenance windows** (often via **Azure Update Manager**, WSUS, or SCCM/Configuration Manager).
- **Verify and report** compliance.

On your CV you owned "the full patch management… lifecycle… across a 20,000+ server estate," so be ready to discuss scheduling, ring-based rollout, rollback, and reporting at scale.

### Q70. What is Zero Trust security?
**Answer:** **Zero Trust** is a security model based on **"never trust, always verify."** Instead of assuming everything inside the corporate network is safe, every access request is verified regardless of where it comes from.

Its core principles:
- **Verify explicitly:** Authenticate and authorize every request using all available signals (identity, device, location, risk).
- **Use least-privilege access:** Give just enough access, just in time (e.g., PIM).
- **Assume breach:** Segment networks, encrypt, monitor, and limit blast radius as if attackers are already inside.

In Azure this is delivered through Conditional Access, MFA, micro-segmentation, private endpoints, and continuous monitoring — many controls you already use.

### Q71. What is Microsoft Sentinel?
**Answer:** **Microsoft Sentinel** is Azure's cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automation, and Response)** platform. It:
- **Collects** logs and security data from across your environment (Azure, on-prem, other clouds, SaaS) via connectors.
- **Detects** threats using analytics rules, correlation, and machine learning.
- **Investigates** incidents with a unified view.
- **Responds/automates** using playbooks (Logic Apps) to take action automatically.

It builds on a Log Analytics workspace. If asked about security monitoring at scale or your cyber-security analyst background, Sentinel is the go-to answer for centralized threat detection and response.

### Q72. What is the difference between a vulnerability, a threat, and a risk?
**Answer:** These are core security concepts often confused:
- **Vulnerability:** A weakness or flaw in a system (e.g., an unpatched OS, an open port, a weak password).
- **Threat:** A potential danger that could exploit a vulnerability (e.g., a hacker, malware, ransomware).
- **Risk:** The likelihood and impact of a threat exploiting a vulnerability (the actual potential for harm).

A simple formula: **Risk = Threat × Vulnerability × Impact.** Security work reduces vulnerabilities (patching/hardening), defends against threats (firewalls/Defender), and thereby lowers risk. Your "Certified Cyber Security Analyst" background makes this a likely question.

### Q73. What is the CIA triad in security?
**Answer:** The **CIA triad** is the foundational model for information security, standing for:
- **Confidentiality:** Ensuring data is accessible only to authorized people (achieved via encryption, access control, RBAC).
- **Integrity:** Ensuring data is accurate and unaltered (achieved via hashing, checksums, signing, version control).
- **Availability:** Ensuring systems and data are accessible when needed (achieved via redundancy, HA, backups, DR).

Almost every security control maps to one or more of these. For example, MFA protects confidentiality, backups protect availability, and digital signatures protect integrity.

---

## 6. Infrastructure as Code — Terraform

### Q74. What is Infrastructure as Code (IaC) and what are its benefits?
**Answer:** **IaC (Infrastructure as Code)** means defining and provisioning your infrastructure (networks, VMs, policies, etc.) through **machine-readable code/config files** instead of clicking in a portal manually.

Benefits:
- **Consistency & repeatability:** The same code creates identical environments every time (no "it works in dev but not prod").
- **Version control:** Infrastructure changes are tracked in Git, with history, reviews, and rollback.
- **Automation & speed:** Deploy entire environments in minutes, integrated into CI/CD.
- **Documentation:** The code itself documents the environment.
- **Reduced human error:** No manual misconfiguration.

It's the foundation of modern DevOps and exactly what your Terraform projects demonstrate.

### Q75. What is Terraform and how does it differ from ARM/Bicep?
**Answer:** **Terraform** (by HashiCorp) is an open-source IaC tool using a declarative language (**HCL — HashiCorp Configuration Language**). You describe the desired end state, and Terraform figures out how to reach it.

Differences from Azure-native tools:
- **Terraform is multi-cloud/provider-agnostic** — the same tool can manage Azure, AWS, GCP, and many SaaS providers. Your CV's multi-cloud (Azure + AWS) context fits this well.
- **ARM templates** are Azure-only JSON, more verbose.
- **Bicep** is Azure-only too, a cleaner language that transpiles to ARM.

Terraform also has a strong **state** concept and a huge provider/module ecosystem. Choose Terraform for multi-cloud or when standardizing one tool across clouds; choose Bicep/ARM for deep, day-one Azure-only feature support.

### Q76. What is declarative vs imperative infrastructure?
**Answer:**
- **Declarative:** You describe **what** the end state should be, and the tool determines **how** to get there (Terraform, ARM, Bicep). You don't write step-by-step instructions; you describe the desired result.
- **Imperative:** You specify the exact **steps/commands** to execute in order (e.g., a Bash or PowerShell script that runs `az` commands one by one).

Declarative is generally preferred for infrastructure because it's idempotent (running it repeatedly converges to the same state) and self-documenting. Terraform is declarative.

### Q77. Explain the Terraform workflow (init, plan, apply, destroy).
**Answer:** The core Terraform commands:
- **`terraform init`:** Initializes the working directory — downloads providers/modules and configures the backend (where state is stored).
- **`terraform plan`:** Creates an **execution plan** showing what will be created, changed, or destroyed — a dry run you review before applying. Critical for safety.
- **`terraform apply`:** Executes the plan to reach the desired state (creates/updates/deletes resources).
- **`terraform destroy`:** Tears down all resources managed by that configuration.

Other useful ones: `terraform validate` (syntax/config check), `terraform fmt` (format code), `terraform import` (bring existing resources under management), and `terraform state` (manage state). Your CI pipeline runs `fmt/validate` and `plan` automatically.

### Q78. What is Terraform state and why is it important? (Tied to your project.)
**Answer:** Terraform **state** is a file (`terraform.tfstate`) that records the **mapping between your configuration and the real resources** it created, along with metadata and resource attributes. Terraform uses it to know what already exists so it can compute the difference when you change code.

Why it matters:
- Without state, Terraform wouldn't know which real resources correspond to your code.
- It enables `plan` to show accurate changes.
- It can contain **sensitive data** (secrets, IPs), so it must be protected.

This is why you use **remote state** (next question) — your CV mentions "remote state" specifically.

### Q79. What is remote state and why use it instead of local state? (Tied to your project.)
**Answer:** **Remote state** stores the state file in a shared, remote **backend** (e.g., an **Azure Storage account**) instead of on your local machine. Your CV mentions remote state in both the policy and networking projects.

Why use it:
- **Team collaboration:** Everyone works from the same single source of truth, not separate local copies.
- **State locking:** The backend (Azure Storage uses blob leases) **locks** the state during operations so two people can't apply at once and corrupt it.
- **Security:** State (which may contain secrets) is stored encrypted and access-controlled, not sitting on laptops.
- **Durability:** It's backed up/redundant, not lost if a laptop dies.
- **CI/CD friendly:** Pipelines can access shared state.

### Q80. What is state locking and why is it needed?
**Answer:** **State locking** prevents **concurrent operations** on the same Terraform state. When someone runs `apply` (or `plan` with locking), Terraform acquires a lock on the state; anyone else trying to run at the same time is blocked until it's released.

Without locking, two simultaneous applies could read/write the state at the same time and **corrupt it** or create conflicting/duplicate resources. With an Azure Storage backend, locking is handled automatically via **blob leases**. This is a key reason remote backends are used in teams.

### Q81. What are Terraform modules and why use them? (Tied to your project.)
**Answer:** A **module** is a reusable, self-contained package of Terraform configuration (a set of resources with inputs/outputs) that you can call multiple times. The root configuration is itself a module; you create **child modules** for reusable building blocks (e.g., a "network" module, a "VM" module).

Why use them:
- **Reusability (DRY — Don't Repeat Yourself):** Write once, reuse across environments/projects.
- **Consistency:** Everyone uses the same vetted building blocks.
- **Maintainability:** Fix or improve a module once and all consumers benefit.
- **Abstraction:** Hide complexity behind simple inputs.

Your CV's "reusable, semantically versioned Terraform module library" is exactly this pattern, productionized.

### Q82. What is semantic versioning and why pin module versions with Git tags? (Tied to your project.)
**Answer:** **Semantic Versioning (SemVer)** uses a `MAJOR.MINOR.PATCH` format (e.g., `1.4.2`):
- **MAJOR:** Breaking changes (incompatible).
- **MINOR:** New features, backward-compatible.
- **PATCH:** Backward-compatible bug fixes.

**Pinning** a module to a specific **Git tag/version** (e.g., `?ref=v1.4.2`) means consumers always get a known, tested version — they aren't silently broken when the module's `main` branch changes. Your CV says modules are "consumed via pinned Git tags for safe upstream changes," meaning teams upgrade deliberately by bumping the version, not by accident. This is a best practice for stability and controlled rollouts.

### Q83. What are Terraform variables, outputs, and locals?
**Answer:**
- **Variables (input variables):** Parameters you pass into a configuration/module to make it flexible (e.g., region, VM size). Declared with `variable` and supplied via `.tfvars`, CLI, or environment.
- **Outputs:** Values a module/configuration exposes after apply (e.g., a VNet ID, a public IP). Used to pass data between modules or display results.
- **Locals:** Named expressions computed within a configuration to avoid repetition (e.g., a constructed naming convention). Declared with `locals`.

Together they make configurations parameterized, composable, and clean.

### Q84. What is the difference between count and for_each in Terraform? (Tied to your CV.)
**Answer:** Both create multiple instances of a resource, but:
- **`count`:** Creates N copies indexed by number (0,1,2…). Best when instances are identical and order/number is all that matters. Downside: if you remove an item from the middle of a list, everything after it shifts index and may be recreated.
- **`for_each`:** Iterates over a **map or set**, keying each instance by a stable string. Best when instances are distinct and you want to add/remove specific ones without disturbing the others. Removing one key only affects that one.

Your CV lists `for_each`, signalling you prefer the safer, key-based iteration. Rule of thumb: use `for_each` when you have named, distinct items; use `count` for simple N-identical cases.

### Q85. What are dynamic blocks in Terraform? (Tied to your CV.)
**Answer:** A **dynamic block** generates **repeatable nested configuration blocks** inside a resource programmatically, instead of writing them out by hand. For example, a network security group may need many `security_rule` blocks — a dynamic block loops over a variable (a list/map of rules) and generates one nested block per item.

It keeps code **DRY** and data-driven: you define the rules as a variable, and the dynamic block expands them. Your CV lists dynamic blocks, indicating you write flexible, parameter-driven modules (e.g., NSG rules or firewall rules generated from input data).

### Q86. What are Terraform workspaces and when would you use them? (Tied to your CV.)
**Answer:** **Workspaces** let you maintain **multiple separate state files** from the same configuration — for example, a `dev`, `staging`, and `prod` workspace each with its own state, so the same code can manage parallel environments.

When to use: lightweight environment separation where the infrastructure is similar across environments and you just need isolated state. Caveats: workspaces don't isolate *credentials* or backends, and for very different environments many teams prefer **separate directories/state files or separate backends** instead. Knowing both the feature and its limits shows maturity. Your CV lists workspaces, so be ready to explain when they're appropriate vs. directory-per-environment.

### Q87. What is the difference between a Terraform provider and a resource?
**Answer:**
- **Provider:** A plugin that lets Terraform talk to a specific platform's API (e.g., the `azurerm` provider for Azure, `aws` for AWS). You configure it with authentication and region. It's how Terraform knows how to create things on that platform.
- **Resource:** A single infrastructure object you declare (e.g., `azurerm_virtual_network`, `azurerm_storage_account`). Resources belong to a provider.

You can also use **data sources** (read-only lookups of existing infrastructure) from a provider. Providers are the "drivers"; resources are the "things" you build with them.

### Q88. How do you manage secrets in Terraform securely?
**Answer:** Never hard-code secrets in `.tf` files or commit them to Git. Secure approaches:
- **Reference Azure Key Vault:** Use a `data` source to pull secrets at runtime, or use Key Vault references rather than storing values.
- **Environment variables / pipeline secrets:** Pass sensitive values via CI/CD secret stores (e.g., GitHub Actions secrets) rather than in code.
- **Mark variables `sensitive = true`:** So they're not shown in console output/plans.
- **Use OIDC/managed identity for auth** so there are no long-lived credentials at all (your pipeline uses OIDC).
- **Protect the state file:** Since state can contain secrets, store it in an encrypted, access-controlled remote backend.

The overarching principle: no secrets in source control.

### Q89. What is "idempotency" in the context of Terraform/IaC?
**Answer:** **Idempotency** means you can run the same operation multiple times and the **result is the same** — applying the same Terraform config repeatedly converges to the desired state without creating duplicates or making unnecessary changes.

If nothing in your code or the real environment changed, a second `terraform apply` reports "no changes." Terraform achieves this by comparing the **desired state (code)** against the **current state (state file + real resources)** and only acting on the difference. This is a major advantage of declarative IaC over imperative scripts, which may not be safely re-runnable.

### Q90. What is configuration drift and how do you handle it?
**Answer:** **Configuration drift** happens when the real infrastructure changes **outside** of Terraform — e.g., someone manually edits a setting in the portal. Now the live state no longer matches the code.

How to handle it:
- **Detect it:** Run `terraform plan` (it will show differences between code and reality). Some teams run scheduled drift-detection plans in CI.
- **Reconcile it:** Either re-apply Terraform to push the environment back to the code's desired state, or update the code (and `terraform import` if needed) to reflect intended manual changes.
- **Prevent it:** Restrict manual portal changes (RBAC/least privilege), enforce changes through pipelines, and use Azure Policy/locks to block unauthorized edits.

The goal is to keep code as the single source of truth.

### Q91. What does `terraform import` do and when do you use it?
**Answer:** **`terraform import`** brings an **existing** resource (created manually or by another process) **under Terraform management** by mapping it into the state file. You use it when you want to start managing infrastructure that wasn't originally created by Terraform — for example, adopting a manually built environment into IaC.

Important nuance: import populates the **state**, but you still must **write the matching configuration** in your `.tf` files so future plans don't try to destroy/recreate it. Newer Terraform also supports `import` blocks for a more declarative, plannable import. It's a key tool for migrating brownfield environments into IaC.

### Q92. How would you structure a Terraform project for multiple environments?
**Answer:** Common, scalable patterns:
- **Directory-per-environment:** Separate folders (`/dev`, `/staging`, `/prod`), each with its own backend/state and `.tfvars`, calling **shared modules**. This gives strong isolation and is widely recommended.
- **Reusable modules:** Keep the actual resource logic in versioned modules (like your module library) and have each environment call them with different inputs.
- **Workspaces:** For simpler cases where environments are very similar.
- **Separate state per environment** to limit blast radius (a mistake in dev can't affect prod state).
- **Remote backends + state locking** for each environment.
- **CI/CD with branch protection** so changes are reviewed and applied through pipelines.

This mirrors your projects: versioned modules + remote state + CI/CD + environment separation.

---

## 7. CI/CD & DevOps (GitHub Actions, OIDC)

### Q93. What is CI/CD and what's the difference between continuous integration, delivery, and deployment?
**Answer:** **CI/CD** automates building, testing, and releasing software/infrastructure.
- **CI (Continuous Integration):** Developers frequently merge code into a shared branch; each change triggers automated **build and tests** to catch issues early.
- **CD (Continuous Delivery):** Every change that passes CI is automatically prepared and **ready to release**, but the final push to production is a **manual approval/click**.
- **CD (Continuous Deployment):** Goes one step further — every change that passes all stages is **automatically deployed to production** with no manual gate.

For infrastructure, CI runs `fmt/validate/lint/scan/plan` and CD runs `apply` — exactly your pipeline.

### Q94. What is GitHub Actions and how does a workflow work? (Tied to your project.)
**Answer:** **GitHub Actions** is GitHub's built-in CI/CD automation platform. You define **workflows** as YAML files in `.github/workflows/`. Key concepts:
- **Workflow:** The automated process, triggered by **events** (push, pull request, schedule, manual).
- **Job:** A set of steps that run on a **runner** (a virtual machine). Jobs can run in parallel or depend on each other.
- **Step:** An individual task — either a shell command or a reusable **action**.
- **Action:** A packaged, reusable unit of automation (e.g., checkout code, set up Terraform).

Your CV's secure CI pipeline (fmt/validate, tflint, Checkov, plan, Infracost) is a sequence of jobs/steps triggered on pull requests/pushes.

### Q95. What is OIDC and why use it for GitHub-to-Azure authentication instead of secrets? (Tied to your project.)
**Answer:** **OIDC (OpenID Connect)** lets GitHub Actions authenticate to Azure using **short-lived tokens** instead of storing long-lived credentials. Your CV calls this "passwordless OIDC GitHub to Azure auth."

How it works: GitHub issues a signed OIDC token for the workflow run; Azure (via a **federated credential** on an app registration/managed identity) trusts that token and grants a **temporary** access token scoped via RBAC. No client secret is stored anywhere.

Why it's better:
- **No long-lived secrets** to leak, rotate, or accidentally commit.
- **Tokens are short-lived** and scoped to specific repos/branches/environments.
- **Stronger security & easier auditing.**

This is a modern best practice and a great security talking point.

### Q96. What is Checkov and why include it in a pipeline? (Tied to your project.)
**Answer:** **Checkov** is an open-source **static analysis / IaC security scanning** tool. It scans Terraform (and other IaC) **before deployment** for misconfigurations and policy violations — e.g., a storage account allowing public access, an NSG open to the internet, or unencrypted disks.

Why include it: it "shifts security left" — catching insecure configurations **at the pull-request stage** instead of after they're deployed. If Checkov finds a high-severity issue, the pipeline can **fail the build**, preventing insecure infrastructure from ever reaching Azure. This embeds security into the development workflow (DevSecOps).

### Q97. What is tflint and terraform fmt/validate in a pipeline? (Tied to your project.)
**Answer:** These are code-quality gates that run early in CI:
- **`terraform fmt`:** Checks/auto-formats code to the canonical style, so the codebase stays consistent and readable.
- **`terraform validate`:** Checks the configuration is **syntactically valid and internally consistent** (correct argument names, types) without contacting Azure.
- **tflint:** A **linter** that catches issues `validate` misses — provider-specific errors, deprecated syntax, invalid instance sizes, naming-convention violations, and best-practice problems.

Running these first means obvious mistakes are caught in seconds before heavier steps (plan, scan), giving fast feedback to developers.

### Q98. What is Infracost and why use it? (Tied to your project.)
**Answer:** **Infracost** estimates the **cloud cost impact of Terraform changes** and posts it in the pull request — e.g., "this change adds $120/month." 

Why use it: it gives **cost visibility before deployment**, so engineers and reviewers see the financial impact of infrastructure changes during code review, not on the next month's bill. It supports **FinOps** practices, prevents costly surprises (like accidentally provisioning an expensive VM tier), and helps teams make cost-aware decisions. It pairs naturally with the cost-management focus on your CV.

### Q99. What is branch protection and why is it important? (Tied to your project.)
**Answer:** **Branch protection** is a set of rules on important branches (like `main`) that enforce quality and security before code is merged. Typical rules:
- **Require pull requests** (no direct pushes to `main`).
- **Require status checks to pass** (the CI pipeline — fmt, validate, lint, Checkov, plan — must succeed).
- **Require code-owner/peer reviews/approvals.**
- **Prevent force-pushes and deletions.**

Why it matters: it ensures **every change is reviewed and validated** before it reaches the production branch (and thus production infrastructure). It's a key control for safe, auditable infrastructure changes and enforces the "four-eyes" principle. Your CV explicitly lists branch protection.

### Q100. What is the difference between a runner, a job, and a step in CI/CD?
**Answer:** Using GitHub Actions terminology:
- **Runner:** The machine (GitHub-hosted or self-hosted) that **executes** the work.
- **Job:** A collection of steps that runs on a single runner. Jobs can run in parallel or be chained with dependencies (`needs`).
- **Step:** A single unit of work within a job — a shell command or a reusable action.

Hierarchy: Workflow → Jobs → Steps, executed on Runners. Knowing this lets you discuss parallelizing jobs, sharing artifacts between them, and choosing self-hosted runners for private-network access.

### Q101. What are GitHub Actions secrets and environments?
**Answer:**
- **Secrets:** Encrypted variables (API keys, credentials) stored in GitHub and injected into workflows at runtime, never exposed in logs. With OIDC you minimize these, but they're still used for non-OIDC values.
- **Environments:** Named deployment targets (e.g., `dev`, `staging`, `production`) that can have their **own secrets/variables** and **protection rules** — like **required reviewers/approvals** and **wait timers** before a deployment proceeds.

Together they enable safe, gated deployments — e.g., requiring a manual approval and using environment-specific credentials before applying to production.

### Q102. What is a blue-green deployment and a canary deployment?
**Answer:** Two safe release strategies:
- **Blue-Green:** Run two identical environments — **Blue** (current/live) and **Green** (new version). Deploy and test on Green, then switch all traffic from Blue to Green at once. If something breaks, switch back instantly. (Azure App Service slot swaps are essentially this.)
- **Canary:** Roll the new version out to a **small percentage of users first** (the "canary"), monitor for errors, then gradually increase to 100% if healthy. If problems appear, you only affected a few users.

Both reduce deployment risk and downtime; blue-green favours instant full cutover/rollback, canary favours gradual exposure.

### Q103. What is GitOps?
**Answer:** **GitOps** is an operational model where **Git is the single source of truth** for both application and infrastructure state. The desired state is declared in a Git repository, and an automated process **continuously reconciles** the live environment to match what's in Git.

Key ideas: all changes go through Git (pull requests, reviews, audit history), and deployments happen automatically when Git changes (or an agent detects drift and corrects it). Benefits: full auditability, easy rollback (revert a commit), consistency, and self-healing. Your Terraform-in-Git + CI/CD approach embodies GitOps principles for infrastructure.

### Q104. How do you roll back a failed infrastructure deployment?
**Answer:** Several approaches depending on tooling:
- **Revert the Git commit** and re-run the pipeline so Terraform reconciles back to the previous known-good state (the GitOps way).
- **Use versioned/pinned modules:** Roll back to a prior module version tag.
- **`terraform plan` review:** Always review the plan before apply to avoid bad changes in the first place.
- **State backups:** Remote backends keep state history; you can restore a previous state if needed (carefully).
- **For apps:** Use blue-green/slot swap to flip back to the previous version instantly.
- **Have a tested rollback runbook** and avoid destructive changes without backups.

The best rollback strategy is prevention: plan review, automated tests/scans, and gated approvals — all of which your pipeline includes.

---

## 8. Windows Server & Active Directory

### Q105. What is Active Directory (AD) and what are its main components? (Tied to your CV.)
**Answer:** **Active Directory (AD)**, specifically **AD DS (Active Directory Domain Services)**, is Microsoft's on-premises directory service for managing users, computers, and resources in a Windows domain. It handles authentication and authorization using **Kerberos** and **LDAP (Lightweight Directory Access Protocol)**.

Main components:
- **Domain:** A logical group of objects (users, computers) sharing a database and security boundary.
- **Domain Controller (DC):** A server running AD DS that authenticates users and stores the directory.
- **Forest:** The top-level container holding one or more domain trees; the security boundary.
- **Tree:** A hierarchy of domains sharing a contiguous namespace.
- **OU (Organizational Unit):** A container to organise objects and apply Group Policy/delegation.
- **Objects:** Users, groups, computers, etc.

Your CV mentions configuring AD for "user/group lifecycle and object management."

### Q106. What is an Organizational Unit (OU) and how is it used?
**Answer:** An **OU (Organizational Unit)** is a container within a domain used to **organise objects** (users, groups, computers) hierarchically. Its two main purposes:
- **Apply Group Policy:** You link **GPOs (Group Policy Objects)** to OUs so settings apply to all objects within them (e.g., a "Finance" OU gets specific security policies).
- **Delegate administration:** You can grant specific admins control over just one OU (e.g., a helpdesk team manages only the "Sales" OU), supporting least privilege.

OUs are about administration and policy targeting — unlike security groups, which are about granting access to resources.

### Q107. What is Group Policy and how does it work?
**Answer:** **Group Policy** is the Windows mechanism for centrally managing the configuration of users and computers in an AD domain. Settings are bundled into **GPOs (Group Policy Objects)** and linked to **sites, domains, or OUs**.

How it applies — the **LSDOU** order (Local → Site → Domain → OU), with later levels overriding earlier ones (unless enforced/blocked). Computers refresh policy at startup and periodically; users at logon and periodically.

Uses: enforce password policies, lock down desktops, deploy software, map drives, configure security baselines (hardening). It's a core tool for enforcing the baselines mentioned on your CV. In the cloud, **Intune** plays a similar role.

### Q108. What is the difference between security groups and distribution groups in AD?
**Answer:**
- **Security groups:** Used to **assign permissions/access** to resources (files, folders, applications). You can also email-enable them, but their primary purpose is access control.
- **Distribution groups:** Used **only for email distribution lists** (sending email to many people at once). They **cannot** be used to assign permissions.

For controlling access to resources, you must use security groups. This distinction comes up in identity governance (your SC-300 area) and AD administration.

### Q109. What are AD group scopes (Domain Local, Global, Universal)?
**Answer:** AD group scope determines where a group can be used and what it can contain. The best-practice strategy is **AGDLP**:
- **Global groups:** Contain users from the **same domain**; used to group users by role/function (e.g., "Sales Users").
- **Domain Local groups:** Used to **grant permissions** to resources in their domain; can contain global groups from any domain.
- **Universal groups:** Used in **multi-domain forests**; can contain accounts/groups from any domain and are replicated to the global catalog.

**AGDLP** = put **A**ccounts into **G**lobal groups, put those into **D**omain **L**ocal groups, then assign **P**ermissions to the domain local group. This keeps permissions manageable at scale.

### Q110. What is the difference between Windows Server roles and features? (Tied to your CV.)
**Answer:**
- **Roles:** Major functions that define a server's primary purpose — e.g., **AD DS**, **DNS**, **DHCP**, **IIS (Internet Information Services / Web Server)**, File Server, Hyper-V.
- **Features:** Supporting capabilities that augment a role or the OS — e.g., .NET Framework, Failover Clustering, Windows Backup, BitLocker, SMB features.

Your CV mentions "role/feature configuration" during server hardening — installing only the required roles/features (and removing unneeded ones) reduces the attack surface, a core hardening practice.

### Q111. What is DNS and explain A records vs CNAME records. (Tied to your CV.)
**Answer:** **DNS (Domain Name System)** translates human-friendly names (e.g., `server01.contoso.com`) into IP addresses computers use.

- **A record (Address record):** Maps a hostname **directly to an IPv4 address** (e.g., `web01 → 10.0.0.5`). (AAAA does the same for IPv6.)
- **CNAME record (Canonical Name):** Maps a name to **another name** (an alias), not directly to an IP (e.g., `www → web01.contoso.com`). The resolver then looks up that target's A record.

Your CV mentions administering "DNS (CNAME/A-records)." Use A records to point names at IPs; use CNAMEs to alias one name to another so you can change the underlying target in one place. Other common types: MX (mail), TXT (verification/SPF), NS, PTR (reverse lookup).

### Q112. What is DHCP and how does it work?
**Answer:** **DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses and network settings (subnet mask, gateway, DNS servers) to devices, so admins don't configure each one manually.

The process is **DORA**:
1. **Discover:** Client broadcasts looking for a DHCP server.
2. **Offer:** Server offers an available IP from its scope.
3. **Request:** Client requests that offered IP.
4. **Acknowledge:** Server confirms and **leases** the IP for a set time.

Key concepts: **scope** (range of addresses), **lease** (how long an IP is held), and **reservations** (always give a specific device the same IP). It's foundational Windows Server networking knowledge.

### Q113. What is the difference between FSMO roles in Active Directory?
**Answer:** **FSMO (Flexible Single Master Operations)** roles are five special domain-controller roles that handle operations which can't be multi-master. There are **5 roles** — two forest-wide and three domain-wide:
- **Forest-wide:** **Schema Master** (controls schema changes) and **Domain Naming Master** (controls adding/removing domains).
- **Domain-wide:** **RID Master** (Relative ID — hands out pools of IDs for new objects), **PDC Emulator** (Primary Domain Controller — handles time sync, password changes, account lockouts), and **Infrastructure Master** (tracks cross-domain object references).

If a DC holding a role fails, you may need to **transfer** (graceful) or **seize** (forced) the role. This is deeper AD admin knowledge that can come up for infrastructure roles.

### Q114. How do you join a Windows Server to a domain and what happens behind the scenes?
**Answer:** You join via System Properties → Computer Name → Change → Domain (or PowerShell `Add-Computer`), supplying domain credentials with rights to add machines.

Behind the scenes:
- A **computer account** is created in AD (in the default Computers container or a specified OU).
- The machine establishes a **secure channel/trust** with a domain controller (a shared secret/machine password).
- The machine can now apply **Group Policy** and authenticate domain users via **Kerberos**.
- DNS must be correctly pointed at the domain's DNS servers, or the join fails (a very common real-world issue).

After joining, domain users can log on and centralized policies/permissions apply.

### Q115. What is the difference between a workgroup and a domain?
**Answer:**
- **Workgroup:** A **decentralized**, peer-to-peer model where each computer manages its own local accounts and security. No central authentication. Fine for very small networks (a handful of PCs).
- **Domain:** A **centralized** model managed by Active Directory/domain controllers. User accounts, security policies, and permissions are managed centrally; users log in with a single domain account that works across all domain machines.

Domains scale to thousands of machines with centralized control (SSO, Group Policy, central management) — which is why enterprises like the ones you support use domains, not workgroups.

### Q116. What tools do you use to manage Active Directory and Windows Server?
**Answer:** Common tools:
- **ADUC (Active Directory Users and Computers):** Manage users, groups, computers, OUs.
- **ADAC (Active Directory Administrative Center):** A modern interface, includes the Recycle Bin and fine-grained password policies.
- **GPMC (Group Policy Management Console):** Create, link, and manage GPOs.
- **DNS Manager / DHCP Manager:** Manage name resolution and IP leasing.
- **Server Manager:** Add/remove roles and features, manage multiple servers.
- **PowerShell (ActiveDirectory module):** Automate bulk operations (`New-ADUser`, `Get-ADUser`, etc.) — essential at the 20,000-server scale on your CV.
- **RSAT (Remote Server Administration Tools):** Manage servers remotely from a workstation.

Emphasising PowerShell automation shows you can manage at enterprise scale, not just click through GUIs.

---

## 9. Monitoring, Backup & Disaster Recovery

### Q117. What is Azure Monitor and what does it collect?
**Answer:** **Azure Monitor** is the unified platform for collecting, analysing, and acting on telemetry from your Azure (and hybrid) environment. It collects two fundamental data types:
- **Metrics:** Numeric, time-series data sampled regularly (CPU %, memory, request count). Good for real-time dashboards and alerting.
- **Logs:** Detailed, structured event records stored in a **Log Analytics workspace**, queried with **KQL (Kusto Query Language)**.

On top of these it provides **alerts**, **dashboards/workbooks**, **Application Insights** (for app performance), and **autoscale** triggers. It's the foundation of the "proactive monitoring" mentioned on your CV.

### Q118. What is Log Analytics and KQL?
**Answer:** **Log Analytics** is the service (and workspace) where Azure Monitor stores log data and where you run queries against it. A **Log Analytics workspace** is the container that aggregates logs from many sources (VMs, activity logs, security data, etc.).

**KQL (Kusto Query Language)** is the read-only query language used to search and analyse that data — e.g., filtering logs, aggregating errors, or building alert conditions. A simple example concept: "show all failed sign-ins in the last 24 hours grouped by user." KQL also powers Microsoft Sentinel and Defender. Being comfortable writing basic KQL is a strong, practical skill to mention.

### Q119. What types of alerts can you create in Azure Monitor?
**Answer:** Azure Monitor supports several alert types:
- **Metric alerts:** Fire when a metric crosses a threshold (e.g., CPU > 85% for 5 minutes).
- **Log alerts:** Fire based on a KQL query result (e.g., more than 10 errors in 5 minutes).
- **Activity log alerts:** Fire on control-plane events (e.g., a VM was deleted, a policy assignment changed).
- **Smart detection / health alerts:** Anomaly-based or service-health driven.

Alerts trigger **action groups**, which can email/SMS, call a webhook, run an Azure Function/Logic App/Automation runbook, or create an ITSM ticket. This enables the proactive response that keeps uptime high.

### Q120. Explain RPO and RTO in disaster recovery. (Tied to your CV.)
**Answer:** These two metrics define your DR requirements:
- **RPO (Recovery Point Objective):** The maximum amount of **data loss** you can tolerate, measured in time. An RPO of 1 hour means you can lose up to 1 hour of data, so you must back up/replicate at least hourly.
- **RTO (Recovery Time Objective):** The maximum acceptable **downtime** — how quickly you must be back up and running after a disaster. An RTO of 2 hours means recovery must complete within 2 hours.

Lower RPO/RTO = more resilient but more expensive (more frequent replication, hot standby). You size your backup/DR solution (Azure Backup, Site Recovery, zone/region redundancy) to meet the business's RPO/RTO. This directly supports the DR work on your CV.

### Q121. What is Azure Site Recovery (ASR) and how does it differ from Azure Backup?
**Answer:**
- **Azure Site Recovery (ASR):** A **disaster recovery** service that **continuously replicates** running VMs to another region (or Azure). In a disaster, you **fail over** to the replica with minimal downtime, then **fail back** later. It's about **business continuity** (keeping the app running).
- **Azure Backup:** A **data protection** service that stores periodic backups for **restore** after data loss/corruption. It's about **recovering data/points in time**, not instant failover.

Simple distinction: **Backup = restore lost data** (handles deletion, corruption, ransomware recovery); **Site Recovery = keep running during a regional outage** (handles downtime). Robust strategies use both.

### Q122. What is the 3-2-1 backup rule?
**Answer:** The **3-2-1 backup rule** is a best-practice guideline for resilient backups:
- **3** copies of your data (the original plus two backups),
- on **2** different types of media/storage,
- with **1** copy stored **offsite** (e.g., a different region or geo-redundant storage).

This protects against various failure modes — hardware failure, site disaster, accidental deletion, and increasingly ransomware. In Azure terms, geo-redundant Recovery Services Vaults plus immutable/locked backups help satisfy this. It's a clean, memorable answer for backup-strategy questions.

### Q123. How do you protect backups against ransomware?
**Answer:** Ransomware often tries to delete or encrypt backups, so backups need extra protection:
- **Immutability / soft delete:** Azure Backup keeps deleted recovery points for a retention period (soft delete) so they can be recovered even if "deleted" by an attacker.
- **Multi-user authorization (MUA):** Require a second approval for destructive operations (like deleting a vault or disabling protection).
- **Separate identity/permissions:** Use least privilege and separate accounts so a compromised admin can't wipe backups.
- **Geo-redundant, offsite copies** (3-2-1) so a local compromise doesn't destroy all copies.
- **Immutable vaults / WORM (Write Once Read Many)** storage where supported.
- **Alerting** on suspicious backup operations.

Showing awareness of ransomware-resilient backups is very relevant given current threat trends.

### Q124. What is the difference between a recovery point and a restore point?
**Answer:** They're often used interchangeably, but precisely:
- A **recovery point** is a specific point-in-time snapshot of your data that a backup captured (e.g., "the backup from 2 AM today"). It's *what you can recover to*.
- **Restoring** is the act of using a recovery point to bring data/systems back. A "restore point" loosely means the recovery point you choose to restore from.

The practical interview takeaway: your backup policy creates multiple **recovery points** over time (per your retention settings), and during recovery you pick the appropriate one — usually the latest clean point before an incident (e.g., before ransomware encryption).

### Q125. How do you ensure 99.9% uptime for workloads? (Tied to your CV.)
**Answer:** Your CV cites maintaining 99.9% uptime. Achieving it combines several practices:
- **Eliminate single points of failure:** Multiple instances across **Availability Zones**/Availability Sets behind a load balancer.
- **Proactive monitoring & alerting:** Azure Monitor metrics/log alerts to catch issues before they cause outages.
- **Disciplined patching with maintenance windows** and ring-based rollout to avoid update-induced outages.
- **Hardening and security** to prevent compromise-driven downtime.
- **Backup and DR** (Azure Backup, ASR) to recover quickly from failures.
- **Capacity planning/autoscaling** so load spikes don't crash services.
- **Health checks and automated remediation** (self-healing where possible).

The key narrative is *proactive* (prevent issues) plus *reactive* (fast recovery), which is exactly how you'd justify the uptime figure in an interview.

---

## 10. AWS Essentials

### Q126. What are the AWS equivalents of common Azure services?
**Answer:** Since your CV spans both clouds, mapping helps:
| Azure | AWS |
|-------|-----|
| Virtual Machine | EC2 (Elastic Compute Cloud) |
| Blob Storage | S3 (Simple Storage Service) |
| Virtual Network (VNet) | VPC (Virtual Private Cloud) |
| Entra ID | IAM (Identity and Access Management) / IAM Identity Center |
| Azure SQL Database | RDS (Relational Database Service) |
| Azure Functions | AWS Lambda |
| Resource Group | (no exact equal; tags/CloudFormation stacks) |
| Azure Monitor | CloudWatch |
| Key Vault | KMS / Secrets Manager |
| ARM/Bicep | CloudFormation |

Knowing these mappings lets you talk fluently in a multi-cloud interview and shows you understand concepts, not just one vendor's branding.

### Q127. What is an AWS VPC and how does it compare to an Azure VNet?
**Answer:** An **AWS VPC (Virtual Private Cloud)** is your isolated private network in AWS, very similar to an **Azure VNet**. Both define an address space, contain subnets, and isolate resources.

Key differences in terminology/behaviour:
- AWS subnets are tied to a **single Availability Zone**; Azure subnets can span zones within a region.
- AWS uses **Security Groups** (stateful, instance-level) and **NACLs (Network Access Control Lists** — stateless, subnet-level); Azure uses **NSGs** (stateful) at subnet/NIC level.
- Routing in AWS uses **route tables** attached to subnets (similar to Azure UDRs), with **Internet Gateways/NAT Gateways** for internet access.

The concepts map closely, which is why your hub-and-spoke and segmentation skills transfer across clouds.

### Q128. What is the difference between an AWS Security Group and a NACL?
**Answer:**
- **Security Group (SG):** **Stateful**, operates at the **instance (ENI) level**, supports **allow rules only** (everything not allowed is denied). Return traffic is automatically allowed. Closest to an Azure NSG.
- **NACL (Network Access Control List):** **Stateless**, operates at the **subnet level**, supports both **allow and deny rules**, and evaluates rules in **numbered order**. Because it's stateless, you must explicitly allow both inbound and return (outbound) traffic.

In practice, Security Groups do most of the work; NACLs add a coarse subnet-level layer (often used to block specific IPs). Understanding stateful vs stateless is the key interview point.

### Q129. What is AWS IAM and how does it differ from Azure RBAC/Entra ID?
**Answer:** **AWS IAM (Identity and Access Management)** manages identities (users, groups, roles) and permissions within an AWS account using **policies** (JSON documents granting/denying actions on resources).

Differences from Azure:
- AWS IAM combines **identity and authorization** in one service; Azure splits **identity (Entra ID)** from **authorization (RBAC)**.
- AWS uses **JSON policies** attached to users/groups/roles; Azure uses **role definitions/assignments at scopes**.
- **IAM Roles** (assumable, temporary credentials) are central in AWS — similar in spirit to Azure managed identities/service principals.

Both follow least privilege. Knowing both models shows genuine multi-cloud competence.

### Q130. What is Amazon S3 and what are its storage classes?
**Answer:** **Amazon S3 (Simple Storage Service)** is AWS's object storage (the rough equivalent of Azure Blob). Data is stored in **buckets** as objects, accessed via APIs/URLs, with very high durability.

Storage classes (cost vs access trade-offs, similar to Azure tiers):
- **S3 Standard:** Frequent access.
- **S3 Standard-IA / One Zone-IA (Infrequent Access):** Cheaper storage, retrieval fee.
- **S3 Intelligent-Tiering:** Auto-moves data between tiers based on usage.
- **S3 Glacier / Glacier Deep Archive:** Cheap long-term archive with retrieval delay (like Azure Archive tier).

A frequent interview/security point: **never make S3 buckets public unless intended** — misconfigured public buckets are a common breach cause.

### Q131. What is the AWS shared responsibility model?
**Answer:** It's the same concept as Azure's: AWS secures the **"security of the cloud"** (physical infrastructure, hardware, the global network, managed-service internals), while the customer secures **"security in the cloud"** (their data, IAM configuration, OS patching on EC2, security group rules, encryption settings).

As with Azure, the line shifts by service type — for managed services (e.g., Lambda, RDS) AWS handles more, while for EC2 you handle the OS and above. The unified takeaway across both clouds: the provider secures the platform; you secure your data, identities, and configurations.

### Q132. What is multi-cloud and what are its benefits and challenges?
**Answer:** **Multi-cloud** means using more than one cloud provider (e.g., Azure + AWS, as on your CV).

Benefits:
- **Avoid vendor lock-in** and gain negotiating leverage.
- **Best-of-breed:** Use each provider's strongest services.
- **Resilience:** Reduce dependence on a single provider's outages.
- **Meet regional/regulatory needs.**

Challenges:
- **Complexity:** Different APIs, tooling, and skill sets.
- **Consistent security/governance** across clouds is harder.
- **Cost visibility** across providers.
- **Networking** between clouds.

This is where **Terraform shines** — one IaC tool to manage multiple clouds consistently, which is exactly why your Terraform + multi-cloud combination is valuable.

---

## 11. Scripting (PowerShell & Bash)

### Q133. When would you use PowerShell vs Bash vs Azure CLI?
**Answer:**
- **PowerShell (with the Az module):** Best on Windows and for Windows Server/Active Directory administration, object-oriented automation, and complex logic. It returns **objects**, not just text, which makes filtering/processing powerful.
- **Bash:** Best on Linux/macOS and for shell scripting in those environments; common in Linux automation and many CI runners.
- **Azure CLI (`az`):** Cross-platform, concise commands for Azure that work well in both Bash and PowerShell and in pipelines.

In practice you choose based on the platform and team standards. Your CV lists PowerShell and Bash, signalling you automate across both Windows and Linux. A good answer notes that PowerShell's object pipeline is a key differentiator.

### Q134. What is the difference between a script and a function, and why use functions?
**Answer:**
- A **script** is a file of commands run top to bottom to accomplish a task.
- A **function** is a reusable, named block of code (inside a script or module) that performs a specific job and can take parameters and return values.

Why use functions:
- **Reusability:** Call the same logic many times without duplicating code.
- **Readability & maintainability:** Break a big task into named, understandable pieces.
- **Testing:** Test small units independently.
- **Parameterization:** Make behaviour flexible via inputs.

Using functions (and modules) is the difference between a quick hack and maintainable automation at enterprise scale.

### Q135. How do you handle errors in PowerShell?
**Answer:** Key error-handling techniques:
- **`try { } catch { } finally { }`:** Wrap risky code; `catch` handles the error, `finally` always runs (cleanup).
- **`-ErrorAction Stop`:** Force a non-terminating error to become terminating so `catch` can handle it.
- **`$ErrorActionPreference`:** Set default error behaviour for a script (e.g., `Stop`).
- **`$Error` automatic variable:** Inspect recent errors.
- **`Write-Error` / proper exit codes:** Signal failure to callers/pipelines.

Robust error handling matters especially in automation that runs unattended (e.g., scheduled tasks or CI), so failures are caught, logged, and don't silently corrupt things.

### Q136. What is idempotency in scripting and why does it matter?
**Answer:** An **idempotent** script produces the **same result no matter how many times it's run**. For example, a script that "ensures a folder exists" should check first and only create it if missing — running it ten times leaves exactly one folder, with no errors.

Why it matters: automation often re-runs (retries, scheduled jobs, pipeline re-runs). Non-idempotent scripts can create duplicates, throw errors on the second run, or cause inconsistent state. Writing idempotent scripts (check-then-act, "ensure" logic) makes automation safe and reliable — the same principle that makes declarative IaC like Terraform powerful.

### Q137. How would you automate creating 100 AD users from a CSV file? (Practical.)
**Answer:** The standard PowerShell approach:
1. **Prepare a CSV** with columns like Name, SamAccountName, OU, Department, Password.
2. **Import it** with `Import-Csv`.
3. **Loop** over each row (`ForEach-Object`) and call **`New-ADUser`** with the row's values, setting the target OU, attributes, and an initial password (with "change at next logon").
4. **Add error handling** (`try/catch`) so one bad row doesn't stop the whole batch, and **log** successes/failures.
5. Optionally **add users to groups** with `Add-ADGroupMember`.

This demonstrates exactly the "user/group lifecycle" automation on your CV — bulk operations done reliably with PowerShell instead of manual clicks, which is essential at enterprise scale.

### Q138. What are variables, loops, and conditionals — and give a real automation example?
**Answer:** These are the building blocks of any script:
- **Variables:** Store values (e.g., a server name, a threshold).
- **Loops:** Repeat actions over a collection (`foreach`, `for`, `while`).
- **Conditionals:** Make decisions (`if/else`, `switch`).

**Example:** A maintenance script could loop over a list of servers, **check each one's free disk space** (variable), and **if** free space is below 10% (conditional), trigger a cleanup and send an alert. This kind of "check many things, act on the ones that need it" pattern is the backbone of operational automation — monitoring, patching reports, cleanup jobs — directly relevant to the infra-managed-service work on your CV.

### Q139. How do you securely handle credentials in scripts?
**Answer:** Never hard-code passwords/keys in plain text. Secure methods:
- **Managed identities:** Best for Azure — the script authenticates with no stored secret at all.
- **Azure Key Vault:** Retrieve secrets at runtime via the Key Vault module/CLI.
- **PowerShell `SecureString` / `Get-Credential`:** Prompt for or store credentials securely rather than as plain text.
- **Credential managers / secret stores:** Use the OS or a vault, not source files.
- **Pipeline secret stores** (GitHub Actions secrets) for CI.
- **Never commit secrets to Git;** use `.gitignore` and secret-scanning.

The principle mirrors IaC: keep secrets out of code and out of source control, and prefer credential-less authentication (managed identity/OIDC) wherever possible.

---

## 12. Solutions Architecture (AZ-305 mindset)

### Q140. How do you design a highly available, multi-tier web application in Azure?
**Answer:** A standard resilient design:
- **Front end / global:** **Azure Front Door** or Traffic Manager for global routing and failover, with **WAF** for protection.
- **Web/app tier:** **App Service** (or VM Scale Sets/AKS) spread across **Availability Zones**, behind a load balancer/Application Gateway, with **autoscaling**.
- **Data tier:** A managed database (Azure SQL) with **zone-redundancy** and **geo-replication** for DR.
- **Networking:** Segmented subnets, NSGs, **private endpoints** for PaaS, and a hub-and-spoke for centralized security.
- **Security:** Managed identities, Key Vault for secrets, RBAC least privilege, Defender for Cloud.
- **Observability:** Azure Monitor + Application Insights + alerts.
- **DR:** Backups + multi-region strategy sized to RPO/RTO.

The narrative weaves together HA (zones), scalability (autoscale), security (your specialty), and DR — exactly the AZ-305 architect mindset.

### Q141. How do you decide between IaaS, PaaS, and SaaS for a workload?
**Answer:** Decide by balancing **control vs management overhead**:
- Choose **SaaS** when a ready-made product meets the need (e.g., email) — least effort.
- Choose **PaaS** when you're building custom apps but want to avoid managing servers/OS (e.g., App Service, Azure SQL) — good default for cloud-native.
- Choose **IaaS** when you need full OS control, have specific software/compliance requirements, or are doing lift-and-shift of legacy apps.

Other factors: existing skills, compliance/data-residency, cost model, migration effort, and how much you want to operate vs build. The architect's instinct is to **prefer the most managed option that still meets requirements** (less to operate, more reliability).

### Q142. How do you design for security in an Azure landing zone?
**Answer:** A **landing zone** is a pre-configured, governed environment that new workloads "land" in. Security design includes:
- **Management groups + Azure Policy** to enforce guardrails (allowed regions, required tags, deny public IPs) across all subscriptions — your policy-as-code work.
- **RBAC with least privilege** and **PIM** for privileged access.
- **Hub-and-spoke networking** with centralized firewall, NSGs, private endpoints, and Bastion — your networking project.
- **Centralized identity** (Entra ID), Conditional Access, MFA.
- **Key Vault** for secrets; encryption at rest/in transit.
- **Defender for Cloud + Sentinel** for posture and threat detection.
- **Centralized logging** to a Log Analytics workspace.

Your CV projects basically *are* the security building blocks of a landing zone, which is a strong story to tell.

### Q143. How do you choose the right Azure data store for a workload?
**Answer:** Match the store to the data and access pattern:
- **Relational/structured + transactions:** Azure SQL Database / SQL Managed Instance.
- **Key-value / document / global, massive scale:** Azure Cosmos DB (NoSQL, multi-model, low latency).
- **Unstructured objects (files, images, backups):** Blob Storage.
- **File shares (lift-and-shift, SMB/NFS):** Azure Files.
- **Analytics / big data warehouse:** Azure Synapse / Data Lake.
- **Caching:** Azure Cache for Redis.
- **Message/queue:** Service Bus / Storage Queues / Event Hubs.

Decision factors: structured vs unstructured, consistency needs, scale, latency, query patterns, and cost. Architects pick the **purpose-built** store rather than forcing everything into one database.

### Q144. What is the difference between availability, scalability, and resiliency in design?
**Answer:**
- **Availability:** The proportion of time the system is up and working (uptime %). Improved by redundancy and removing single points of failure.
- **Scalability:** The ability to handle growth in load gracefully (scale up/out). About performance under increasing demand.
- **Resiliency:** The ability to **recover from failures** and keep functioning (or degrade gracefully). About handling and bouncing back from faults (retries, circuit breakers, failover, DR).

They're related but distinct: a system can be scalable yet not resilient (scales well but crashes on a dependency failure). Good architecture designs for all three, plus security and cost (the WAF pillars).

### Q145. How do you approach cost optimization at an architectural level?
**Answer:** Beyond tactical savings, architectural cost optimization includes:
- **Choosing the right service model:** PaaS/serverless can be cheaper to operate than VMs; pay-per-use (Functions) suits spiky workloads.
- **Right-sizing and autoscaling:** Match capacity to demand instead of over-provisioning.
- **Reservations/Savings Plans/Hybrid Benefit** for steady workloads.
- **Storage tiering and lifecycle policies.**
- **Designing for elasticity** so you only pay for what you use (scale to zero where possible).
- **Governance:** Tags, budgets, alerts, and tools like **Infracost** in CI to catch costs *before* deploy.
- **Regular reviews** with Azure Advisor and Cost Management.

The architect's mindset treats cost as a **design constraint** (a WAF pillar), not an afterthought — which ties to the Infracost/cost-management work on your CV.

### Q146. What is a well-architected review and what are the five pillars?
**Answer:** A **Well-Architected Review** assesses a workload against Microsoft's **Well-Architected Framework (WAF)** to find risks and improvement areas. The **five pillars**:
1. **Reliability:** Recovering from failures, meeting availability targets.
2. **Security:** Protecting data and systems (your strongest area).
3. **Cost Optimization:** Managing and reducing spend.
4. **Operational Excellence:** Monitoring, automation, and processes to run efficiently.
5. **Performance Efficiency:** Using resources efficiently and scaling to demand.

A review scores the workload across these and produces prioritized recommendations. Being able to name and apply the five pillars demonstrates architect-level (AZ-305) thinking and gives a structured way to evaluate any design.

---

## 13. Scenario-Based / Real-World Troubleshooting

### Q147. Scenario: A VM is unreachable via RDP. How do you troubleshoot?
**Answer:** Work methodically from the outside in:
1. **Check the VM state:** Is it running (not stopped/deallocated)? Check the portal and **boot diagnostics/serial console**.
2. **Network rules:** Is **port 3389** allowed by the **NSG** (subnet and NIC)? Is there a UDR/firewall blocking it? Is it behind Azure Firewall expecting Bastion?
3. **Public connectivity:** Does it have a public IP or are you using Bastion/VPN? Use **Network Watcher → Connection Troubleshoot / IP Flow Verify**.
4. **OS level:** Is the Windows Firewall blocking RDP? Is the RDP service running? (Check via serial console / Run Command.)
5. **Resource health & recent changes:** Check Azure **Resource Health**, recent deployments, or policy changes that may have altered config.
6. **Credentials/account:** Lockout, expired password, or AD/DC reachability issues.

A strong answer shows a **layered, elimination-based** method and names the right Azure tools (Network Watcher, serial console, Bastion).

### Q148. Scenario: Your Terraform apply fails with a state lock error. What do you do?
**Answer:** This means the state is **locked** (someone/something else is running, or a previous run crashed without releasing the lock).
1. **Check for a running operation:** Confirm no teammate/pipeline is mid-apply. Coordinate before touching the lock.
2. **Wait and retry:** If another run is genuinely in progress, let it finish.
3. **Inspect the lock info:** Terraform's error shows who/when acquired it (the backend, lock ID).
4. **If it's a stale lock** (a crashed run), carefully use **`terraform force-unlock <LOCK_ID>`** — only after confirming nothing is actually running, since force-unlocking during a live apply can corrupt state.
5. **Prevent recurrence:** Ensure proper remote backend config and avoid concurrent pipeline runs (concurrency controls in CI).

The key is to **verify no active operation** before force-unlocking — showing you understand the risk to state integrity.

### Q149. Scenario: A storage account behind a private endpoint suddenly can't be reached by the app. What's likely wrong?
**Answer:** Given your project experience, the usual culprit is **DNS or network configuration**:
1. **DNS resolution:** Is the **Private DNS zone** (e.g., `privatelink.blob.core.windows.net`) **linked to the VNet**, and does the storage name resolve to the **private IP**? If it resolves to the public IP, traffic fails because public access is disabled. This is the #1 cause.
2. **Private endpoint health:** Is the private endpoint provisioned and **approved**, with the correct subresource (blob/file)?
3. **Network path:** NSG/UDR/firewall rules — does a route or firewall rule block the private endpoint subnet? Did a recent UDR change reroute traffic incorrectly?
4. **Public access setting:** Confirm the account's firewall/public-access settings match the intended private-only design.
5. **Use Network Watcher** to trace the path.

Leading with the **Private DNS zone link** shows real hands-on depth with the exact pattern on your CV.

### Q150. Scenario: Azure Policy is blocking a legitimate deployment. How do you handle it?
**Answer:** A balanced, governance-aware response:
1. **Read the policy error:** It names the policy and the reason (e.g., "denied: region not allowed" or "missing required tag").
2. **Fix the resource if it's genuinely non-compliant:** e.g., add the required tag or use an allowed region/SKU — usually the deployment *should* comply.
3. **If the requirement is legitimate but the policy is too strict:** Work through the **change process** — adjust the policy **parameters** (e.g., add the region to the allowed list) or scope, rather than disabling governance.
4. **Use exemptions sparingly:** Azure Policy supports **exemptions** for specific resources/scopes with justification and expiry — appropriate for true one-offs, not as a workaround.
5. **Avoid simply turning policies off**, which undermines compliance.

This shows you respect guardrails (your policy-as-code work) while solving the immediate need through proper change control, not shortcuts.

### Q151. Scenario: You're seeing intermittent high latency on an app. How do you investigate?
**Answer:** Systematic performance triage:
1. **Gather data:** Use **Application Insights** (request/dependency duration) and **Azure Monitor metrics** to see *where* latency occurs — app tier, database, or network.
2. **Identify the pattern:** Is it tied to load spikes (scale issue), specific dependencies (slow DB/API), or a region/network path?
3. **Check resources:** CPU/memory/disk on compute; **DTU/throughput** on the database; throttling (HTTP 429) on services.
4. **Network:** Latency between tiers, cross-region calls, DNS, or a misrouted UDR sending traffic the long way.
5. **Scaling:** Is autoscale keeping up? Consider scaling out, adding caching (Redis), or a CDN/Front Door for global users.
6. **Recent changes:** Correlate with deployments or config changes.

A good answer is **data-driven** (measure first), isolates the layer, then fixes the root cause (scaling, caching, query tuning, routing).

### Q152. Scenario: A patch caused production servers to fail. What's your response and prevention plan?
**Answer:** Drawing on your patch-management experience:

**Immediate response:**
1. **Assess impact and communicate** (incident process, stakeholders).
2. **Roll back** the problematic patch or restore from backup/snapshot to the last known-good state to restore service (meeting RTO).
3. **Stabilize and verify** services are healthy.

**Root cause & prevention:**
- **Ring-based deployment:** Patch a small **pilot/canary ring** first, validate, then expand in waves — never all 20,000 servers at once.
- **Test in non-production** before production.
- **Maintenance windows** and change approval.
- **Backups/snapshots before patching** for fast rollback.
- **Post-incident review** to capture lessons and update the runbook.

This shows mature operations: contain, recover, then prevent recurrence with ring-based rollout — exactly the discipline needed at enterprise scale.

### Q153. Scenario: A developer needs access to a resource group but you must follow least privilege. How do you grant it?
**Answer:** A clean least-privilege workflow:
1. **Clarify the need:** What exactly must they do (read, deploy, manage a specific resource type)?
2. **Pick the narrowest role:** Assign a **specific built-in role** (e.g., "Contributor" only if they must create/manage resources, or a narrower role like "Virtual Machine Contributor"/"Reader" if that suffices) — **not Owner**.
3. **Scope it tightly:** Assign at the **resource group** (or even a single resource), not the subscription.
4. **Use a group:** Assign the role to a **security group** and add the user, so access is manageable and auditable.
5. **Time-bound if privileged:** For elevated needs, use **PIM** for just-in-time access with approval and expiry.
6. **Document and review:** Record the justification and include it in periodic **access reviews**.

This demonstrates the RBAC least-privilege discipline central to your identity/security background.

### Q154. Scenario: Costs spiked unexpectedly this month. How do you find and fix the cause?
**Answer:** A structured FinOps investigation:
1. **Analyze in Cost Management:** Use **cost analysis** to break spend down by **resource, resource group, service, tag, and time** to pinpoint what jumped and when.
2. **Common culprits:** A large VM/disk left running, an expensive tier deployed by mistake, egress/bandwidth charges, orphaned resources (unattached disks, idle public IPs, old snapshots), or autoscale running hot.
3. **Use tags** (Environment/Owner/CostCenter) to attribute the spike to a team/project quickly — which is why tagging governance matters.
4. **Fix it:** Right-size or deallocate, remove orphaned resources, apply lifecycle policies, and correct any misconfiguration.
5. **Prevent recurrence:** Set **budgets and alerts**, enforce tagging via **Azure Policy**, add **Infracost** to CI to catch costly changes pre-merge, and review **Azure Advisor** recommendations.

This ties together tagging, policy, budgets, and Infracost — all on your CV — into a real cost-control story.

---

## ✅ Final Preparation Tips

- **Tie answers to your CV:** When a topic matches your projects (hub-and-spoke, policy-as-code, module library/CI-CD), say *"In my project I did exactly this…"* — concrete experience beats theory.
- **Explain the "why," not just the "what":** Interviewers probe reasoning. For every choice (e.g., private endpoint vs service endpoint), be ready with the trade-off.
- **Use the STAR method for behavioural/scenario questions:** **S**ituation, **T**ask, **A**ction, **R**esult — especially for your Accenture experience (uptime, patching at scale).
- **Know your certifications' core themes:** AZ-104 (admin), AZ-305 (architecture/trade-offs), AZ-500 (security), SC-300 (identity) — interviewers may target the cert on your CV.
- **Practice drawing architectures:** Be ready to sketch hub-and-spoke or an HA web app on a whiteboard.
- **Quantify impact:** "100% policy compliance," "99.9% uptime," "20,000+ servers" — lead with measurable results.

> **Good luck, Dinto!** Work through these 154 questions until you can explain each in your own words, and you'll be well prepared for basic through advanced Azure Cloud & Infrastructure interviews.
