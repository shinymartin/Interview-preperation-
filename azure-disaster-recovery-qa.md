# Azure Disaster Recovery — Interview & Study Q&A

A comprehensive question-and-answer bank covering Azure Disaster Recovery (DR), Business Continuity, Azure Site Recovery (ASR), Azure Backup, and related high-availability concepts.

---

## 1. Fundamentals & Concepts

**1. What is Disaster Recovery (DR), and how does it differ from High Availability (HA)?**
DR is the set of policies, tools, and procedures used to recover or continue IT operations after a disruptive event such as a regional outage, cyberattack, or hardware failure — typically involving failover to a secondary site/region. HA, by contrast, focuses on minimizing downtime *within* a single site/region using redundancy (multiple VMs, zones, load balancers) so the application keeps running through smaller-scale failures. HA handles component-level failures; DR handles site/region-level catastrophic failures.

**2. What is Business Continuity and Disaster Recovery (BCDR)?**
BCDR is the umbrella discipline combining Business Continuity Planning (keeping critical business functions running during and after a disruption) with Disaster Recovery (the technical recovery of IT systems and data). It includes risk assessment, business impact analysis, recovery strategies, communication plans, and regular testing.

**3. Define RTO (Recovery Time Objective) and RPO (Recovery Point Objective). How are they different?**
RTO is the maximum acceptable time to restore a system/application after a disaster (a measure of downtime tolerance). RPO is the maximum acceptable amount of data loss measured in time (how much data, since the last backup/replication point, you can afford to lose). RTO answers "how long can we be down?"; RPO answers "how much data can we lose?"

**4. How do you determine acceptable RTO/RPO values for a business-critical application?**
Conduct a Business Impact Analysis (BIA) with stakeholders to quantify the cost of downtime and data loss per hour/minute, classify applications by criticality (tier 0/1/2/3), and align RTO/RPO targets with budget and technical feasibility — tighter RTO/RPO generally means higher cost (e.g., synchronous replication, active-active architectures).

**5. What is the difference between backup and disaster recovery?**
Backup is a copy of data taken at a point in time, primarily used to recover from data loss, corruption, or accidental deletion — it protects data. DR is a broader strategy to restore entire systems/applications and infrastructure after a major outage, often involving replication and failover of compute, network, and data together — it protects business operations. Backup is typically a *component* of a DR strategy, not a replacement for it.

**6. What are the key components of a DR strategy?**
Risk assessment and BIA, defined RTO/RPO targets, replication/backup mechanisms, a documented recovery plan/runbook, network and DNS failover design, regular testing (DR drills), monitoring/alerting, roles and communication plans, and post-failover validation and failback procedures.

**7. What is a DR drill/failover test, and why is it important?**
A DR drill is a planned, controlled exercise (often a "test failover" in an isolated network) that validates whether systems can actually be recovered within the target RTO/RPO. It's important because untested DR plans frequently fail in a real disaster due to configuration drift, missing dependencies, or stale runbooks — testing surfaces these gaps safely.

**8. What is the difference between failover and failback?**
Failover is the process of switching operations from the primary (failed/impacted) site to the secondary (DR) site. Failback is the reverse — switching operations back to the original primary site once it has been restored and re-synchronized, typically after re-protecting and resyncing data.

**9. What is meant by "Recovery Plan" in Azure Site Recovery?**
A Recovery Plan in ASR is an orchestration construct that groups multiple VMs (often across application tiers) and defines the order, grouping, and automation (scripts, Azure Automation runbooks, manual approval steps) used to fail them over together — e.g., start database VMs first, wait, then start application VMs, then web VMs.

**10. What is the shared responsibility model in the context of DR on Azure?**
Microsoft is responsible for the resiliency of the underlying platform (physical datacenters, regions, the availability of Azure services per SLA). The customer is responsible for designing, configuring, testing, and maintaining their own application-level DR strategy — Azure gives you the tools (ASR, Backup, Traffic Manager, geo-redundant storage) but doesn't automatically make your application DR-ready.

**11. What is a Service Level Agreement (SLA), and how does Azure define SLAs for availability and DR-related services?**
An SLA is a contractual commitment from Microsoft about a service's uptime/availability, with financial credits if not met. Azure publishes per-service SLAs (e.g., 99.9%–99.99% depending on configuration, such as VM SLA improving with Availability Zones or Availability Sets). SLAs generally apply to the *platform's* availability, not to whether your specific DR architecture meets your business RTO/RPO — that remains the customer's responsibility.

**12. What is the difference between site-level, region-level, and zone-level disaster recovery?**
Site-level DR typically refers to on-premises datacenter failure recovering to the cloud. Zone-level DR/resilience protects against failure of a single Availability Zone (a physically separate datacenter within a region) by spreading resources across zones. Region-level DR protects against the loss of an entire Azure region by replicating/failing over to a geographically distant paired (or other) region.

---

## 2. Azure Regions, Availability Zones & Resilience

**13. What is an Azure Region, Region Pair, and Availability Zone?**
A Region is a set of datacenters deployed within a defined geography. A Region Pair is two regions within the same geography (usually 300+ miles apart) that Azure links for sequential update rollout and prioritized recovery. An Availability Zone is a physically separate location within a region, with independent power, cooling, and networking, used to protect against datacenter-level failures.

