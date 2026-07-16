# Azure Infrastructure & Security Engineer — Interview Prep Guide
### Enkaytech | Interviewer: Senior Azure Infrastructure & Security Consultant

This guide is built from two sources:
1. The **job posting** (Terraform/IaC, networking, monitoring, security, DR, cost, governance).
2. The **interviewer's own role** (Landing Zones, Zero Trust, hybrid connectivity, client delivery across Insurance/Logistics/Hospitality).

Questions move from **basics → intermediate → advanced/architectural**, since a senior consultant interviewing you will typically start with fundamentals to check depth, then push into design and trade-off discussions.

---

## 1. Networking Basics (TCP/IP, DNS, VPN, Firewalls, Load Balancing)

**Q: Explain the OSI/TCP-IP model briefly. Why does it matter for cloud networking?**
A: TCP/IP has 4 practical layers — Network Access, Internet, Transport, Application. In Azure, this maps to VNets/subnets (Internet layer), NSGs/Load Balancers (Transport layer), and Application Gateway/App services (Application layer). Understanding it helps you know *where* to apply a control (e.g., NSG vs WAF).

**Q: What is DNS and how does Azure handle it?**
A: DNS resolves names to IPs. Azure offers **Azure DNS** (public zones), **Azure Private DNS** (name resolution inside VNets, auto-registration for VMs), and DNS forwarding for hybrid setups (conditional forwarders between on-prem DNS and Azure Private DNS Resolver).

**Q: Difference between a Load Balancer, Application Gateway, and Traffic Manager?**
A:
- **Azure Load Balancer** — Layer 4 (TCP/UDP), regional, distributes traffic across VMs/VMSS.
- **Application Gateway** — Layer 7, HTTP(S), supports URL-based routing, SSL termination, and WAF.
- **Traffic Manager** — DNS-based global traffic routing across regions (not a proxy, just DNS redirection).
- **Front Door** — Global Layer 7 with WAF + CDN + routing, replacing Traffic Manager + App Gateway combos for global apps.

**Q: What's the difference between a VPN Gateway and ExpressRoute?**
A: VPN Gateway is an encrypted tunnel over the public internet (Site-to-Site, Point-to-Site). ExpressRoute is a private, dedicated circuit through a connectivity provider — no public internet, more bandwidth, lower latency, SLA-backed. ExpressRoute is preferred for enterprise hybrid connectivity; VPN is often used as a **backup path** to ExpressRoute.

**Q: What is SMTP and why would an infra engineer care about it?**
A: SMTP is the protocol for sending email. Azure blocks outbound port 25 by default on VMs to prevent spam; relaying app-generated emails requires a smart host (e.g., SendGrid) or explicit exception request from Azure. Relevant when a client's app needs to send transactional emails.

**Q: Explain NSGs vs Azure Firewall vs a 3rd-party NVA (e.g., Palo Alto/Fortinet).**
A: NSG = stateless-ish, basic L3/L4 allow/deny rules per subnet/NIC. Azure Firewall = managed, stateful, centralized, supports FQDN filtering, threat intelligence. NVAs = full-featured firewalls (IPS/IDS, deep packet inspection) deployed as VMs, usually in a hub VNet for centralized security, common in enterprises with existing on-prem firewall vendors.

---

## 2. Azure Core Infrastructure Basics

**Q: What is a Resource Group and how should it be used?**
A: A logical container for resources sharing a lifecycle (deploy/delete together). Best practice: group by lifecycle/environment (e.g., `rg-prod-networking`, `rg-dev-app1`), not just by application, since RGs are also an IAM and policy scope boundary.

**Q: Explain Azure subscription vs Management Group vs Tenant hierarchy.**
A: Tenant (Azure AD/Entra ID) → Management Groups (organize subscriptions, apply policy/RBAC at scale) → Subscriptions (billing + scale boundary) → Resource Groups → Resources. This hierarchy is exactly what a **Landing Zone** is built on.

**Q: VNet peering vs VPN Gateway between VNets — when to use which?**
A: Peering = private, low-latency, backbone-routed, no bandwidth bottleneck, but no transitive routing by default (needs Virtual WAN or hub-spoke with UDRs/NVA for transitivity). VPN Gateway between VNets = used when peering isn't possible (e.g., cross-tenant without proper permissions) or encryption-in-transit is mandated over Microsoft backbone.

