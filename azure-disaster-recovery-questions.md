# Azure Disaster Recovery — Interview & Study Questions

A comprehensive question bank covering Azure Disaster Recovery (DR), Business Continuity, Azure Site Recovery (ASR), Azure Backup, and related high-availability concepts.

---

## 1. Fundamentals & Concepts

1. What is Disaster Recovery (DR), and how does it differ from High Availability (HA)?
2. What is Business Continuity and Disaster Recovery (BCDR)?
3. Define RTO (Recovery Time Objective) and RPO (Recovery Point Objective). How are they different?
4. How do you determine acceptable RTO/RPO values for a business-critical application?
5. What is the difference between backup and disaster recovery?
6. What are the key components of a DR strategy?
7. What is a DR drill/failover test, and why is it important?
8. What is the difference between failover and failback?
9. What is meant by "Recovery Plan" in Azure Site Recovery?
10. What is the shared responsibility model in the context of DR on Azure?
11. What is a Service Level Agreement (SLA), and how does Azure define SLAs for availability and DR-related services?
12. What is the difference between site-level, region-level, and zone-level disaster recovery?

---

## 2. Azure Regions, Availability Zones & Resilience

13. What is an Azure Region, Region Pair, and Availability Zone?
14. How does Azure Region Pairing help with disaster recovery?
15. What is the difference between Availability Sets and Availability Zones?
16. How does deploying resources across Availability Zones improve resilience compared to a single zone?
17. What happens during a region-wide outage, and how does Azure handle it?
18. What is a Availability Zone vs. paired region failover strategy — when would you use each?
19. Which Azure services are zone-redundant, zonal, or region-redundant by default?
20. What is Azure's definition of a "Recovery Region" and how do you choose one?

---

## 3. Azure Site Recovery (ASR)

21. What is Azure Site Recovery (ASR), and what workloads does it support?
22. What is the architecture of Azure Site Recovery for Azure VM to Azure VM replication?
23. How does ASR replicate on-premises VMware/Hyper-V VMs to Azure?
24. What are the prerequisites for setting up ASR for Azure VMs?
25. What is a Recovery Services Vault, and what role does it play in ASR?
26. What is the difference between a Configuration Server, Process Server, and Master Target Server in ASR (VMware scenario)?
27. What are replication policies in ASR, and what settings do they include (RPO threshold, recovery point retention, app-consistent snapshot frequency)?
28. What is the difference between crash-consistent and app-consistent recovery points?
29. How do you perform a test failover in ASR, and why is it important not to skip this step?
30. What is the difference between a planned failover, unplanned failover, and test failover?
31. What is a Recovery Plan in ASR, and how do you sequence multi-tier application failover (e.g., DB tier before app tier)?
32. How do you configure network mapping between source and target regions in ASR?
33. What is re-protection, and why is it needed after a failover?
34. How do you fail back to the primary region after a disaster is resolved?
35. What are the supported replication frequencies in ASR?
36. How does ASR handle encrypted VMs (Azure Disk Encryption) during replication?
37. What are the limitations of Azure Site Recovery (unsupported VM configurations, disk sizes, OS versions)?
38. How do you monitor replication health in ASR, and what alerts can be configured?
39. What is the cost model for Azure Site Recovery (replication cost vs. license cost)?
40. How do you set up disaster recovery for Azure VMs between two Azure regions using ASR?
41. Can you use ASR for disaster recovery of SQL Server, SAP, or other application-specific workloads? How?
42. What is the difference between ASR and Azure Migrate?
43. How does ASR integrate with Azure Automation runbooks during failover (pre/post scripts)?
44. What is the retention period for recovery points in ASR, and how is it configured?
45. How do you handle IP address changes when failing over to a different region/network?

---

## 4. Azure Backup

46. What is Azure Backup, and how does it differ from Azure Site Recovery?
47. What is a Recovery Services Vault vs. a Backup Vault — when is each used?
48. What backup policies are available in Azure Backup (daily, weekly, monthly, yearly retention)?
49. How does Azure Backup support backup for Azure VMs, SQL databases, file shares, and on-premises servers?
50. What is Instant Restore in Azure Backup, and how does it work?
51. What is Soft Delete in Azure Backup, and why is it important for ransomware protection?
52. How does Azure Backup handle encryption of backup data (at rest and in transit)?
53. What is Cross-Region Restore, and when would you use it?
54. What is the difference between Locally Redundant Storage (LRS), Geo-Redundant Storage (GRS), and Zone-Redundant Storage (ZRS) for backup vaults?
55. How do you restore an Azure VM from a backup to a different subscription or region?
56. What is Azure Backup Center, and what value does it provide across multiple vaults/subscriptions?
57. How do you automate backup policy assignment at scale using Azure Policy?
58. What is the MARS agent (Microsoft Azure Recovery Services agent), and when do you use it?
59. What is the difference between application-consistent, file-consistent, and crash-consistent backups?
60. How do you protect against accidental deletion of backup data or the vault itself (soft delete, immutability, MUA)?
61. What is Multi-User Authorization (MUA) in Azure Backup, and why is it used?