**14. How does Azure Region Pairing help with disaster recovery?**
Paired regions are designed so that platform updates are rolled out to only one region of a pair at a time, and in the rare event of a broad regional outage, Microsoft prioritizes restoring one region from each pair first. Some geo-redundant services (like GRS storage) replicate automatically to the paired region, simplifying DR architecture.

**15. What is the difference between Availability Sets and Availability Zones?**
Availability Sets group VMs within a *single datacenter* into fault domains (separate hardware racks/power) and update domains (separate maintenance groups), protecting against hardware/host failures. Availability Zones span *physically separate datacenters within a region*, protecting against a full datacenter failure (power, cooling, network) — a stronger level of resilience than Availability Sets.

**16. How does deploying resources across Availability Zones improve resilience compared to a single zone?**
If one zone experiences an outage (power, cooling, network), workloads in the other zones continue running, since zones have independent infrastructure. This provides higher availability (up to 99.99% SLA for zone-redundant VM configurations) without requiring a full cross-region failover.

**17. What happens during a region-wide outage, and how does Azure handle it?**
Services deployed only within that region become unavailable until Microsoft restores the region. Customers who architected geo-redundancy (GRS storage, ASR replication, Traffic Manager/Front Door routing, multi-region deployments) can fail over to a secondary region to maintain availability; those without cross-region redundancy experience downtime until the region recovers.

**18. What is an Availability Zone vs. paired-region failover strategy — when would you use each?**
Use Availability Zones when you need protection against datacenter-level failures with low latency and near-real-time synchronous replication (suitable for most HA needs). Use a paired-region failover strategy when you need protection against a full regional disaster, compliance/data-residency separation, or extreme resilience requirements — accepting higher latency and typically asynchronous replication.

**19. Which Azure services are zone-redundant, zonal, or region-redundant by default?**
Zone-redundant by default (many PaaS services replicate across zones automatically): Zone-Redundant Storage (ZRS), Azure SQL Database Zone-Redundant configuration, Standard Load Balancer, Application Gateway v2. Zonal (you pin to a specific zone): Virtual Machines, Managed Disks, zonal Public IPs. Region-redundant (span across the paired region): Geo-Redundant Storage (GRS/GZRS), Azure SQL geo-replication, Cosmos DB multi-region.

**20. What is Azure's definition of a "Recovery Region" and how do you choose one?**
A Recovery Region (or target region) is the secondary Azure region designated to host failed-over resources during a DR event. Selection criteria include: proximity/latency to users, data residency and compliance requirements, service/SKU availability parity with the primary region, cost, and (commonly) whether it is the Azure "paired region" for the primary.

---

## 3. Azure Site Recovery (ASR)

**21. What is Azure Site Recovery (ASR), and what workloads does it support?**
ASR is Azure's native DR-as-a-Service offering that orchestrates and manages replication, failover, and recovery of workloads. It supports Azure VM-to-Azure VM (cross-region), on-premises VMware VMs, on-premises Hyper-V VMs, and physical servers replicating to Azure, as well as replication between two on-premises sites.

**22. What is the architecture of Azure Site Recovery for Azure VM to Azure VM replication?**
An Azure VM extension installed on the source VM continuously replicates disk changes to cache storage in the source region, then to Azure Storage/managed disks in the target region via the Site Recovery service, managed through a Recovery Services Vault. No on-premises components (configuration/process servers) are needed for this scenario since it's Azure-native.

**23. How does ASR replicate on-premises VMware/Hyper-V VMs to Azure?**
For VMware: a Configuration Server (deployed as an OVA appliance) coordinates communication, a Process Server handles caching/compression/encryption of replicated data, and Mobility Service agents installed on each VM capture and forward disk writes. For Hyper-V: the Hyper-V Replica Provider and Recovery Services provider (installed on hosts/VMM) handle replication directly to Azure.

**24. What are the prerequisites for setting up ASR for Azure VMs?**
A Recovery Services Vault in the target region, a defined replication policy, network connectivity/mapping planned for the target region (VNet, subnets, NSGs), sufficient quota/capacity in the target region, supported OS and disk configuration on the source VMs, and appropriate RBAC permissions (e.g., Site Recovery Contributor).

**25. What is a Recovery Services Vault, and what role does it play in ASR?**
A Recovery Services Vault is a management entity/storage construct in Azure that stores replication data, backup data, recovery points, and configuration metadata. It's the central resource used to configure, monitor, and orchestrate both Azure Site Recovery and Azure Backup operations.

**26. What is the difference between a Configuration Server, Process Server, and Master Target Server in ASR (VMware scenario)?**
The Configuration Server coordinates communication between on-premises VMware and Azure and manages replication settings. The Process Server (can be co-located or scaled out separately) receives replication data from source VMs, optimizes it (caching, compression, encryption), and sends it to Azure. The Master Target Server manages replicated data during failback (when replicating from Azure back on-premises).

**27. What are replication policies in ASR, and what settings do they include (RPO threshold, recovery point retention, app-consistent snapshot frequency)?**
Replication policies define how ASR replicates and retains data: the RPO threshold (alert trigger if actual RPO exceeds the target), the recovery point retention period (how long point-in-time snapshots are kept, e.g., 24 hours), and the frequency of app-consistent snapshot creation (e.g., every 1–12 hours) which captures application-level consistency in addition to continuous crash-consistent replication.

