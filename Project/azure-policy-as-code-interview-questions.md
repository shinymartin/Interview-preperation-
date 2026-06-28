# Azure Policy-as-Code — Interview Questions & Answers

A study guide of likely interview questions based on this project, organised from basic to advanced. Each question includes a concise answer you can adapt into your own words.

---

## 1. Project overview (start here)

**Q: Describe this project in two minutes.**
I built policy-as-code governance for an Azure subscription using Terraform. It's four custom Azure Policy definitions — enforce tagging, restrict regions, restrict VM SKUs, and deny public IPs — bundled into a single initiative, assigned to the subscription with a system-assigned managed identity, and wired to an auto-remediation task that fixes pre-existing non-compliant resources. State is stored remotely in Azure Storage. I deliberately scoped it to one subscription so it runs without management groups, which is the governance layer of a landing zone made approachable.

**Q: Why did you build it / what problem does it solve?**
Organisations need guardrails so resources are tagged, deployed only in approved regions, sized sensibly, and not exposed to the internet by default. Doing this by hand in the portal doesn't scale and isn't repeatable. Expressing it as code makes governance versioned, reviewable, and reproducible.

**Q: What does "policy-as-code" mean?**
It means defining governance rules in version-controlled code (here, Terraform) instead of configuring them manually in a UI. You get repeatability, peer review, history, and the ability to recreate the whole setup from scratch.

**Q: What is a landing zone, and how does this relate?**
A landing zone is a pre-configured, governed environment in Azure — usually spanning many subscriptions under management groups, with networking, identity, and policy baked in. This project extracts just the policy/governance slice and runs it in a single subscription, so you get the high-value part without tenant-level permissions.

---

## 2. Azure Policy basics

**Q: What is Azure Policy?**
A service that evaluates resources against rules and enforces organisational standards. It can block non-compliant deployments, audit them, or automatically modify/deploy to bring them into compliance.

**Q: How is Azure Policy different from Azure RBAC?**
RBAC controls *who* can perform *what actions* (identity and permissions). Policy controls *what the resources themselves are allowed to look like* (properties, locations, SKUs, tags). They're complementary: RBAC says "you may create a VM"; Policy says "but only in eastus and only these sizes."

**Q: What are the building blocks of Azure Policy?**
A policy definition (a single rule), an initiative / policy set (a bundle of definitions), and an assignment (attaching a definition or initiative to a scope like a management group, subscription, or resource group).

**Q: What scopes can a policy be assigned to?**
Management group, subscription, or resource group. This project assigns at the subscription scope.

**Q: What is a policy effect? Name the ones relevant here.**
The effect is what the rule does when a resource matches. The ones I used are `Deny` (block the request), `Modify` (change the resource, e.g. add a tag), and `Audit` (allow but flag as non-compliant). Others include Append, DeployIfNotExists, AuditIfNotExists, Disabled, DenyAction, and Manual.

**Q: Which effects support remediation, and why does that matter?**
Only `Modify` and `DeployIfNotExists`. They can change or deploy resources, so a remediation task can sweep existing non-compliant resources and fix them. A plain `Deny` only blocks new requests — there's nothing to remediate.

---

## 3. Project-specific design

**Q: Walk through your four policies and the effect each uses.**
Require-tag uses `Modify` so a missing tag is auto-added and existing resources can be remediated. Allowed-locations, allowed-VM-SKUs, and deny-public-IP all use `Deny` because the goal is to block, not fix.

**Q: Why did you use `Modify` for tagging instead of `Deny`?**
Deny would just reject any resource missing the tag, which is annoying and doesn't help existing resources. Modify adds the tag automatically on create and, combined with a remediation task, fixes resources that already exist. It's a better experience and demonstrates auto-remediation.

**Q: Why did you make the effect a parameter on each policy?**
So I can switch a policy from `Deny` to `Audit` without editing the rule — useful for safe testing or gradual rollout. You can observe what *would* be blocked before actually enforcing it, then flip it back to Deny.

**Q: How does the allowed-locations rule avoid blocking global resources?**
Some resources report their location as `global` (or use a non-standard value, like Azure AD B2C directories). The rule explicitly excludes `location == global` and the B2C resource type so they aren't wrongly denied for not being in the allowed-regions list.

**Q: How does the VM SKU policy identify the VM size?**
It matches on resource type `Microsoft.Compute/virtualMachines` and checks the alias `Microsoft.Compute/virtualMachines/sku.name` against the allowed list, denying anything not in it.

**Q: What is a policy alias?**
An alias maps a policy field to a specific property in a resource type's API (e.g. `Microsoft.Compute/virtualMachines/sku.name`). Aliases are how policies reach into resource-specific properties that aren't top-level fields like `location` or `type`.

---

## 4. Initiatives, parameters, and modes

