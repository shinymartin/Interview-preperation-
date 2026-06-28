# Project Interview Questions — Secure Hub-Spoke Network on Azure (Terraform)

A comprehensive set of questions an interviewer might ask about this project, from fundamentals to advanced. Each question includes concise talking points you can expand on. Organized by topic; within each topic, questions roughly progress from basic to advanced.

> Tip: when answering, tie the concept back to *this* project ("in my project I did X because Y"). That turns a textbook answer into evidence of hands-on experience.

---

## 1. Project & Architecture Overview

**1.1 Describe this project in two minutes.**
A secure hub-and-spoke network on Azure, built entirely with Terraform. A hub VNet holds shared services (Azure Firewall, Bastion, VPN Gateway). Two spoke VNets hold workloads. All egress and inter-spoke traffic is forced through the firewall via route tables; PaaS services (Storage, SQL, Key Vault) have no public access and are reached through private endpoints with private DNS; VMs have no public IPs and are accessed only via Bastion.

**1.2 What problem does this architecture solve?**
Centralized security and connectivity with network segmentation. Shared services are deployed once in the hub and reused by all spokes; workloads are isolated; a single inspection point (the firewall) enforces policy; and data services are kept off the public internet.

**1.3 What were the main security goals?**
No public IPs on workload VMs, no public access to PaaS, all egress and east-west traffic inspected by the firewall, and private DNS so private connectivity is transparent to applications.

**1.4 Why Terraform instead of the portal or ARM/Bicep?**
Repeatability, version control, code review, and a clear `plan` before changes. Terraform is also cloud-agnostic and has a large ecosystem. (ARM/Bicep are Azure-native alternatives; Terraform was chosen for its declarative state model and portability.)

**1.5 Walk me through the traffic flow when a spoke VM browses the internet.**
The VM's subnet has a UDR with `0.0.0.0/0` → firewall private IP. Traffic goes to the firewall in the hub (over peering), the firewall checks its application/network rules, and if allowed, SNATs and sends it out via the firewall's public IP. Return traffic comes back through the firewall.

---

## 2. Hub-and-Spoke Topology

**2.1 What is a hub-and-spoke topology?**
A central hub VNet connected to multiple spoke VNets via peering. The hub holds shared services; spokes hold isolated workloads. Spokes communicate with each other and the outside world through the hub.

**2.2 Why hub-and-spoke instead of a single large VNet or a full mesh?**
Segmentation and cost/manageability. A single VNet has no isolation. A full mesh needs N×(N-1) peerings and duplicated services. Hub-spoke centralizes shared services and scales linearly (one peering pair per spoke).

**2.3 Why don't the spokes peer directly to each other?**
So inter-spoke (east-west) traffic is forced through the hub firewall for inspection. Direct peering would let spokes talk without any inspection.

**2.4 How does spoke-to-spoke communication work then?**
Each spoke has a UDR sending the other spoke's CIDR to the firewall. Traffic goes spoke → firewall (hub) → other spoke. It requires `allow_forwarded_traffic = true` on the peerings and a firewall network rule permitting it.

**2.5 What lives in the hub vs. the spokes in your project?**
Hub: Azure Firewall, Bastion, VPN Gateway, and the private DNS zones. Spokes: workloads (test VMs) and, in the data spoke, the PaaS services and their private endpoints.

**2.6 How would you scale this to 20 spokes?**
Add a peering pair, route table, and (if needed) workloads per spoke. In Terraform you'd typically convert the spoke into a module with `for_each` over a map of spokes. Watch hub subnet sizing, firewall throughput, and the per-VNet peering limits.

---

## 3. Virtual Networks, Subnets & IP Addressing

**3.1 What is a VNet and a subnet?**
A VNet is an isolated private network in Azure with its own address space. Subnets segment that space and are where resources (NICs, endpoints, gateways) attach.

**3.2 What address spaces did you use and why non-overlapping?**
Hub `10.0.0.0/16`, spoke1 `10.1.0.0/16`, spoke2 `10.2.0.0/16`. Overlapping ranges break peering and routing — peering requires non-overlapping address spaces.