**28. What is the difference between crash-consistent and app-consistent recovery points?**
A crash-consistent recovery point captures disk data as if the VM had crashed at that moment — data is intact, but in-flight application transactions/memory state may not be consistent. An app-consistent recovery point additionally uses VSS (Volume Shadow Copy Service on Windows) or application-specific quiescing to ensure application data (like a database) is in a consistent, immediately usable state upon recovery.

**29. How do you perform a test failover in ASR, and why is it important not to skip this step?**
A test failover spins up replicated VMs in an isolated network (optionally connected to a test VNet) without impacting ongoing replication or production. It's critical because it validates that the recovery process actually works — network mapping, boot order, application startup, dependencies — well before a real disaster, catching configuration issues in a low-risk setting.

**30. What is the difference between a planned failover, unplanned failover, and test failover?**
A planned failover is a zero-data-loss failover typically used for scheduled maintenance or drills, which synchronizes final changes from source to target before switching over. An unplanned failover is used during an actual disaster when the source is unavailable — it fails over using the latest available recovery point, potentially with some data loss. A test failover creates an isolated, non-disruptive copy purely for validation, without impacting production replication.

**31. What is a Recovery Plan in ASR, and how do you sequence multi-tier application failover (e.g., DB tier before app tier)?**
A Recovery Plan groups VMs into ordered "groups" (e.g., Group 1 = database tier, Group 2 = app tier, Group 3 = web tier) and executes failover group-by-group, optionally inserting manual approval steps or Azure Automation runbook scripts (e.g., to update connection strings, register with a load balancer, or run smoke tests) between groups.

**32. How do you configure network mapping between source and target regions in ASR?**
In the vault's network mapping settings, you map each source VNet (on-premises network or Azure source-region VNet) to a corresponding target VNet in the recovery region. During failover, ASR automatically attaches replicated VM NICs to the mapped target VNet/subnet according to this configuration.

**33. What is re-protection, and why is it needed after a failover?**
Re-protection reverses replication direction — after failing over to the DR region, the (now active) DR VMs are re-protected by replicating changes back toward the original (or another) region, so that a subsequent failback is possible without starting replication from scratch.

**34. How do you fail back to the primary region after a disaster is resolved?**
Once the primary region/site is available again, re-protect the failed-over VMs to replicate back, wait for synchronization, then perform a planned failover from the DR region back to the primary region, followed by re-protection again to resume normal (primary→DR) replication direction.

**35. What are the supported replication frequencies in ASR?**
For most Azure VM and VMware/Hyper-V replication scenarios, ASR uses continuous (near-real-time, typically sub-30-second) replication rather than fixed-interval snapshots, with recovery points generated automatically to provide granular restore options within the retention window.

**36. How does ASR handle encrypted VMs (Azure Disk Encryption) during replication?**
ASR supports replicating VMs with Azure Disk Encryption (ADE) enabled, provided Key Vault access and permissions are correctly configured in both source and target regions so that keys/secrets used to encrypt the disks are available for the replicated (target) VM to boot and decrypt.

**37. What are the limitations of Azure Site Recovery (unsupported VM configurations, disk sizes, OS versions)?**
Limitations include maximum disk size/count per VM, unsupported OS versions (only specific supported Windows/Linux versions), limits on the number of disks that can be replicated, restrictions around Ultra Disks or certain premium SSD v2/shared disks, and constraints on nested virtualization or certain generation-2 VM features (varies by scenario and is updated over time — always check current docs).

**38. How do you monitor replication health in ASR, and what alerts can be configured?**
The Recovery Services Vault dashboard shows replication health status per item (Healthy, Warning, Critical), and Azure Monitor/Log Analytics can be integrated for detailed metrics. Alerts can be configured for replication health issues, RPO threshold breaches, and failover/test-failover job failures, sent via action groups (email, SMS, webhook, ITSM integration).

**39. What is the cost model for Azure Site Recovery (replication cost vs. license cost)?**
ASR charges a per-protected-instance fee (with a free period for the first 31 days per instance), plus the underlying infrastructure costs incurred during replication (storage for replicated data/recovery points, outbound bandwidth/egress) and, upon failover, standard compute costs for the running DR VMs. No separate Windows Server license fee is charged by ASR itself, but VM compute costs apply once VMs are actually running in the target region.

**40. How do you set up disaster recovery for Azure VMs between two Azure regions using ASR?**
Create a Recovery Services Vault in the target region, select the source VMs, define/select a replication policy, configure target region resources (resource group, VNet mapping, storage, availability options), enable replication, monitor initial replication until "Protected" status, and periodically perform test failovers to validate.

**41. Can you use ASR for disaster recovery of SQL Server, SAP, or other application-specific workloads? How?**
Yes — ASR replicates at the VM/disk level, so it can protect SQL Server or SAP VMs as part of a Recovery Plan. However, for application-consistent, database-aware DR (e.g., synchronous replication, automatic failover of SQL Always On Availability Groups, or SAP HANA System Replication), application-native replication technologies are often combined with or preferred over ASR for the data tier, while ASR handles the surrounding VM/infrastructure layer.