**Q: What is a Hub-Spoke topology and why use it?**
A: Hub VNet centralizes shared services (Firewall, VPN/ExpressRoute Gateway, DNS resolver, Bastion). Spoke VNets (per app/team) peer to the hub only. This centralizes security/inspection and reduces duplicate gateway costs. It's the backbone pattern behind Azure Landing Zones.

---

## 3. Infrastructure as Code (Terraform) — Basics to Advanced

**Q: Why Terraform over ARM templates or Bicep?**
A: Terraform is cloud-agnostic (multi-cloud state), has a large module ecosystem, and a mature plan/apply workflow. Bicep is Azure-native with tighter ARM integration and no state file to manage. Many enterprises choose Terraform when they're multi-cloud or already have Terraform tooling/pipelines.

**Q: Explain Terraform's core workflow.**
A: `init` (download providers/modules) → `plan` (dry-run diff against state) → `apply` (execute changes) → `destroy` (teardown). State file (`terraform.tfstate`) tracks real-world resource mapping to config.

**Q: How does Terraform state work, and how do you secure it in a team?**
A: State stores resource IDs/attributes mapping config to real infra. In teams, state must be **remote** (Azure Storage Blob with a lease-based lock, or Terraform Cloud) — never local — to avoid conflicts and enable locking. Sensitive values in state should be encrypted at rest (Storage Account encryption) and access restricted via RBAC/SAS; consider `sensitive = true` on outputs and avoid storing secrets in state where possible (use Key Vault references instead).

**Q: What's a Terraform module and why use them?**
A: A reusable, parameterized bundle of resources (e.g., a "VNet module" or "AKS cluster module"). Promotes DRY code, consistency, and versioned reuse across environments/teams. Structure: `modules/<name>/{main.tf, variables.tf, outputs.tf}`, called via `source = "../modules/vnet"`.

**Q: How do you manage multiple environments (dev/test/prod) in Terraform?**
A: Common patterns: (1) **Workspaces** — same code, different state per workspace (simpler, riskier if misused); (2) **Separate state files per environment folder** with shared modules (more explicit, safer, most enterprises prefer this); combined with **tfvars files** per environment (`dev.tfvars`, `prod.tfvars`).

**Q: How do you handle secrets in Terraform (e.g., DB passwords, API keys)?**
A: Never hardcode in `.tf` files or commit to Git. Use Azure Key Vault data source to fetch secrets at apply time, or a secrets manager integrated with the pipeline (env vars injected by CI/CD with masking). Mark variables `sensitive = true` so they don't print in plan/apply logs.

**Q: What is `terraform import` and when would you use it?**
A: Brings an existing, manually created resource under Terraform management by mapping it to a resource block in state, without recreating it. Common during Terraform adoption in a brownfield environment.

**Q: What is drift, and how do you detect/handle it?**
A: Drift = real infra changed outside Terraform (manual portal change). Detect via scheduled `terraform plan` (CI job) comparing state vs actual; reconcile by either updating code to match, or reapplying to force back to code-defined state. Azure Policy can also prevent/flag manual drift.

**Q: Explain `terraform.tfstate` locking and what happens without it.**
A: Locking prevents concurrent applies corrupting state. Azure backend uses a blob lease as the lock. Without it, two engineers running `apply` simultaneously can corrupt state or cause resource conflicts.

**Q: What are Terraform providers and provisioners?**
A: Providers are plugins that translate HCL into API calls (e.g., `azurerm`). Provisioners (`local-exec`, `remote-exec`) run scripts on resource creation — generally discouraged/last-resort since they break the declarative model; prefer cloud-init, Azure VM extensions, or configuration management tools instead.

**Q: How would you structure Terraform for testing/validation before deployment?**
A: Use `terraform validate` and `terraform fmt` for syntax; **terratest** (Go-based) for integration testing actual deployed infra; **Open Policy Agent (OPA)/Conftest** or **Sentinel** (Terraform Cloud) for policy-as-code — e.g., "no public IP on storage accounts," "all resources must be tagged" — enforced in the CI/CD pipeline before `apply`.