**3.3 Why are some subnet names fixed (AzureFirewallSubnet, AzureBastionSubnet, GatewaySubnet)?**
Azure attaches those services by subnet name. The names are reserved and case-sensitive; using anything else fails the deployment.

**3.4 What are the minimum subnet sizes for those special subnets?**
AzureFirewallSubnet /26, AzureBastionSubnet /26, GatewaySubnet /27 (a larger gateway subnet is fine and recommended for future-proofing).

**3.5 How many usable IPs are in a /24, and how many does Azure reserve per subnet?**
A /24 has 256 addresses, 251 usable — Azure reserves 5 per subnet (network, the first three for gateway/DNS/future use, and broadcast).

**3.6 Can you resize a subnet after deployment?**
Historically you had to remove resources first; Azure has added more flexibility, but it's still constrained. Plan address space up front to avoid it.

---

## 4. VNet Peering

**4.1 What is VNet peering?**
A low-latency, high-bandwidth connection between two VNets over Microsoft's backbone, making them route to each other as if on one network. It's non-transitive.

**4.2 What does "peering is non-transitive" mean for this design?**
If spoke1 peers to hub and hub peers to spoke2, spoke1 cannot reach spoke2 *through* the hub automatically. You need UDRs (and the firewall/NVA) to route that traffic — which is exactly how inter-spoke works here.

**4.3 Explain the four key peering flags you used.**
- `allow_virtual_network_access` — lets the VNets exchange traffic.
- `allow_forwarded_traffic` — lets a VNet accept traffic forwarded by an appliance (needed for spoke→firewall→spoke).
- `allow_gateway_transit` (hub side) — hub advertises its VPN gateway.
- `use_remote_gateways` (spoke side) — spoke uses the hub's gateway instead of its own.

**4.4 Why is `allow_forwarded_traffic` necessary?**
Because the firewall forwards traffic on behalf of another source. Without it, the receiving spoke drops packets that didn't originate in the peered VNet directly.

**4.5 What is gateway transit and why use it?**
It lets spokes use the hub's VPN/ExpressRoute gateway, so you deploy hybrid connectivity once in the hub instead of a gateway per spoke. Saves cost and complexity.

**4.6 You had a `depends_on` between the spoke peering and the gateway — why?**
A spoke can only set `use_remote_gateways = true` after the hub gateway exists. The explicit dependency forces Terraform to create the gateway first.

**4.7 Is peering encrypted? Does it cost?**
Traffic stays on Microsoft's private backbone (not the public internet). Peering is not "encrypted" by default in the IPsec sense, though Microsoft encrypts traffic between datacenters. Peering is billed for ingress/egress data transfer.

---

## 5. Routing & User-Defined Routes (UDRs)

**5.1 What is a UDR / route table?**
A user-defined route overrides Azure's default system routes. Attached to a subnet, it controls the next hop for given destination prefixes.

**5.2 What routes did you create and why?**
Per spoke: `0.0.0.0/0` → firewall (force all egress through inspection) and the other spoke's CIDR → firewall (force east-west through inspection).

**5.3 What is the next-hop type and value for sending traffic to the firewall?**
`next_hop_type = "VirtualAppliance"` and `next_hop_in_ip_address` = the firewall's **private** IP. Never the public IP.

**5.4 Why the private IP and not the public IP of the firewall?**
The public IP is for inbound DNAT and outbound SNAT identity. Internal routing must target the firewall's private interface inside the AzureFirewallSubnet.

**5.5 Why must you NOT put a `0.0.0.0/0` → firewall route on the AzureFirewallSubnet itself?**
It creates a routing loop — the firewall would try to send its own egress back to itself.

**5.6 How does Azure decide which route wins when several match?**
Longest-prefix match (most specific route wins). If prefixes tie, UDR > BGP > system route by priority.

**5.7 With a `0.0.0.0/0` → firewall route on the spoke, why does Bastion still reach the VM?**
The peering creates a more specific system route for the hub range (`10.0.0.0/16`), which beats `0.0.0.0/0`. So hub↔spoke (Bastion) traffic uses the peering directly while internet/east-west goes to the firewall.