**42. What is the difference between ASR and Azure Migrate?**
Azure Site Recovery is designed for ongoing, continuous replication for disaster recovery — the source remains the production environment. Azure Migrate is designed for one-time (or staged) migration of workloads *into* Azure permanently — after cutover, the source is decommissioned. They share underlying replication technology but serve different lifecycle purposes.

**43. How does ASR integrate with Azure Automation runbooks during failover (pre/post scripts)?**
Within a Recovery Plan, you can insert Azure Automation runbook steps before or after each group's failover — commonly used to update DNS records, adjust load balancer configuration, run application health checks, or notify stakeholders — automating tasks that would otherwise require manual intervention during failover.

**44. What is the retention period for recovery points in ASR, and how is it configured?**
Retention period is configured within the replication policy (commonly ranging from a few hours up to 15 days for Azure-to-Azure scenarios, depending on the source type), determining how long point-in-time recovery points are kept and can be used for failover, giving flexibility to recover to a point before corruption or an unwanted change occurred.

**45. How do you handle IP address changes when failing over to a different region/network?**
Plan target VNet/subnet address spaces in advance (often mirroring or non-overlapping with source), optionally configure retained IP addresses if supported, and use DNS-level failover (Traffic Manager/Front Door) or automation scripts (via Recovery Plan) to update application configuration, connection strings, or DNS records to reflect new IPs post-failover.

---

## 4. Azure Backup

**46. What is Azure Backup, and how does it differ from Azure Site Recovery?**
Azure Backup is a managed service for backing up and restoring data (VMs, files, databases, on-premises servers) with long-term retention — its purpose is data protection and point-in-time recovery. ASR's purpose is business continuity — replicating entire VMs/workloads to enable rapid failover with minimal downtime. Backup protects against data loss; ASR protects against site/region unavailability.

**47. What is a Recovery Services Vault vs. a Backup Vault — when is each used?**
A Recovery Services Vault supports both Azure Backup (for VMs, SQL, SAP HANA, on-premises via MARS/DPM/MABS) and Azure Site Recovery. A (newer) Backup Vault is used specifically for Azure Backup of newer workload types like Azure Database for PostgreSQL, Blob storage backup, and Disk backup — it uses a different underlying storage model. Choice depends on the specific workload being backed up.

**48. What backup policies are available in Azure Backup (daily, weekly, monthly, yearly retention)?**
Azure Backup policies let you define scheduled backup frequency (daily or hourly for some workloads) plus tiered retention rules — e.g., retain daily backups for 30 days, weekly backups for 12 weeks, monthly backups for 12 months, and yearly backups for up to 10 years — enabling granular, cost-optimized long-term retention (GFS — Grandfather-Father-Son style).

**49. How does Azure Backup support backup for Azure VMs, SQL databases, file shares, and on-premises servers?**
Azure VMs: agentless, snapshot-based backup orchestrated by the Backup service. SQL Server in Azure VMs: workload-aware backup with log backups for point-in-time restore. Azure File Shares: native snapshot-based backup. On-premises servers/files: via the MARS agent (files/folders) or Azure Backup Server/DPM (VMs, SQL, SharePoint, Exchange) which then backs up to a Recovery Services Vault.