**Q: What is an initiative and why use one here?**
An initiative (policy set) bundles multiple definitions so they can be assigned and managed together. I used one so all four rules are assigned in a single step and configured through one set of parameters — exactly how a landing zone ships governance.

**Q: How do initiative parameters connect to the member policies?**
The initiative declares its own parameters, and each member reference maps those down to the individual policy's parameters using `[parameters('...')]`. Think of the initiative as a control panel whose knobs feed values into the internal policies.

**Q: What is the policy `mode` (Indexed vs All)?**
`Indexed` evaluates only resource types that support tags and location — correct for tag and region policies. `All` evaluates everything, including resource groups and subscriptions. I used Indexed for tag/region/SKU policies and All where I only match on resource type.

**Q: What's a `reference_id` in an initiative?**
A friendly handle for each member policy inside the initiative. I needed the require-tag reference (`requireTag`) when creating the remediation task, so it knew which member policy to remediate.

---

## 5. Managed identity, RBAC, and remediation

**Q: Why does the assignment need a managed identity?**
Because the initiative contains a `Modify` policy. To actually change resources during remediation, Azure needs an identity to act as. A system-assigned managed identity is created with the assignment for this purpose.

**Q: System-assigned vs user-assigned managed identity — which did you use and why?**
System-assigned. It's created and tied to the assignment automatically and is simplest for a single assignment. A user-assigned identity makes sense when you want to share one identity across many assignments or manage its lifecycle independently.

**Q: Why does the managed identity need a role assignment?**
The identity has no permissions by default. To add tags during remediation it needs write access, so I granted it Contributor at the subscription scope. The Modify policy also declares `roleDefinitionIds` listing the roles it's allowed to use.

**Q: How would you apply least privilege to that identity?**
Use a narrower role than Contributor — for tag-only operations, Tag Contributor is sufficient. I noted Contributor as the reliable default but Tag Contributor as the least-privilege option.

**Q: What is a remediation task and why is it needed?**
A Modify/DeployIfNotExists policy only acts on resources at create/update time after assignment. Existing resources stay non-compliant until a remediation task sweeps them and applies the fix. It's how you bring the current estate into compliance, not just future deployments.

**Q: What's the difference between `ExistingNonCompliant` and `ReEvaluateCompliance` discovery modes?**
`ExistingNonCompliant` remediates resources already known to be non-compliant from the last evaluation. `ReEvaluateCompliance` re-checks compliance first, then remediates — more reliable right after a fresh assignment when evaluation may be stale.

---

## 6. Evaluation and behaviour

**Q: When does Azure Policy evaluate resources?**
On resource create/update requests (real-time, which is how Deny blocks happen), on a periodic background scan roughly every 24 hours, when the assignment changes, and on-demand when you trigger a scan.

**Q: A Deny didn't fire immediately after assignment — why?**
Policy assignments take time to propagate; it can be up to ~30 minutes before effects and compliance data are fully active. Deny on new resources usually kicks in within minutes, but it's not instant.

**Q: How can you force compliance to re-evaluate during testing?**
Trigger an on-demand scan (e.g. `az policy state trigger-scan`) instead of waiting for the periodic ~24-hour evaluation.

**Q: At what layer does a Deny actually block a resource?**
At the Azure Resource Manager control plane — the deployment request is rejected before the resource is created, returning a `RequestDisallowedByPolicy` error.

---

## 7. Terraform-specific

**Q: Why store Terraform state remotely instead of locally?**
Local state is fragile (you can lose it), can't be shared by a team, and isn't backed up. A remote backend (Azure Storage) keeps state durable, central, and supports locking. It signals real-world workflow rather than toy local Terraform.

**Q: How does state locking work with the azurerm backend?**
The Azure Storage backend uses a blob lease as a lock, so two `apply` runs can't corrupt state at the same time. If a run crashes mid-apply you may need to break the lock with `terraform force-unlock`.

**Q: The azurerm v4 provider needs the subscription ID — how did you supply it?**
Via the `ARM_SUBSCRIPTION_ID` environment variable rather than hard-coding it in the provider block, so the same code works across subscriptions and nothing sensitive is committed.

**Q: Why did you keep the policy rules in separate JSON files and load them with `file()`?**
Readability and separation of concerns. The raw Azure Policy JSON stays clean and editable on its own, and Terraform just references it, rather than embedding large JSON blobs inline.

**Q: What does `jsonencode()` do in your config and why use it?**
It converts an HCL object into a JSON string, which is what the policy `parameters` and `parameter_values` fields expect. Writing them as HCL objects is cleaner and less error-prone than hand-writing JSON.

**Q: How do you authenticate Terraform to Azure here?**
Through the Azure CLI login (`az login`) for local runs; the azurerm provider and backend pick up that credential. In a team or automated setup you'd typically use a service principal or workload identity instead.