**5.8 Why did you exclude a spoke's own range from its route table?**
Intra-spoke traffic should stay local. Azure's built-in route for the VNet's own address space handles it; overriding it would needlessly hairpin local traffic through the firewall (and could break it).

**5.9 What is forced tunneling?**
Routing all internet-bound traffic through an on-prem device or NVA (via `0.0.0.0/0`) instead of directly out. This project forces tunneling to the firewall, not to on-prem.

**5.10 Why was the route table attached to the workload subnet but not the private-endpoint subnet?**
The PE subnet should keep default routing so endpoints resolve/respond normally; only workload egress needs to be forced through the firewall.

---

## 6. Azure Firewall

**6.1 What is Azure Firewall?**
A managed, stateful, cloud-native network firewall-as-a-service with built-in high availability and scalability. It filters traffic by network rules (L3/4) and application rules (L7/FQDN).

**6.2 Network rules vs. application rules — difference?**
Network rules match on IP/port/protocol (L3/4) — used for inter-spoke and DNS here. Application rules match on FQDN over HTTP/HTTPS (L7) — used to allow specific outbound sites.

**6.3 What is a Firewall Policy and why use it?**
A separate, reusable resource that holds rule collection groups. It can be shared/inherited across multiple firewalls and is the modern way to manage rules (vs. classic inline rules).

**6.4 How are rules evaluated?**
By priority (lower number first), within and across rule collections; first match wins. DNAT rules are processed first, then network rules, then application rules. Anything not allowed is denied (implicit deny).

**6.5 In your traceroute, hop 1 showed 10.0.0.6 but your output said firewall IP 10.0.0.4 — explain.**
Azure Firewall runs multiple backend instances inside the AzureFirewallSubnet. `10.0.0.4` is the stable front-end private IP (the route next hop); the traceroute reply came from a backend instance (`.5`/`.6`/`.7`), all part of the same firewall.

**6.6 Firewall SKUs — Standard vs. Premium?**
Standard: L3/4 + L7 FQDN filtering, threat intel. Premium adds TLS inspection, IDPS, URL filtering, and web categories. This project uses Standard.

**6.7 What is SNAT, and does the firewall do it?**
Source NAT rewrites the source IP. The firewall SNATs outbound internet traffic to its public IP so return traffic comes back to it. It does not SNAT private (RFC1918) destinations by default.

**6.8 How does the firewall achieve high availability and scale?**
It's a managed service that auto-scales across instances and can be deployed across availability zones for resilience.

**6.9 What's the difference between Azure Firewall and an NSG?**
NSGs are basic, free, stateful L3/4 allow/deny rules on subnets/NICs. Azure Firewall is a managed L3–L7 appliance with FQDN filtering, threat intel, centralized policy, logging, and SNAT/DNAT. They're complementary (defense in depth).

**6.10 What is the firewall DNS proxy feature?**
The firewall can act as a DNS proxy so clients point DNS at the firewall; this enables FQDN filtering in network rules and consistent name resolution (including private endpoints) in more complex designs.

---

## 7. Azure Bastion

**7.1 What is Azure Bastion?**
A managed PaaS that provides secure RDP/SSH to VMs over TLS directly from the portal (or native client), without exposing the VMs' RDP/SSH ports or giving them public IPs.

**7.2 Why use Bastion here?**
To keep workload VMs free of public IPs. It's the only admin entry point, reducing the attack surface (no open 3389/22 to the internet).

**7.3 How can Bastion (in the hub) reach a VM in a spoke?**
Over the hub-to-spoke peering. Bastion connects to the VM's private IP across the peered networks.

**7.4 What subnet does Bastion require?**
`AzureBastionSubnet`, minimum /26.

**7.5 Bastion SKUs?**
Basic (portal RDP/SSH) and Standard (adds native client, IP-based connection, tunneling, scaling). This project uses Standard.

**7.6 Does Bastion need an NSG? What about the VM?**
Bastion's subnet has specific NSG requirements if you add one. The VMs here rely on default intra-VNet/peering allow rules; you'd add NSGs for stricter control.

---

## 8. VPN Gateway & Hybrid Connectivity

**8.1 What is a VPN Gateway?**
A VNet gateway that provides encrypted connectivity between Azure and on-premises (Site-to-Site IPsec) or remote users (Point-to-Site), over the public internet.