**Q: Explain remote state data sharing between projects (e.g., networking team's VNet used by app team).**
A: Use `terraform_remote_state` data source (or better, output values published and consumed via a data source/ Key Vault) so the app team's Terraform can reference the networking team's subnet ID without duplicating code or hardcoding IDs.

---

## 4. CI/CD for Infrastructure

**Q: How do you integrate Terraform into a CI/CD pipeline (GitHub Actions/Azure DevOps/GitLab CI)?**
A: Pipeline stages: (1) lint/validate/fmt check, (2) `plan` on PR (posted as a comment for review), (3) manual approval gate for prod, (4) `apply` on merge to main, using a service principal with least-privilege RBAC and secrets stored in pipeline secret store/Key Vault, remote backend state.

**Q: How do you avoid applying to the wrong environment accidentally?**
A: Separate pipelines/service connections per environment, separate state files/backends, environment-specific approval gates, and naming conventions/tfvars that are explicit — never a single pipeline silently targeting "whatever is default."

---

## 5. Identity, Security & Zero Trust (Core to Interviewer's Role)

**Q: What is Zero Trust and how do you implement it in Azure?**
A: Principle: "never trust, always verify" — no implicit trust based on network location. In Azure: enforce **MFA everywhere**, **Conditional Access policies** (device compliance, location, risk-based sign-in), **least-privilege RBAC/PIM (just-in-time access)**, **micro-segmentation** (NSGs, Private Endpoints instead of broad VNet access), and continuous monitoring (Defender for Cloud, Sentinel).

**Q: Explain Azure AD (Entra ID) vs RBAC vs Azure Policy — how do they differ?**
A:
- **Entra ID (Azure AD)** — identity plane: users, groups, service principals, Conditional Access.
- **RBAC** — *who can do what* on *which resource scope* (control-plane permissions, e.g., Contributor on a resource group).
- **Azure Policy** — *what configurations are allowed*, regardless of who deploys (e.g., "deny public storage accounts," "audit VMs without encryption"), enforced continuously, not just at login.

**Q: What is PIM (Privileged Identity Management)?**
A: Provides just-in-time, time-bound elevation to privileged roles (e.g., Owner/Global Admin) with approval workflows and MFA re-verification, instead of standing privileged access — a core Zero Trust control.

**Q: How do you secure identity for hybrid environments (on-prem AD + Azure AD)?**
A: Azure AD Connect/Cloud Sync for hybrid identity sync, password hash sync or pass-through auth (avoid ADFS unless required), enforce MFA/Conditional Access on cloud side even for synced accounts, and monitor with Identity Protection for leaked credentials/risky sign-ins.

**Q: What's the difference between a Managed Identity and a Service Principal?**
A: A Service Principal is an app identity you create/manage (credentials to rotate). A **Managed Identity** (System- or User-assigned) is automatically managed by Azure — no credentials to store/rotate — the preferred way for Azure resources (VMs, Functions, Terraform pipelines) to authenticate to other Azure services/Key Vault.

**Q: How would you secure a storage account holding sensitive client data (Insurance sector example)?**
A: Disable public blob access, use **Private Endpoints** instead of service endpoints where possible, enforce **encryption at rest** (customer-managed keys in Key Vault for compliance-heavy clients), enable **Storage firewall** (deny by default, allow specific VNets/IPs), turn on **diagnostic logging** to Log Analytics, apply **RBAC + SAS token expiry policies**, and tag/classify data per **Microsoft Purview** for compliance visibility.

**Q: What is RBAC's difference between built-in roles like Reader, Contributor, Owner, and User Access Administrator?**
A: Reader = view only. Contributor = manage resources but not grant access to others. Owner = full control including access management. User Access Administrator = manage access only, not resources. Least privilege means assigning the most narrow built-in or custom role scoped to the specific resource/RG, not subscription-wide Owner.

**Q: How do you approach data security & compliance frameworks for regulated clients (Insurance/Hospitality — think GDPR/HIPAA/PCI-DSS)?**
A: Map client's regulatory requirement to Azure controls: encryption in transit/at rest, data residency (region pinning), access auditing (Azure Monitor + Sentinel), **Microsoft Purview** for data classification/DLP, **Azure Policy initiatives** (built-in regulatory compliance blueprints like PCI-DSS, ISO 27001) to continuously assess and enforce, and regular **Defender for Cloud** secure score reviews.

---

## 6. Azure Landing Zones & Governance (Core to Interviewer's Role)

**Q: What is an Azure Landing Zone?**
A: A pre-defined, scalable multi-subscription environment architecture (based on Microsoft's Cloud Adoption Framework) that includes identity, governance, networking, security, and management baselines — so new workloads land on a compliant, secure foundation instead of being provisioned ad hoc.

**Q: What are the key design areas of a Landing Zone (per CAF)?**
A: Azure Billing/Entra ID tenant, Identity & access, Network topology & connectivity, Security, Management (monitoring/backup), Governance (Policy), Platform automation (IaC), and Resource organization (Management Groups/subscription structure).

**Q: Difference between "Platform" landing zones and "Application" landing zones?**
A: Platform landing zones (management, connectivity, identity subscriptions) are owned centrally by the platform team and rarely change. Application landing zones are per-workload subscriptions where app teams deploy, inheriting policies/network from the platform layer but retaining autonomy within guardrails.

**Q: How does Azure Policy support governance at scale?**
A: Policies (or **initiatives** = grouped policies) are assigned at Management Group/Subscription scope with effects like `Deny`, `Audit`, `DeployIfNotExists` (auto-remediation, e.g., auto-enabling diagnostic settings). This enforces guardrails (tagging, allowed regions/SKUs, mandatory encryption) without manually reviewing every deployment.

**Q: How would you design a governance model for a client migrating multiple business units to Azure?**
A: Separate Management Groups per business unit/environment, delegated RBAC so each BU manages its own subscription within guardrails, centrally-enforced Policy initiatives (tagging, security baseline, cost controls), a hub-spoke network with centralized firewall/DNS, and a landing zone deployment pipeline (Terraform modules) so every new subscription is consistent.

---

## 7. Hybrid Connectivity (Core to Interviewer's Role)

**Q: Walk through designing hybrid connectivity between an on-prem datacenter and Azure.**
A: Options ranked by scale: (1) Site-to-Site VPN for smaller/backup connectivity, (2) ExpressRoute for primary enterprise connectivity (private, SLA'd, high bandwidth), often deployed with a VPN as failover. Use a **Virtual Network Gateway** (or **Azure Virtual WAN** for multi-site/branch scale) in the hub VNet, with **BGP** for dynamic routing, and Private DNS/DNS forwarders for name resolution across environments.

**Q: What is Azure Virtual WAN and when would you recommend it over standard hub-spoke with a VPN Gateway?**
A: Virtual WAN is a Microsoft-managed, global full-mesh hub that simplifies branch/site connectivity at scale (many sites/VPNs/ExpressRoute circuits), automates route propagation, and reduces the operational overhead of managing manual gateways — recommended when a client (e.g., Logistics with many branch locations) needs many-to-many connectivity rather than a single hub-spoke.

**Q: How do you ensure secure, private access to PaaS services (e.g., SQL, Storage) from on-prem, without going over the public internet?**
A: **Private Endpoints** (Private Link) — assigns the PaaS service a private IP inside the VNet, reachable via the existing hybrid connection (ExpressRoute/VPN) and Private DNS zone, eliminating public exposure entirely.

---

## 8. Monitoring, Logging & Optimization

**Q: What's the difference between Azure Monitor, Log Analytics, and Application Insights?**
A: Azure Monitor is the umbrella platform. **Log Analytics** is the query/storage workspace (using KQL) for logs/metrics across resources. **Application Insights** is Monitor's APM layer — application-level telemetry (requests, dependencies, exceptions) for app performance monitoring.

**Q: How would you set up proactive alerting for a production environment?**
A: Define metric/log alert rules (e.g., CPU > 80% for 5 min, failed sign-ins spike) via Azure Monitor Action Groups (email/Teams/webhook/ITSM integration), use **Log Analytics + KQL** for custom log-based alerts, and dashboards/workbooks for stakeholder visibility. For security specifically, route logs to **Microsoft Sentinel** for SIEM/SOAR correlation and automated response playbooks.

**Q: How do you approach cost optimization in Azure?**
A: **Azure Cost Management + Budgets** for visibility/alerts, **Reservations/Savings Plans** for predictable steady-state workloads, **right-sizing** using Advisor recommendations, **auto-shutdown/scale-in** for dev/test, deleting orphaned resources (unattached disks, idle public IPs), and tagging for chargeback/showback per business unit or client.

---

## 9. Disaster Recovery & Business Continuity

**Q: Explain RTO vs RPO and why they matter before designing DR.**
A: RTO (Recovery Time Objective) = how fast you must be back up. RPO (Recovery Point Objective) = how much data loss is tolerable. These business-defined numbers dictate the DR pattern chosen (e.g., low RTO/RPO needs active-active multi-region; higher tolerance can use backup/restore).

**Q: How does Azure Site Recovery (ASR) work?**
A: ASR replicates VM disks continuously to a secondary region (or on-prem to Azure), and orchestrates failover/failback with recovery plans (ordered boot sequences, scripts, network mapping) — used for VM-level DR without re-architecting the app.

**Q: What DR patterns would you propose for a client's critical workload (e.g., Hospitality booking system) with near-zero downtime tolerance?**
A: Active-active or active-passive multi-region architecture: Front Door/Traffic Manager for global routing and automatic failover, geo-replicated data tier (e.g., SQL Auto-failover groups, Cosmos DB multi-region writes), Infra defined in Terraform so the secondary region is a same-code redeploy, and regular DR drills (not just paper runbooks) to validate RTO/RPO actually met.

---

## 10. Scenario / Behavioral Questions (Client-Facing Consulting Role)

**Q: A client in Logistics wants to migrate 200 VMs to Azure with minimal downtime — how do you approach it?**
A: Discovery/assessment first (Azure Migrate for dependency mapping, sizing), categorize by migration strategy (rehost/replatform/refactor — 6 R's), pilot migrate a non-critical batch first, use **Azure Migrate + ASR-based replication** for near-zero downtime cutover, validate landing zone/network/security readiness before wave 1, and run migrations in waves with rollback plans.

**Q: How do you handle a disagreement with a client stakeholder who wants to skip security best practices to hit a deadline?**
A: Explain the risk in business terms (compliance fines, breach cost, reputational damage relevant to their sector, e.g., Insurance/PCI), propose a phased approach — meet the deadline with baseline controls (MFA, NSGs, encryption) now, and schedule the remaining hardening (e.g., Sentinel, full Zero Trust rollout) as fast-follow work, documented and signed off, rather than silently skipping it.

**Q: How do you prioritize competing initiatives across multiple clients/business units?**
A: Assess by business impact and risk (security/compliance gaps first), effort/dependency mapping, and align with stakeholders on a visible roadmap/backlog — typical tools: risk-scoring matrix, regular steering committee syncs.

**Q: Tell me about a time you troubleshot a production issue under pressure. (Prepare a STAR-format real example.)**
A: *(Personalize this with your own experience — Situation, Task, Action, Result — ideally involving Azure networking/Terraform/monitoring, since that's what this role tests.)*

---

## 11. Rapid-Fire Fundamentals (Warm-up Round)

| Question | Short Answer |
|---|---|
| What is a subnet delegation? | Reserves a subnet for a specific Azure service (e.g., App Service VNet Integration, Azure Bastion). |
| What is Azure Bastion? | Browser-based, no-public-IP RDP/SSH access to VMs via the Azure Portal. |
| What is an NSG flow log? | Logs traffic allowed/denied by NSGs, sent to Storage/Log Analytics for auditing/troubleshooting. |
| What is a Service Endpoint vs Private Endpoint? | Service Endpoint keeps traffic on Azure backbone but service still has a public IP; Private Endpoint gives the service a private IP in your VNet — more secure. |
| What is Azure Blueprints vs Landing Zone accelerators? | Blueprints (being retired/deprecated in favor of Template Specs + Policy) packaged RG/Policy/RBAC templates; Landing Zone accelerators (Terraform/Bicep-based) are the modern replacement. |
| What's a UDR (User Defined Route)? | Custom route table overriding default Azure routing, e.g., forcing subnet traffic through a firewall/NVA. |
| CMK vs Microsoft-managed keys? | Customer-Managed Keys (stored in Key Vault) give the client control/rotation ownership of encryption keys — often required for compliance-heavy sectors. |

---

## 12. Smart Questions to Ask the Interviewer

- "What does a typical Landing Zone deployment look like for a new client here — Terraform-driven from day one, or do you retrofit governance later?"
- "How is the team currently handling Zero Trust rollout — is Conditional Access/PIM standardized across clients, or tailored per engagement?"
- "Across Insurance, Logistics, and Hospitality clients, what's been the most common security or compliance gap you've had to fix post-migration?"
- "What does the Terraform module/repo structure look like across projects — shared internal module registry, or per-client repos?"

---

### Final Prep Tips
- Be ready to **whiteboard a hub-spoke Landing Zone diagram** from memory — this is very likely given the interviewer's exact role.
- Know **one real Terraform state/module story** in depth (even from personal/lab projects) — "tell me about a time you dealt with state drift/conflicts" is a near-certain question.
- Have a crisp one-liner distinguishing **RBAC vs Policy vs Zero Trust** — interviewers use this to gauge conceptual clarity fast.
- Since this is a **client-facing consulting role**, expect at least one soft-skill/prioritization question — prepare a real STAR example, not just technical answers.