**50. What is Instant Restore in Azure Backup, and how does it work?**
Instant Restore uses locally retained snapshots (stored in the customer's own subscription/storage for a short window, typically 1–7 days) to enable very fast VM or disk restores without needing to retrieve data from the vault, significantly reducing restore time for recent recovery points.

**51. What is Soft Delete in Azure Backup, and why is it important for ransomware protection?**
Soft Delete retains backup data for a configurable period (e.g., 14 days) even after a backup or the entire vault item is deleted, whether accidentally or maliciously (e.g., by ransomware attempting to destroy backups). It gives administrators a window to recover deleted backup data before it's permanently purged.

**52. How does Azure Backup handle encryption of backup data (at rest and in transit)?**
Data in transit is encrypted using HTTPS/TLS. Data at rest in the vault is encrypted by default using Microsoft-managed keys, with the option to use customer-managed keys (CMK) via Azure Key Vault for organizations requiring control over encryption keys.

**53. What is Cross-Region Restore, and when would you use it?**
Cross-Region Restore lets you restore Azure VM (or other supported workload) backups directly into a secondary (paired) Azure region, without waiting for a regional outage — useful for DR drills, or to restore into the secondary region if the primary region is degraded/unavailable. It requires the vault to use GRS or RA-GRS storage redundancy.

**54. What is the difference between Locally Redundant Storage (LRS), Geo-Redundant Storage (GRS), and Zone-Redundant Storage (ZRS) for backup vaults?**
LRS replicates data three times within a single datacenter (protects against hardware failure only). ZRS replicates synchronously across three Availability Zones in the same region (protects against datacenter failure). GRS replicates data to a secondary, paired region asynchronously (protects against regional failure) — RA-GRS additionally allows read access to the secondary region's copy.

**55. How do you restore an Azure VM from a backup to a different subscription or region?**
Use Cross-Region Restore (to the paired region using GRS) for regional restores, or use the "restore as files" option to extract disk data and manually recreate the VM/disks in a different subscription, or leverage Backup Center's cross-subscription restore capability where supported, attaching restored disks to a new VM in the target subscription.

**56. What is Azure Backup Center, and what value does it provide across multiple vaults/subscriptions?**
Backup Center is a unified management dashboard across all Recovery Services and Backup vaults, subscriptions, and even Azure Arc-enabled resources — providing centralized monitoring, policy management, compliance reporting (backup coverage/inventory), and job tracking without needing to navigate to each vault individually.

**57. How do you automate backup policy assignment at scale using Azure Policy?**
Use built-in Azure Policy definitions (e.g., "Configure backup on VMs without a given tag to a new/existing Recovery Services vault") assigned at management group or subscription scope with "DeployIfNotExists" effect, so any new VM matching criteria is automatically enrolled into a defined backup policy without manual intervention.

**58. What is the MARS agent (Microsoft Azure Recovery Services agent), and when do you use it?**
The MARS agent is a lightweight agent installed directly on Windows Servers (on-premises or in any cloud) to back up individual files, folders, and system state directly to a Recovery Services Vault — used for simple file-level backup without needing System Center DPM or Azure Backup Server.

**59. What is the difference between application-consistent, file-consistent, and crash-consistent backups?**
Crash-consistent captures data as-is (like an abrupt power loss) — files may be intact but applications may need recovery/repair on restart. File-consistent ensures files themselves aren't corrupted mid-write, but doesn't guarantee application transaction consistency. Application-consistent uses VSS or application APIs to quiesce the application (flush memory, complete transactions) before the snapshot, ensuring the app can start cleanly with no additional recovery steps.

**60. How do you protect against accidental deletion of backup data or the vault itself (soft delete, immutability, MUA)?**
Enable Soft Delete (retains deleted backup data for a recovery window), configure Immutable Vaults (prevents reducing retention or disabling soft delete, even by an admin, for a defined period), and enable Multi-User Authorization (MUA) requiring a second approver via Azure Resource Guard before critical/destructive operations can be performed.

**61. What is Multi-User Authorization (MUA) in Azure Backup, and why is it used?**
MUA requires that critical operations (like disabling soft delete, changing retention, or stopping backup with delete-data) be approved by a second, independent user through an Azure Resource Guard resource (ideally in a different subscription/tenant with separate RBAC) — protecting against a single compromised or malicious admin account destroying backup protections.

---

## 5. Application & Data Tier DR Strategies

**62. How do you design DR for a multi-tier application (web, app, database) in Azure?**
Replicate each tier appropriately: use ASR for VM-based compute tiers (sequenced via a Recovery Plan), use native database replication (SQL Auto-Failover Groups, Cosmos DB multi-region) for the data tier, replicate network/security configuration to the target region, and use Traffic Manager/Front Door for DNS-level traffic redirection after failover, validated through regular test failovers.

**63. What DR options exist for Azure SQL Database (Active Geo-Replication, Auto-failover Groups)?**
Active Geo-Replication creates up to four readable secondary replicas in different regions with manual failover control. Auto-Failover Groups build on geo-replication by adding automatic failover (based on policy) and a stable read-write/read-only listener endpoint, so applications don't need connection-string changes after failover.

**64. What is an Auto-Failover Group in Azure SQL, and how does it differ from manual geo-replication?**
An Auto-Failover Group groups one or more databases (or an entire Managed Instance) for coordinated failover using fixed DNS endpoints, and can trigger failover automatically based on a configurable grace period after detecting an outage. Manual (Active) Geo-Replication requires the administrator to explicitly initiate failover and manage endpoint/connection-string changes.

**65. How do you achieve DR for Azure SQL Managed Instance?**
Use Auto-Failover Groups at the Managed Instance level (available since instance-level failover groups were introduced), replicating an entire secondary Managed Instance in another region with automatic or manual failover and a stable listener endpoint for applications.

**66. What DR mechanisms exist for Azure Cosmos DB (multi-region writes, automatic failover)?**
Cosmos DB supports global distribution with multiple read regions, optional multi-region (multi-master) writes for active-active scenarios, and Automatic Failover, which promotes a secondary region to primary automatically if the primary region becomes unavailable, based on regional priority ordering you define.

**67. How do you design DR for Azure Storage Accounts (GRS, RA-GRS, GZRS, RA-GZRS)?**
Choose a redundancy option based on requirements: GRS/RA-GRS asynchronously replicates to a paired region (RA- variants allow read access to the secondary). GZRS/RA-GZRS combine zone redundancy in the primary region with geo-replication to a secondary region, offering the highest availability and durability. For a true DR/failover capability (not just read access), enable account failover (customer-managed or Microsoft-managed) as well.

**68. What is the difference between GRS and RA-GRS, and what does "read access" mean during an outage?**
GRS replicates data to the secondary region but that copy isn't accessible unless a full account failover is performed. RA-GRS (Read-Access GRS) additionally exposes a read-only endpoint to the secondary region's data at all times, allowing applications to read (not write) data from the secondary region even before/without a failover — useful for read availability during a primary region outage.

**69. How do you implement DR for Azure Kubernetes Service (AKS) workloads?**
Deploy AKS clusters in multiple regions (active-active or active-passive), use GitOps/Infrastructure-as-Code to keep cluster configuration/manifests consistent, replicate persistent data (e.g., via geo-replicated storage or database services outside the cluster), and use Traffic Manager/Front Door to route traffic; container images should be stored in a geo-replicated Azure Container Registry.

**70. What DR strategies exist for App Service (Azure Web Apps) — deployment slots, Traffic Manager, multi-region App Service Plans?**
Deploy the same App Service app to App Service Plans in two (or more) regions, use Azure Traffic Manager or Azure Front Door with priority/failover routing to detect outages and redirect traffic, and keep deployments synchronized via CI/CD pipelines deploying to both regions simultaneously. Deployment slots handle in-region blue/green deployments, not cross-region DR, so they're complementary, not a DR substitute.

**71. How do you design DR for Azure Virtual Desktop / VDI environments?**
Deploy host pools in a secondary region with pre-staged (or golden image-based) session hosts, replicate FSLogix profile containers/user data via geo-redundant storage or Azure Files with backup, and use a DR-ready identity setup (e.g., Azure AD DS or AD DS replicated to the DR region) so users can be redirected to the secondary host pool during an outage.

**72. What is the recommended DR approach for stateful vs. stateless applications?**
Stateless applications (e.g., web/API front-ends) are simpler to make DR-ready — just deploy identical instances in multiple regions and use traffic routing, since there's no data to synchronize. Stateful applications (databases, file stores) require careful data replication strategy (synchronous vs asynchronous, consistency guarantees) since the state itself must be consistently available in the DR region.

**73. How do you handle DNS failover during a DR event (Azure Traffic Manager vs. Azure Front Door)?**
Both provide DNS/traffic-based failover using health probes: if the primary endpoint fails health checks, traffic is automatically routed to the secondary/DR endpoint. Traffic Manager operates at the DNS layer (works for any protocol, but failover speed is bound by DNS TTL and doesn't inspect HTTP payload). Front Door operates at Layer 7 (HTTP/S) as a global reverse-proxy/CDN, offering faster failover, WAF integration, and URL-based routing, but is HTTP(S)-specific.

**74. What is the role of Azure Load Balancer/Application Gateway in a multi-region DR architecture?**
Load Balancer and Application Gateway distribute traffic *within* a region across healthy VM instances/backend pools. In a multi-region DR design, they sit behind a global traffic manager (Traffic Manager/Front Door), which handles the *cross-region* routing/failover decision, while the regional load balancer/gateway handles intra-region distribution and health probing.

**75. How do you ensure data consistency across regions for a globally distributed application?**
Choose an appropriate consistency model per data store (e.g., Cosmos DB's tunable consistency levels — strong, bounded staleness, session, eventual), use synchronous replication for the strongest consistency (at a latency/cost trade-off) or asynchronous replication with conflict resolution logic for eventual consistency, and design the application to handle read-after-write and conflict scenarios appropriately.

---

## 6. Networking Considerations in DR

**76. How do you replicate network configurations (VNets, NSGs, route tables) to the DR region?**
Use Infrastructure-as-Code (ARM/Bicep templates, Terraform) to define VNets, subnets, NSGs, route tables, and other network resources so identical (or intentionally adapted) configurations can be deployed consistently and repeatably in the DR region, avoiding manual configuration drift.

**77. What is the role of VNet peering or VPN Gateway/ExpressRoute in a DR setup?**
VNet peering connects VNets within or across regions for private connectivity between primary and DR resources (e.g., for data replication traffic). VPN Gateway/ExpressRoute provide connectivity from on-premises networks to Azure — for DR, redundant connections (dual ExpressRoute circuits, or ExpressRoute + VPN as backup) into both primary and DR regions ensure on-premises users/systems can still reach Azure resources after failover.

**78. How do you handle IP address space planning across primary and DR regions to avoid conflicts?**
Design non-overlapping CIDR ranges for primary and DR region VNets from the outset (especially if VNet peering or on-premises connectivity will be used simultaneously), and document/reserve address space centrally (e.g., via an IPAM tool or Azure Virtual Network Manager) to prevent conflicts during expansion or failover.

**79. How does Azure Traffic Manager support DR failover at the DNS level?**
Traffic Manager continuously health-probes configured endpoints (in different regions) and, using a "Priority" routing method, directs DNS resolution to the primary endpoint under normal conditions; if the primary fails health checks, it automatically resolves client requests to the next-priority (DR) endpoint instead — all without changing application configuration.

**80. What is the difference between Azure Traffic Manager and Azure Front Door for DR routing?**
Traffic Manager is a DNS-based global traffic router (protocol-agnostic, coarser failover granularity tied to DNS TTL/caching). Front Door is an application-layer (HTTP/S) global load balancer and CDN with faster failover (it proxies traffic rather than just resolving DNS), integrated WAF, SSL offload, and URL path-based routing — generally preferred for modern web/API DR scenarios requiring faster failover and Layer-7 features.

**81. How do you test network connectivity in an isolated test-failover environment without impacting production?**
ASR's test failover feature lets you optionally connect recovered VMs to a separate, isolated "test" VNet (not connected to production or on-premises networks), so you can validate boot-up, application startup, and connectivity between tiers without any risk of IP conflicts or traffic leakage into production systems.

**82. How do you handle ExpressRoute redundancy for DR scenarios?**
Provision dual ExpressRoute circuits from different peering locations/providers (or ExpressRoute + Site-to-Site VPN as a backup path) connecting to both the primary and DR regions, using ExpressRoute Global Reach or appropriately configured routing (BGP) so connectivity automatically fails over if one circuit or region's connectivity is lost.

---

## 7. Monitoring, Testing & Compliance

**83. How do you monitor the health and status of DR replication in Azure (Azure Monitor, Log Analytics)?**
Use the Recovery Services Vault's built-in replication health dashboard for a quick per-item status, and integrate with Azure Monitor/Log Analytics workbooks for deeper metrics (RPO trends, replication throughput, job history) — enabling custom dashboards, long-term trend analysis, and integration with alerting/action groups.

**84. What alerts should be configured for a robust DR monitoring strategy?**
Alerts for: replication health degraded/critical, RPO threshold breaches, failed/stuck replication or failover jobs, low free space on cache storage, expired or soon-to-expire vault credentials/keys, and failed test-failover jobs — routed to on-call teams via email/SMS/Teams/ITSM webhook through Azure Monitor action groups.

**85. How often should DR drills be performed, and what should be validated during a drill?**
Best practice is at least quarterly (or per compliance requirements, sometimes semi-annually/annually as a minimum, with more frequent tests for tier-0/critical systems). Validate: successful boot and network connectivity of recovered VMs, application startup and functional correctness, actual RTO/RPO achieved versus targets, and that runbooks/documentation are accurate and complete.

**86. What documentation is required to support a DR runbook/playbook?**
A DR runbook should include: system/application inventory and dependencies, defined RTO/RPO per tier, step-by-step failover and failback procedures, roles/responsibilities and escalation/communication plan, network diagrams (pre- and post-failover), validation/testing checklist, and rollback procedures if failover itself encounters issues.

**87. How do you calculate and validate whether actual RTO/RPO meets business requirements after a test failover?**
Measure elapsed time from failover initiation to full application availability/functional validation (actual RTO), and measure the timestamp of the last successfully replicated/recoverable transaction versus the failure point (actual RPO), then compare these measured values against the business-defined targets, documenting any gaps for remediation.

**88. What compliance/regulatory considerations impact DR design (data residency, industry regulations)?**
Data residency/sovereignty laws (e.g., GDPR in the EU) may restrict which regions can legally hold copies of certain data, industry regulations (HIPAA, PCI-DSS, financial services regulations) may mandate specific RTO/RPO, retention periods, encryption standards, or audit trails, and some sectors require documented, tested DR/BCP plans as part of compliance audits.

**89. How do you audit and report on DR readiness across an organization using Azure tools?**
Use Azure Backup Center and the Site Recovery dashboard for coverage/compliance reporting across vaults and subscriptions, Azure Policy compliance reports to confirm backup/DR policy enforcement, Azure Resource Graph queries for inventory-level reporting, and Log Analytics/Workbooks for custom dashboards summarizing replication health, test failover history, and RPO compliance trends.

**90. What is Azure Chaos Studio, and how can it be used to proactively test resilience?**
Azure Chaos Studio is a managed chaos engineering service that lets you intentionally inject faults (VM shutdowns, network latency, service outages, CPU pressure, AZ/region-down simulations) into Azure resources in a controlled way, to proactively validate that applications and DR mechanisms respond and recover as expected — before a real disaster occurs.

---

## 8. Cost & Governance

**91. How do you estimate the cost of implementing DR in Azure (compute, storage, network, licensing)?**
Estimate: ASR per-instance protection fees, replicated storage costs in the target region, egress/bandwidth costs for cross-region replication traffic, standby compute costs (if running warm-standby VMs) or compute costs incurred only upon failover (for pilot-light/cold designs), Backup vault storage costs based on redundancy tier (LRS/GRS) and retention, and any licensing implications (e.g., Azure Hybrid Benefit applicability in the DR region).

**92. What are cost optimization strategies for DR (e.g., using lower-tier VMs in DR region until failover)?**
Use a "pilot light" or "warm standby" model — smaller/fewer VM sizes running (or de-allocated VMs pre-configured) in the DR region, scaling up only during an actual failover; use reserved instances/savings plans only for baseline capacity, not full DR capacity; use lifecycle-managed storage tiers for backups; and use Cross-Region Restore instead of always-running DR infrastructure where RTO permits.

**93. How does Azure Policy help enforce DR/backup compliance across subscriptions?**
Azure Policy can audit or automatically remediate resources that lack required backup/DR configuration (e.g., flag or auto-enroll VMs without a backup policy, enforce GRS on storage accounts, require Recovery Services Vault encryption settings), ensuring consistent DR governance across many subscriptions without relying on manual processes.

**94. What is a "warm standby" vs. "pilot light" vs. "cold standby" vs. "multi-site active-active" DR strategy, and what are the cost/complexity trade-offs of each?**
Cold standby: infrastructure exists only as templates/backups, provisioned from scratch during a disaster — lowest cost, highest RTO. Pilot light: a minimal core (e.g., database replication only) runs continuously in the DR region, with the rest scaled up on failover — low-moderate cost, moderate RTO. Warm standby: a scaled-down but fully functional copy runs continuously, scaled up to full capacity on failover — higher cost, lower RTO. Multi-site active-active: full production capacity runs simultaneously in multiple regions, serving live traffic — highest cost/complexity, near-zero RTO/RPO.

**95. How do you govern DR configurations across a multi-subscription/multi-tenant Azure environment using Azure Landing Zones?**
Bake DR requirements into the Landing Zone's platform foundation — e.g., centrally managed Recovery Services Vaults per region/workload landing zone, Azure Policy assignments (via management groups) enforcing backup/DR-related configuration on all subscriptions, standardized network topology (hub-spoke with DR region hub) via Azure Virtual Network Manager, and centralized monitoring/alerting through a shared Log Analytics workspace.

---

## 9. Scenario-Based / Troubleshooting Questions

**96. A failover was triggered, but replication was behind by several hours — what could have caused this, and how would you investigate?**
Possible causes: insufficient network bandwidth between source and target, high change-rate on source disks exceeding replication throughput, throttling/resource constraints on the Process Server (VMware scenario) or cache storage, or an issue with the replication agent itself. Investigate via the vault's replication health/diagnostics, Process Server performance counters, network bandwidth utilization, and Azure Monitor logs for replication job errors/latency trends.

**97. How would you design a DR solution for a hybrid environment (on-premises + Azure)?**
Use ASR to replicate on-premises VMware/Hyper-V VMs to Azure as the DR target, establish redundant connectivity (ExpressRoute/VPN) into Azure, replicate identity infrastructure (e.g., Azure AD Connect, domain controllers or Azure AD DS) so authentication works post-failover, and ensure DNS and any on-premises-dependent services have an Azure-hosted equivalent or fallback during a disaster.

**98. Your company requires an RPO of near-zero for a critical database — what Azure services would you recommend?**
For SQL: Auto-Failover Groups with synchronous-like replication behavior via Business Critical tier (which uses Always On Availability Groups internally) for minimal data loss. For Cosmos DB: multi-region writes with strong/bounded-staleness consistency. In general, prioritize services offering synchronous or near-synchronous cross-region/cross-zone replication rather than asynchronous VM-level replication (like standard ASR), which inherently has some replication lag.

**99. How would you architect DR for an application that must remain available even if an entire Azure region is unavailable?**
Design a true multi-region active-active (or active-passive with automated failover) architecture: deploy application tiers in at least two regions, use globally distributed data stores (Cosmos DB multi-region, SQL Auto-Failover Groups), route traffic via Azure Front Door/Traffic Manager with automated health-probe-based failover, and ensure supporting services (Key Vault, Container Registry, identity) are also geo-redundant.

**100. A test failover fails due to network mapping issues — how would you troubleshoot it?**
Verify that network mapping is correctly configured in the vault (source VNet mapped to the intended target/test VNet), confirm the target/test VNet exists in the correct region with available subnet address space, check that NSGs/route tables on the target VNet don't block required traffic, and review the specific ASR job error details for the precise failure reason (e.g., missing target resources, quota limits).

**101. How do you handle licensing (Windows/SQL Server) considerations when failing over to a DR region?**
If using Azure Hybrid Benefit (bringing your own on-premises licenses with Software Assurance), ensure the benefit is applied/available for the DR region's VMs as well; verify SQL Server licensing (per-core or Azure Hybrid Benefit for SQL) covers the failed-over instance; and confirm any third-party software licenses aren't tied to specific hardware/region identifiers that would block activation after failover.

**102. Post-failover, how do you ensure monitoring, backup, and security policies are reapplied in the new (DR) region?**
Use Infrastructure-as-Code and Azure Policy (with DeployIfNotExists effects) so monitoring agents, backup enrollment, and security baselines (NSGs, Defender for Cloud, diagnostic settings) are automatically reapplied to resources in the DR region as part of the failover automation, rather than relying on manual post-failover configuration steps.

**103. How would you design a cost-effective DR solution for a non-critical, dev/test workload vs. a mission-critical production workload?**
For dev/test: rely on backups with restore-on-demand (cold standby), accept longer RTO/RPO, and skip continuous replication entirely to minimize cost. For mission-critical production: invest in continuous replication (ASR or native database geo-replication), warm standby or active-active architecture, tighter RTO/RPO targets, and regular automated DR testing — accepting the higher ongoing cost as justified by business impact.

**104. What steps would you take if the primary region recovers but data has diverged between primary and DR sites?**
Assess and reconcile the divergence (e.g., using database-level conflict detection/merge tools, or accepting the DR site's data as authoritative and treating any primary-only changes as lost), then re-establish replication direction (re-protect) from the current authoritative site to the other before performing a controlled, planned failback — avoiding a blind failback that could silently overwrite newer or conflicting data.

**105. How do you handle third-party dependencies (e.g., external APIs, licensing servers) that aren't present in the DR region?**
Identify all external dependencies during DR planning (via a dependency mapping exercise), ensure network paths (firewall rules, outbound connectivity) from the DR region can reach those external endpoints, consider deploying redundant or regional instances of on-premises dependencies (like license servers) into the DR region if feasible, and explicitly test these external integrations during DR drills rather than assuming they'll "just work."

---

*Use this list as a study guide, interview prep resource, or a basis for building an Azure DR readiness checklist and runbook.*