**8.2 Site-to-Site vs. Point-to-Site vs. ExpressRoute?**
S2S: IPsec tunnel between Azure and an on-prem VPN device. P2S: individual clients (laptops) connect via VPN client. ExpressRoute: a private, dedicated circuit (not over the internet) via a connectivity provider.

**8.3 Route-based vs. policy-based VPN?**
Route-based uses route tables/any-to-any selectors and supports more features (P2S, coexistence, multiple tunnels). Policy-based matches specific traffic selectors and is legacy/limited. This project uses route-based.

**8.4 What gateway SKU did you use and why `VpnGw1AZ` specifically?**
`VpnGw1AZ`. Azure has consolidated SKUs: as of late 2025 you can no longer create the non-AZ `VpnGw1–5` SKUs, only the AZ (availability-zone-supported) variants. So the AZ SKU is now required for new gateways.

**8.5 You hit an error creating the gateway — what was it and how did you fix it?**
The original `VpnGw1` (non-AZ) SKU was rejected because non-AZ VPN SKUs can no longer be created. I changed it to `VpnGw1AZ`. AZ gateways also require a zone-redundant Standard public IP, so I added `zones = ["1","2","3"]` to the gateway's public IP, and deployed to an availability-zone-enabled region (Central India).

**8.6 Why does region choice matter for the AZ gateway?**
Zone-redundant resources need a region with availability zones. South India (Chennai) has none, so I used Central India (Pune), which has three zones.

**8.7 Is the gateway actually carrying traffic in this project?**
Not by itself — it's deployed and shared via gateway transit, but there's no S2S/P2S connection configured, so no live traffic flows through it yet. It demonstrates the gateway-transit pattern; to make it functional I'd add a local network gateway + connection (S2S) or a P2S client configuration.

**8.8 How would you let your laptop connect to the VNets via this gateway?**
Add a Point-to-Site configuration: a client address pool, an auth method (Microsoft Entra ID or certificates), and the OpenVPN protocol, then install the VPN client. Gateway transit already lets P2S clients reach the spokes; you'd also need DNS handling (a forwarder/custom DNS) to resolve private endpoints over the tunnel.

**8.9 Why is the VPN gateway slow to deploy?**
It provisions dedicated gateway infrastructure (typically 20–45 minutes). It's also billed hourly, making it the most expensive/slow component.

---

## 9. Private Endpoints & Azure Private Link

**9.1 What is a private endpoint?**
A network interface with a private IP in your subnet that connects privately to a specific PaaS resource via Azure Private Link, removing the need for public exposure.

**9.2 What is Azure Private Link?**
The underlying service that maps a PaaS resource to a private endpoint, keeping traffic on the Microsoft backbone.

**9.3 Private endpoint vs. service endpoint — key differences? (common interview question)**
Service endpoints keep the traffic on the backbone but the service still uses its **public** IP and is reachable publicly (you restrict by VNet); they're free and VNet-scoped, not reachable from on-prem. Private endpoints give the service a **private IP** in your VNet, fully removing public exposure, reachable from on-prem/peered networks, but cost money and need private DNS. This project uses private endpoints.

**9.4 What is a sub-resource (`subresource_names`)?**
A PaaS resource can expose multiple endpoint types; you specify which one — e.g. `blob` (storage), `sqlServer` (SQL), `vault` (Key Vault). Using the wrong sub-resource fails to attach.

**9.5 Where did you place the private endpoints?**
In the data spoke's dedicated `snet-private-endpoints` subnet.

**9.6 How is the PaaS service's public access disabled?**
`public_network_access_enabled = false` on the storage account, SQL server, and Key Vault (plus Key Vault `network_acls` default-deny). After that, only the private endpoint can reach them.

**9.7 Once public access is off, how does Terraform (or you) manage the service's data plane?**
You need a path inside the VNet (e.g. a self-hosted agent/VM) or a temporary IP allow-list. Managing it from outside the VNet would fail — a real operational consideration.

**9.8 Can one private endpoint serve multiple regions or resources?**
A private endpoint maps to one resource + sub-resource. You create separate endpoints for different services (as done here: blob, SQL, vault).