**Q: How does Terraform know the order to create things (assignment before role assignment)?**
Mostly through implicit dependencies — the role assignment references the assignment's identity, so Terraform builds the graph automatically. I also added an explicit `depends_on` where needed to be safe.

**Q: What is idempotency and how does Terraform provide it?**
Running apply repeatedly converges to the same desired state without duplicating resources. Terraform compares state to config and only changes what differs, so re-running is safe.

**Q: How would you detect drift (someone changed a policy in the portal)?**
Run `terraform plan` — it compares real infrastructure against state and config and shows any differences, which you can then re-apply to correct.

---

## 8. Advanced / design trade-offs

**Q: How would you scale this to an entire organisation?**
Assign the initiative at a management group instead of a single subscription, so it cascades to all child subscriptions. That needs tenant-level permissions and a management-group hierarchy, which is exactly the landing-zone setup I deliberately scoped out for accessibility.

**Q: Custom policies vs Azure's built-in policies — when use each?**
Built-ins cover many common cases and need no maintenance, so use them when they fit. Custom definitions are for organisation-specific logic or when you want full control over parameters and behaviour. Several of mine mirror built-ins intentionally, to demonstrate writing the rules.

**Q: What happens if two policies conflict — e.g. a Deny and a Modify on the same resource?**
Deny takes precedence over Modify (the request is blocked, so there's nothing to modify). The Modify policy also has a `conflictEffect` setting to control behaviour when conflicts arise, which can fall back to audit.

**Q: How would you roll out a new Deny policy safely to avoid breaking teams?**
Start with the effect set to `Audit`, watch the compliance dashboard to see what would be blocked, communicate with affected teams, then switch to `Deny`. Parameterising the effect makes this a config change, not a code rewrite.

**Q: How do you handle exceptions — a resource that legitimately must break a rule?**
Use a policy exemption scoped to that resource or resource group, which excludes it from evaluation for a given assignment, ideally with an expiry and a documented reason.

**Q: How would you secure the remote state further?**
Restrict network access to the storage account, use Azure AD auth instead of access keys, enable soft delete/versioning on the blob, and lock down RBAC so only the pipeline/identity that runs Terraform can read or write state.

**Q: How would you test policy changes before applying to production?**
Apply to a test resource group or a non-prod subscription first, use Audit mode to preview impact, and review `terraform plan` output. You can also validate the JSON rules independently.

**Q: How could you extend this project?**
Add more controls (enforce HTTPS-only storage, require disk encryption, restrict allowed resource types), use DeployIfNotExists to auto-deploy missing configs (like diagnostic settings), add tag inheritance from resource group, or wrap it in a CI/CD pipeline for plan-on-PR / apply-on-merge.

---

## 9. Scenario & troubleshooting (things you actually hit)

**Q: You tried to create a compliant VM but it failed — what happened and how did you fix it?**
`az vm create` attaches a public IP by default, and my deny-public-IP policy blocked that public IP, failing the whole deployment with `RequestDisallowedByPolicy`. The fix was to create the VM with no public IP (`--public-ip-address ""`). It also confirmed the policy was working as intended.

**Q: A remediation task ran but fixed 0 resources — how do you debug?**
Check that compliance has actually been evaluated (trigger a scan), confirm the managed identity has the right role, verify the `policy_definition_reference_id` matches the Modify member, and check `location_filters` aren't excluding the resources.

**Q: How do you read a policy denial error?**
The `RequestDisallowedByPolicy` message names the policy assignment, the specific definition, and the evaluated condition — so you can see exactly which rule blocked the resource and why.

**Q: Remediation fails with an authorization error — likely cause?**
The assignment's managed identity is missing the required role. Confirm the role assignment (Contributor or Tag Contributor) succeeded for that identity's principal ID.

---

## 10. Quick-fire conceptual checks

**Q: Definition vs initiative vs assignment in one line each?**
Definition = one rule. Initiative = a bundle of rules. Assignment = applying a rule or bundle to a scope.

**Q: Does Azure Policy cost anything?**
No — definitions, initiatives, assignments, and remediation are free. You only pay for resources like the state storage account or any test VMs.

**Q: Can a Deny policy fix existing non-compliant resources?**
No. Deny only blocks new/updated requests. Fixing existing resources requires Modify or DeployIfNotExists plus a remediation task.

**Q: What's the difference between compliant and non-compliant in the dashboard?**
Compliant resources satisfy the policy; non-compliant ones violate it (or, for audit effects, are flagged without being blocked). The dashboard shows counts per policy and an overall percentage.

**Q: Why one subscription instead of management groups?**
Management groups require tenant-level rights most people don't have on a personal account. Scoping to one subscription keeps the high-value governance layer accessible to anyone.