---

## 5. Application & Data Tier DR Strategies

62. How do you design DR for a multi-tier application (web, app, database) in Azure?
63. What DR options exist for Azure SQL Database (Active Geo-Replication, Auto-failover Groups)?
64. What is an Auto-Failover Group in Azure SQL, and how does it differ from manual geo-replication?
65. How do you achieve DR for Azure SQL Managed Instance?
66. What DR mechanisms exist for Azure Cosmos DB (multi-region writes, automatic failover)?
67. How do you design DR for Azure Storage Accounts (GRS, RA-GRS, GZRS, RA-GZRS)?
68. What is the difference between GRS and RA-GRS, and what does "read access" mean during an outage?
69. How do you implement DR for Azure Kubernetes Service (AKS) workloads?
70. What DR strategies exist for App Service (Azure Web Apps) — deployment slots, Traffic Manager, multi-region App Service Plans?
71. How do you design DR for Azure Virtual Desktop / VDI environments?
72. What is the recommended DR approach for stateful vs. stateless applications?
73. How do you handle DNS failover during a DR event (Azure Traffic Manager vs. Azure Front Door)?
74. What is the role of Azure Load Balancer/Application Gateway in a multi-region DR architecture?
75. How do you ensure data consistency across regions for a globally distributed application?

---

## 6. Networking Considerations in DR

76. How do you replicate network configurations (VNets, NSGs, route tables) to the DR region?
77. What is the role of VNet peering or VPN Gateway/ExpressRoute in a DR setup?
78. How do you handle IP address space planning across primary and DR regions to avoid conflicts?
79. How does Azure Traffic Manager support DR failover at the DNS level?
80. What is the difference between Azure Traffic Manager and Azure Front Door for DR routing?
81. How do you test network connectivity in an isolated test-failover environment without impacting production?
82. How do you handle ExpressRoute redundancy for DR scenarios?

---

## 7. Monitoring, Testing & Compliance

83. How do you monitor the health and status of DR replication in Azure (Azure Monitor, Log Analytics)?
84. What alerts should be configured for a robust DR monitoring strategy?
85. How often should DR drills be performed, and what should be validated during a drill?
86. What documentation is required to support a DR runbook/playbook?
87. How do you calculate and validate whether actual RTO/RPO meets business requirements after a test failover?
88. What compliance/regulatory considerations impact DR design (data residency, industry regulations)?
89. How do you audit and report on DR readiness across an organization using Azure tools?
90. What is Azure Chaos Studio, and how can it be used to proactively test resilience?

---

## 8. Cost & Governance

91. How do you estimate the cost of implementing DR in Azure (compute, storage, network, licensing)?
92. What are cost optimization strategies for DR (e.g., using lower-tier VMs in DR region until failover)?
93. How does Azure Policy help enforce DR/backup compliance across subscriptions?
94. What is a "warm standby" vs. "pilot light" vs. "cold standby" vs. "multi-site active-active" DR strategy, and what are the cost/complexity trade-offs of each?
95. How do you govern DR configurations across a multi-subscription/multi-tenant Azure environment using Azure Landing Zones?

---

## 9. Scenario-Based / Troubleshooting Questions

96. A failover was triggered, but replication was behind by several hours — what could have caused this, and how would you investigate?
97. How would you design a DR solution for a hybrid environment (on-premises + Azure)?
98. Your company requires an RPO of near-zero for a critical database — what Azure services would you recommend?
99. How would you architect DR for an application that must remain available even if an entire Azure region is unavailable?
100. A test failover fails due to network mapping issues — how would you troubleshoot it?
101. How do you handle licensing (Windows/SQL Server) considerations when failing over to a DR region?
102. Post-failover, how do you ensure monitoring, backup, and security policies are reapplied in the new (DR) region?
103. How would you design a cost-effective DR solution for a non-critical, dev/test workload vs. a mission-critical production workload?
104. What steps would you take if the primary region recovers but data has diverged between primary and DR sites?
105. How do you handle third-party dependencies (e.g., external APIs, licensing servers) that aren't present in the DR region?

---

*Use this list as a study guide, interview prep resource, or a basis for building an Azure DR readiness checklist.*