---

## 10. Private DNS & Name Resolution

**10.1 How does DNS work in this project?**
VMs use Azure's default resolver (`168.63.129.16`), which is VNet-aware. The `privatelink.*` private DNS zones are linked to the VNets, so the resolver returns private endpoint IPs for the PaaS hostnames.

**10.2 Walk through resolving `mystorage.blob.core.windows.net` from a spoke VM.**
The public name is a CNAME to `mystorage.privatelink.blob.core.windows.net`. Because the VNet is linked to the `privatelink.blob.core.windows.net` private zone, Azure DNS returns the A record (the private endpoint IP, e.g. `10.2.1.x`). The VM connects privately.

**10.3 What creates the A record in the private zone?**
The `private_dns_zone_group` block on the private endpoint auto-registers the record. Omit it and the endpoint exists but the hostname won't resolve to the private IP.

**10.4 Which private DNS zones did you create?**
`privatelink.blob.core.windows.net`, `privatelink.database.windows.net`, `privatelink.vaultcore.azure.net`.

**10.5 Why link the zones to all three VNets (hub + both spokes)?**
So any VNet can resolve the private endpoints. Centralizing zones in the hub and linking everywhere is the standard enterprise pattern.

**10.6 What does `168.63.129.16` do?**
It's Azure's virtual public IP for the platform DNS resolver and host communication (DHCP, health probes). It's the default DNS for VNets and is aware of linked private zones.

**10.7 From your laptop, the same lookup returns a public IP — why?**
Your laptop isn't on the VNet and has no link to the private zone, so it falls through to the public DNS record (public IP). Since public access is disabled, the connection is then refused — proving the inside/outside distinction.

**10.8 If you connected via P2S VPN, would private DNS resolve automatically?**
No — a P2S client doesn't automatically use the VNet's resolver. You'd configure custom DNS / a DNS forwarder (or the firewall DNS proxy) so the client resolves `privatelink` names to private IPs.

**10.9 What's `registration_enabled = false` on the VNet links?**
It disables auto-registration of VM hostnames into the zone. These are `privatelink` resolution zones, not VM auto-registration zones, so it's off.

---

## 11. Security, NSGs & "No Public Access"

**11.1 What are the layers of security in this design?**
Network segmentation (hub-spoke), forced inspection (firewall + UDRs), no public IPs on VMs (Bastion), no public PaaS (private endpoints), and private DNS. NSGs can be added per subnet for defense in depth.

**11.2 NSG vs. Azure Firewall vs. UDR — what role does each play?**
NSG: stateful allow/deny at subnet/NIC (L3/4). Firewall: centralized L3–L7 inspection and logging. UDR: directs traffic to the firewall. They work together — UDR steers, firewall inspects, NSG provides micro-segmentation.

**11.3 What is the principle of least privilege here?**
Deny-by-default firewall rules (only specific FQDNs allowed), PaaS default-deny network ACLs, no public exposure, and admin access constrained to Bastion.

**11.4 How does this align with Zero Trust?**
Assume breach, verify explicitly, least privilege: no implicit trust between spokes (east-west is inspected), no public attack surface on data/VMs, and explicit allow-listing of egress.

**11.5 What would NSGs add that you don't have yet?**
Fine-grained, subnet/NIC-level allow/deny rules (e.g. only allow Bastion subnet to reach VM management ports), complementing the firewall.

**11.6 Where are secrets (passwords) handled in your Terraform?**
Marked `sensitive` and supplied via `terraform.tfvars`, which is gitignored. In production you'd source them from Key Vault or a secrets manager and use a remote backend with state encryption.

**11.7 Is Terraform state sensitive? How do you protect it?**
Yes — state can contain secrets in plaintext. Use a remote backend (e.g. Azure Storage) with encryption, access control, and state locking; never commit `*.tfstate`.

---

## 12. Terraform / Infrastructure as Code

**12.1 What is Terraform state and why does it matter?**
A file mapping your config to real resources. Terraform uses it to plan diffs and track resource identity. It must be protected and, for teams, stored remotely with locking.

**12.2 `plan` vs. `apply` vs. `destroy`?**
`plan` previews changes, `apply` makes them, `destroy` removes them. Always review a plan before applying.

**12.3 What is idempotency in Terraform?**
Re-running `apply` with no config change makes no changes — Terraform converges to the declared state regardless of current state.

**12.4 How does Terraform determine resource creation order?**
From references between resources (implicit dependencies). `depends_on` adds explicit ordering when there's no direct reference (e.g. the spoke peering waiting on the gateway).

**12.5 You used a "direct/flat" approach — what does that mean and why?**
Every resource is declared explicitly with no modules or `for_each` loops. It's maximally readable and maps one-to-one to the plan output — good for learning and clarity. The tradeoff is repetition (the two spokes are near-duplicate blocks).

**12.6 When would you refactor into modules?**
When the topology grows or repetition becomes error-prone — e.g. many spokes. A spoke module called with `for_each` over a map removes copy-paste and makes scaling a one-line change.

**12.7 What is `random_string` used for here?**
To generate globally-unique suffixes for resource names that require global uniqueness (storage account, SQL server, Key Vault).

**12.8 What does the `azurerm` provider `features {}` block do?**
It's a required (often empty) block enabling provider-level behavior toggles (e.g. how resource deletion is handled). v4 also requires `subscription_id`.

**12.9 How does Terraform authenticate to Azure here?**
Via the Azure CLI session (`az login`) and `subscription_id`. In CI you'd use a service principal or OIDC/workload identity federation.

**12.10 What's the difference between `count` and `for_each`?**
`count` creates N indexed copies (a list). `for_each` creates one per map/set key (keyed instances), which is more stable when adding/removing items. (This project's flat code uses neither for spokes, but `count`-style toggles or `for_each` are the scaling path.)

**12.11 How would you set up remote state and CI/CD for this?**
Azure Storage backend with state locking, a service principal or OIDC for auth, and a pipeline running `fmt`/`validate`/`plan` on PRs and `apply` on merge with approvals.

**12.12 What is `terraform fmt` and `terraform validate`?**
`fmt` standardizes formatting; `validate` checks syntax and internal consistency (not cloud-side correctness).

---

## 13. This Project's Specific Decisions & Troubleshooting

**13.1 Why Central India and not South India?**
The AZ VPN Gateway SKU requires availability zones (zone-redundant public IP). South India (Chennai) has no availability zones; Central India (Pune) has three.

**13.2 You hit "non-AZ SKUs are no longer supported for VPN gateways" — explain.**
Azure deprecated creation of non-AZ `VpnGw1–5` SKUs. Fix: use `VpnGw1AZ` and give the gateway a zone-redundant public IP, in an AZ-enabled region.

**13.3 Earlier your ping and curl from a spoke VM both failed — what was the root cause?**
The previous apply errored on the gateway SKU and stopped. Because the spoke peerings had `depends_on` on the gateway, the peerings never got created — so the spoke had no path to the hub firewall, and every firewall-routed destination dead-ended. Fixing the SKU and re-applying created the peerings and resolved it.

**13.4 How did you prove inter-spoke traffic goes through the firewall?**
`traceroute` from the spoke1 VM to the spoke2 VM showed the firewall (an address in the AzureFirewallSubnet) as the first hop, then the spoke2 VM as the second — i.e. spoke1 → firewall → spoke2, not a direct hop.

**13.5 Why did `az` fail inside the VM?**
The Azure CLI runs on your workstation, not on the VM (it wasn't installed there). That check is meant to run locally.

**13.6 How do you skip the VPN gateway if you don't need it?**
Remove `vpn.tf`, set `allow_gateway_transit`/`use_remote_gateways` to `false` in the peerings, and drop the gateway `depends_on`. This avoids the cost, the long provisioning time, and the AZ-SKU/region constraints.

**13.7 Why is the data spoke separate from the app spoke?**
Tiering/segmentation: keeping data services (and their private endpoints) isolated from the application tier, with east-west traffic between them inspected by the firewall.

---

## 14. Scenario / Design / Extension Questions

**14.1 How would you make this multi-region / highly available?**
Deploy hub-spoke per region, connect hubs (peering or via a transit), use zone-redundant SKUs (already using AZ gateway), geo-redundant storage, and Traffic Manager/Front Door for global entry. Consider Azure Virtual WAN for large-scale hub interconnection.

**14.2 When would you use Azure Virtual WAN instead of a manual hub-spoke?**
At scale (many hubs/regions, branch connectivity, transit routing). Virtual WAN is a managed hub with built-in any-to-any routing, simplifying large topologies.

**14.3 How would you inspect on-prem-to-spoke traffic with the firewall?**
Add UDRs on the GatewaySubnet sending spoke ranges to the firewall (and handle the firewall return routing), so traffic arriving via the gateway is also inspected.

**14.4 How would you add a DMZ / inbound web app securely?**
Use the firewall DNAT (or Application Gateway/WAF in the hub or a perimeter spoke) to publish the app, keeping backends private.

**14.5 How do you centralize logging and monitoring?**
Send Azure Firewall logs, NSG flow logs, and diagnostic settings to Log Analytics; build workbooks/alerts; optionally Microsoft Sentinel for SIEM.

**14.6 How would you enforce these standards across many teams?**
Azure Policy (deny public IPs, require private endpoints, enforce tags), landing zones (Azure Landing Zone / CAF), and RBAC.

**14.7 How would you handle private DNS at enterprise scale?**
Centralized private DNS zones in a connectivity subscription, linked to all VNets, often combined with the firewall as DNS proxy or a dedicated DNS forwarder VM, and Azure Policy to auto-create A records for private endpoints.

**14.8 How would you give developers access without public IPs or Bastion sprawl?**
A single Bastion in the hub serving all spokes (as here), or P2S VPN / Entra-authenticated access, plus just-in-time access and RBAC.

**14.9 What are the limits to watch (peering count, firewall throughput, gateway tunnels)?**
Per-VNet peering limits, firewall SKU throughput/SNAT port exhaustion, gateway SKU tunnel/throughput limits, and subnet sizing for the firewall/gateway subnets.

**14.10 How would you reduce cost in this design?**
Drop or downsize the gateway if unused, use the firewall Standard (or Basic) SKU, deallocate test VMs, use smaller VM sizes, and tear down non-prod environments. Azure Firewall and the gateway are the main cost drivers.

---

## 15. Cost, Operations & Monitoring

**15.1 What are the most expensive components here?**
Azure Firewall and the VPN Gateway (both billed hourly), then Bastion. PaaS and small VMs are comparatively cheap.

**15.2 How do you keep a lab like this cheap?**
`terraform destroy` when idle, skip the gateway if not needed, use burstable VM sizes (B-series), and avoid premium SKUs.

**15.3 How would you monitor the firewall?**
Enable diagnostic settings to Log Analytics, review allowed/denied flows, set alerts on denies or SNAT exhaustion, and visualize with a workbook.

**15.4 How do you handle drift (manual changes outside Terraform)?**
`terraform plan` detects drift; re-apply to reconcile. Lock down portal access via RBAC and use policy to discourage manual changes.

**15.5 What's your teardown process and any gotchas?**
`terraform destroy`. The gateway and DNS links can be slow or order-sensitive to delete; re-running destroy usually clears it. Confirm resource groups are gone afterward.

---

## 16. Quickfire / Rapid-Round

- **AzureFirewallSubnet minimum size?** /26.
- **GatewaySubnet minimum size?** /27.
- **Next hop type for firewall routing?** VirtualAppliance.
- **Which IP is the route next hop — firewall public or private?** Private.
- **Can spokes talk directly?** No — only via the hub firewall.
- **What makes a PaaS hostname resolve to a private IP?** Private DNS zone linked to the VNet + the endpoint's DNS zone group A record.
- **Service endpoint or private endpoint removes the public IP?** Private endpoint.
- **Flag that lets spoke-to-spoke through the firewall work?** `allow_forwarded_traffic`.
- **Why VpnGw1AZ?** Non-AZ VPN SKUs can no longer be created.
- **Azure platform DNS IP?** 168.63.129.16.
- **Tool that gives RDP/SSH without a public IP?** Azure Bastion.
- **What disables public access on the storage account?** `public_network_access_enabled = false`.
