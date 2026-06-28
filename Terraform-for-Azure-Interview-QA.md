# Terraform for Azure — Interview Questions & Answers

> A complete, interview-ready guide covering basic → advanced Terraform concepts as applied to **Microsoft Azure**. Every answer is written in plain language, with jargon and short forms expanded in parentheses, and includes practical / real-world context wherever it helps.

---

## 📊 Total Questions & Answers: **120**

| # | Sub-Topic | Questions |
|---|-----------|-----------|
| 1 | Terraform & IaC Fundamentals | 1–10 |
| 2 | Azure Provider & Authentication | 11–19 |
| 3 | Core Workflow & CLI Commands | 20–29 |
| 4 | Variables, Outputs & Data Sources | 30–38 |
| 5 | State Management | 39–50 |
| 6 | Modules | 51–58 |
| 7 | Meta-Arguments & Expressions | 59–68 |
| 8 | Provisioners, Lifecycle & Drift | 69–75 |
| 9 | Secrets & Security | 76–82 |
| 10 | CI/CD & Team Collaboration | 83–89 |
| 11 | Advanced Topics & Troubleshooting | 90–100 |
| 12 | Real-World Scenario Questions | 101–112 |
| 13 | Best Practices (Rapid-Fire) | 113–120 |

---

## 1. Terraform & IaC Fundamentals

### Q1. What is Terraform?
**Answer:** Terraform is an open-source **IaC (Infrastructure as Code)** tool built by HashiCorp. Instead of clicking through the Azure Portal to create resources manually, you describe your desired infrastructure (virtual machines, networks, databases, etc.) in human-readable configuration files. Terraform then talks to the cloud provider's **API (Application Programming Interface)** and creates, updates, or deletes resources to match what you declared.

It is **declarative**, meaning you describe the *end state* you want ("I want 3 VMs"), not the step-by-step instructions to get there. Terraform figures out the steps. It is also **cloud-agnostic** — the same tool works with Azure, AWS, GCP, Kubernetes, and hundreds of other platforms via **providers**.

> **Real-world note:** As of August 2023, Terraform moved from the open-source MPL (Mozilla Public License) to the BUSL (Business Source License). A community fork called **OpenTofu** (under the Linux Foundation) remains fully open-source and is largely a drop-in replacement. Interviewers sometimes ask if you're aware of this.

---

### Q2. What is IaC (Infrastructure as Code) and why does it matter?
**Answer:** **IaC (Infrastructure as Code)** is the practice of managing and provisioning infrastructure using machine-readable definition files rather than manual processes or interactive tools.

Benefits:
- **Repeatability:** The same code produces the same environment every time — no "it works on my machine" for infrastructure.
- **Version control:** Configurations live in Git, so you get history, code review, and rollback.
- **Speed & automation:** Spin up entire environments in minutes via CI/CD.
- **Documentation:** The code *is* the documentation of what exists.
- **Disaster recovery:** Rebuild an environment from scratch if a region fails.

---

### Q3. What is the difference between declarative and imperative IaC? Where does Terraform fit?
**Answer:**
- **Imperative** ("how"): You write step-by-step commands — *create a resource group, then create a VNet, then create a subnet*. Shell scripts and Azure CLI scripts are imperative.
- **Declarative** ("what"): You describe the final desired state and let the tool determine the steps. Terraform, ARM templates, and Bicep are declarative.

Terraform is **declarative**. You declare what you want, and Terraform compares it to reality (the state file) and makes only the changes needed to converge.

---

### Q4. How does Terraform compare to Azure-native IaC tools like ARM templates and Bicep?
**Answer:**
- **ARM (Azure Resource Manager) templates:** Azure's native JSON-based IaC. Verbose and Azure-only.
- **Bicep:** A friendlier DSL (Domain-Specific Language) that compiles down to ARM. Cleaner syntax, Azure-only, no separate state file (it reads live state from Azure).
- **Terraform:** Multi-cloud, uses HCL (HashiCorp Configuration Language), maintains its own **state file**, has a huge module/provider ecosystem.

**When to choose Terraform over Bicep:** multi-cloud or hybrid environments, existing Terraform skills/modules, or when you want a consistent workflow across clouds. **When Bicep wins:** pure-Azure shops wanting day-one support for the newest Azure features and no state file to manage.

---

### Q5. What is HCL?
**Answer:** **HCL (HashiCorp Configuration Language)** is the language used to write Terraform configurations (`.tf` files). It's designed to be both human-readable and machine-friendly. A basic block looks like:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-demo"
  location = "East US"
}
```

The structure is `block_type "resource_type" "local_name" { arguments }`. HCL supports variables, expressions, functions, loops, and conditionals. You can also write Terraform config in JSON, but HCL is the standard.

---

### Q6. What is a Provider in Terraform?
**Answer:** A **provider** is a plugin that lets Terraform talk to a specific platform's API. For Azure, the main provider is **`azurerm`** (Azure Resource Manager). Each provider exposes **resources** (things you can create/manage) and **data sources** (things you can read).

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

The `features {}` block is **mandatory** for the azurerm provider (even when empty) — a common gotcha.

---

### Q7. What are the main Azure-related Terraform providers?
**Answer:**
- **`azurerm`:** The primary provider for Azure infrastructure (VMs, networks, storage, databases, etc.).
- **`azuread`:** Manages **Microsoft Entra ID** (formerly Azure AD / Azure Active Directory) — users, groups, service principals, app registrations.
- **`azapi`:** A "pass-through" provider that talks directly to the Azure Resource Manager REST API. Useful for **brand-new Azure features** not yet supported in `azurerm`, or preview resources.
- **`azurestack`:** For Azure Stack (on-premises Azure) deployments.

> **Real-world tip:** When an interviewer asks "what do you do if a resource isn't supported in azurerm yet?", the expected answer is **use the `azapi` provider** (or `azurerm`'s `azurerm_resource_group_template_deployment` for ARM, as a fallback).

---

### Q8. What is a Resource vs a Data Source?
**Answer:**
- A **resource** is something Terraform *manages* — it creates, updates, and destroys it. Example: `azurerm_virtual_machine`.
- A **data source** is something Terraform *reads but does not manage* — used to fetch information about existing infrastructure. Example: reading an existing resource group created by another team.

```hcl
# Resource: Terraform owns this
resource "azurerm_storage_account" "sa" { ... }

# Data source: just reading existing info
data "azurerm_resource_group" "existing" {
  name = "rg-shared-network"
}
```

---

### Q9. What is the Terraform state file and why is it critical?
**Answer:** The **state file** (`terraform.tfstate`) is a JSON file where Terraform records what it has created and maps your configuration to real-world resources (storing each resource's Azure ID). It is Terraform's "source of truth" about the real world.

Why it matters:
- It lets Terraform know what already exists, so `plan` can compute the *difference* between your code and reality.
- It stores resource metadata and dependency info.
- It can contain **sensitive data** (passwords, keys) in plain text — which is why it must be stored securely.

Losing or corrupting the state file can make Terraform think nothing exists (and try to recreate everything) — so protecting it is essential.

---

### Q10. What are the core steps of the Terraform workflow?
**Answer:** The standard lifecycle is **Write → Init → Plan → Apply**:
1. **`terraform init`** — Downloads providers/modules and sets up the backend.
2. **`terraform plan`** — Shows a preview (dry run) of what will be created, changed, or destroyed.
3. **`terraform apply`** — Executes the changes to reach the desired state.
4. **`terraform destroy`** — Tears down everything Terraform manages (used in dev/test or cleanup).

Supporting commands include `terraform validate` (syntax check) and `terraform fmt` (auto-format code).

---

## 2. Azure Provider & Authentication

### Q11. What are the different ways Terraform can authenticate to Azure?
**Answer:** There are several methods, chosen based on whether you run locally or in a pipeline:
1. **Azure CLI** — You run `az login` first; Terraform reuses that session. Best for local development.
2. **Service Principal with Client Secret** — A non-human identity with an app ID and secret. Common for CI/CD.
3. **Service Principal with Client Certificate** — Same as above but using a certificate instead of a secret (more secure).
4. **Managed Identity** — When Terraform runs on an Azure resource (like a VM or Azure DevOps agent), it uses that resource's identity. No secrets to manage.
5. **OIDC / Workload Identity Federation** — Pipelines (GitHub Actions, Azure DevOps) get a short-lived token without storing any long-lived secret. **This is the modern best practice for CI/CD.**

---

### Q12. What is a Service Principal (SP) and how is it used with Terraform?
**Answer:** A **Service Principal (SP)** is a non-human identity (like a "service account") in **Microsoft Entra ID** that applications use to authenticate and access Azure resources. It has an **Application (Client) ID**, a **Tenant ID**, and either a **client secret** or **certificate**.

For Terraform, you assign the SP an Azure role (commonly **Contributor** scoped to a subscription or resource group), then provide its credentials via environment variables:

```bash
export ARM_CLIENT_ID="xxxx"
export ARM_CLIENT_SECRET="xxxx"
export ARM_SUBSCRIPTION_ID="xxxx"
export ARM_TENANT_ID="xxxx"
```

Using environment variables (the `ARM_*` prefix) keeps secrets out of your `.tf` code.

---

### Q13. Why is OIDC (Workload Identity Federation) preferred over client secrets in pipelines?
**Answer:** **OIDC (OpenID Connect)** based **Workload Identity Federation** lets your CI/CD pipeline exchange a trusted token for a short-lived Azure access token — **without storing any long-lived secret** anywhere.

Advantages:
- **No secret to leak or rotate** — eliminates the risk of a stolen `ARM_CLIENT_SECRET`.
- **Short-lived tokens** reduce the blast radius if compromised.
- Trust is configured once via a **federated credential** on the app registration.

This is now the recommended approach for GitHub Actions and Azure DevOps connecting to Azure.

---

### Q14. How do you configure the azurerm provider for a specific subscription?
**Answer:** You can pin the subscription either via environment variable (`ARM_SUBSCRIPTION_ID`) or directly in the provider block:

```hcl
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}
```

> **Note:** As of azurerm provider **v4.x**, specifying `subscription_id` is **required** (it no longer auto-detects from CLI silently for apply). This is a frequently-tested recent change.

---

### Q15. How do you manage multiple Azure subscriptions in a single Terraform configuration?
**Answer:** Use **provider aliases**. You declare multiple `provider` blocks with different `alias` values and point resources to the one they need:

```hcl
provider "azurerm" {
  features {}
  subscription_id = var.prod_sub
}

provider "azurerm" {
  alias           = "hub"
  features {}
  subscription_id = var.hub_sub
}

resource "azurerm_virtual_network" "hub_vnet" {
  provider = azurerm.hub
  # ...
}
```

This is common in **hub-and-spoke** network designs where networking lives in a separate "hub" subscription.

---

### Q16. What is the `features {}` block in the azurerm provider?
**Answer:** It's a mandatory block (even if empty) that controls provider-level behaviors. You can customize how the provider handles certain operations, such as whether to purge soft-deleted Key Vaults or how to handle resource group deletion:

```hcl
provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = true
    }
    key_vault {
      purge_soft_delete_on_destroy = true
    }
  }
}
```

Forgetting the `features {}` block entirely causes an init/plan error — a classic beginner mistake.

---

### Q17. How do you pin provider versions and why is it important?
**Answer:** You set version constraints in `required_providers`:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.85"  # allows 3.85.x but not 4.0
    }
  }
}
```

Operators:
- `=` exact, `>=` minimum, `~>` pessimistic ("allow rightmost increments").

**Why pin?** Provider upgrades can introduce breaking changes. Pinning ensures everyone on the team and every pipeline run uses the same version, producing consistent plans. The `.terraform.lock.hcl` lock file records the exact versions and hashes — commit it to Git.

---

### Q18. What is the `.terraform.lock.hcl` file?
**Answer:** It's the **dependency lock file** that records the exact provider versions and their checksums that Terraform selected during `init`. It guarantees that everyone — teammates and CI — uses identical provider versions, preventing "works for me" drift. You **commit it to source control**. To update it intentionally, run `terraform init -upgrade`.

---

### Q19. How would you grant Terraform least-privilege access in Azure?
**Answer:** Instead of giving the Service Principal broad **Owner** or subscription-wide **Contributor**, you:
- Scope the role assignment to the **specific resource group(s)** Terraform manages.
- Use **Contributor** for resource management but a **custom role** if it only needs a subset of actions.
- Use **separate identities per environment** (dev SP can't touch prod).
- Grant **User Access Administrator** or specific role-assignment permissions only if Terraform itself needs to create RBAC (Role-Based Access Control) assignments.

This follows the **principle of least privilege** and limits blast radius if credentials leak.

---

## 3. Core Workflow & CLI Commands

### Q20. What does `terraform init` do?
**Answer:** It initializes a working directory. Specifically it:
- Downloads and installs the **providers** referenced in your config.
- Downloads any **modules** you reference.
- Configures the **backend** (where state is stored).
- Creates the `.terraform` directory and the `.terraform.lock.hcl` file.

You must run it before `plan`/`apply`, and re-run it after adding providers/modules or changing the backend (`terraform init -reconfigure` or `-migrate-state`).

---

### Q21. What is the difference between `terraform plan` and `terraform apply`?
**Answer:**
- **`plan`** is a **dry run** — it shows what Terraform *would* do (create `+`, change `~`, destroy `-`) without making any changes. It's safe and read-only.
- **`apply`** actually executes the changes. By default it shows the plan again and asks for confirmation; with `-auto-approve` it skips the prompt (used in pipelines).

A best practice is to save the plan (`terraform plan -out=tfplan`) and apply exactly that file (`terraform apply tfplan`) so the applied changes match what was reviewed.

---

### Q22. What does `terraform validate` do, and how is it different from `plan`?
**Answer:** `terraform validate` checks the **syntax and internal consistency** of your configuration (correct argument names, valid references, type correctness) **without contacting Azure**. It's fast and offline. `plan`, by contrast, **reaches out to the Azure API** and compares against real state to compute actual changes. Use `validate` for quick CI checks; use `plan` to see real changes.

---

### Q23. What is `terraform fmt`?
**Answer:** It auto-formats `.tf` files to the canonical HCL style (consistent indentation, alignment, spacing). Teams run `terraform fmt -check` in CI to enforce consistent formatting and reject unformatted code.

---

### Q24. What does `terraform destroy` do, and how do you destroy a single resource?
**Answer:** `terraform destroy` tears down **all** resources managed by the current configuration/state. To remove just one resource, the modern approach is to **delete it from your code and run `apply`**. To target a single resource imperatively (use sparingly), you can use:

```bash
terraform destroy -target=azurerm_storage_account.sa
```

`-target` is meant for troubleshooting, not routine use, because it can leave state in an inconsistent shape.

---

### Q25. What is `terraform refresh` and is it still recommended?
**Answer:** `terraform refresh` updates the state file to match the real-world infrastructure (detecting **drift**). The standalone command is **deprecated** because it mutates state without showing you a plan. The recommended way to see drift is:

```bash
terraform plan -refresh-only
terraform apply -refresh-only
```

This shows you what changed in reality before updating state.

---

### Q26. What does `terraform show` do?
**Answer:** It displays human-readable output of either the current state or a saved plan file:
- `terraform show` → current state.
- `terraform show tfplan` → contents of a saved plan.
- `terraform show -json` → machine-readable output for tooling/automation.

---

### Q27. What is `terraform output` used for?
**Answer:** It prints the values of **output variables** defined in your config (e.g., a VM's public IP, a storage account's connection string). Useful for passing values to scripts or other tools:

```bash
terraform output                       # all outputs
terraform output storage_account_name  # one output
terraform output -json                 # JSON for automation
```

Mark sensitive outputs with `sensitive = true` so they don't appear in logs.

---

### Q28. What is `terraform console`?
**Answer:** An interactive REPL (Read-Eval-Print Loop) for evaluating Terraform expressions and functions against your current state. Great for testing interpolations, functions, and `for` expressions before putting them in code:

```bash
$ terraform console
> upper("east us")
"EAST US"
> length(var.subnet_list)
3
```

---

### Q29. How do you target specific resources during plan/apply?
**Answer:** Use the `-target` flag to limit operations to specific resources or modules:

```bash
terraform apply -target=module.network -target=azurerm_resource_group.rg
```

It's intended for **recovering from mistakes or specific debugging**, not normal workflow, because it bypasses Terraform's full dependency graph and can produce a partial, inconsistent apply.

---

## 4. Variables, Outputs & Data Sources

### Q30. What are input variables and how do you declare them?
**Answer:** **Input variables** parameterize your configuration so the same code works across environments. You declare them with type, description, default, and optional validation:

```hcl
variable "location" {
  type        = string
  description = "Azure region for resources"
  default     = "East US"
}
```

You then reference them as `var.location`.

---

### Q31. What are the different ways to assign values to variables, and what is the precedence order?
**Answer:** Values can come from (highest to lowest precedence):
1. **Command line** `-var` flags and `-var-file`.
2. **`*.auto.tfvars` / `*.auto.tfvars.json`** files (alphabetical).
3. **`terraform.tfvars`** file.
4. **Environment variables** prefixed `TF_VAR_` (e.g., `TF_VAR_location`).
5. **Default** value in the variable declaration.

If none are provided and there's no default, Terraform prompts interactively.

---

### Q32. What is a `.tfvars` file and when do you use multiple of them?
**Answer:** A `.tfvars` file holds variable *values* (not declarations). You commonly keep one per environment:

```bash
terraform apply -var-file="prod.tfvars"
terraform apply -var-file="dev.tfvars"
```

This lets a single codebase deploy to dev, staging, and prod with different sizing, regions, and counts. (Don't commit secrets in tfvars — use Key Vault or pipeline secrets.)

---

### Q33. What variable types does Terraform support?
**Answer:**
- **Primitive:** `string`, `number`, `bool`.
- **Collection:** `list(...)`, `set(...)`, `map(...)`.
- **Structural:** `object({...})`, `tuple([...])`.
- **`any`** for flexible typing.

```hcl
variable "vm_config" {
  type = object({
    size  = string
    count = number
    tags  = map(string)
  })
}
```

Strong typing catches errors early and self-documents expected input.

---

### Q34. How do you validate input variables?
**Answer:** Use a `validation` block with a `condition` and `error_message`:

```hcl
variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

This fails fast with a clear message instead of producing an invalid Azure resource name later.

---

### Q35. What are output variables and why are they useful?
**Answer:** **Outputs** expose values from your configuration after apply — like a VM IP, a resource ID, or a connection string. They're useful for:
- Displaying important values to the user.
- Passing data **between modules** (a child module's output becomes available to the parent).
- Sharing data **between root configurations** via `terraform_remote_state`.

```hcl
output "public_ip" {
  value       = azurerm_public_ip.pip.ip_address
  description = "Public IP of the load balancer"
}
```

---

### Q36. What does marking a variable or output as `sensitive` do?
**Answer:** Setting `sensitive = true` tells Terraform to **redact the value in CLI output and plan/apply logs** (shows `(sensitive value)`). It prevents accidental exposure in console output and CI logs. **Important caveat:** it is **still stored in plain text in the state file**, so the state must still be secured and encrypted.

---

### Q37. What is a data source and give a practical Azure example.
**Answer:** A **data source** reads information about existing infrastructure without managing it. Practical example: another team created a shared VNet (Virtual Network); you reference it to deploy a subnet into it:

```hcl
data "azurerm_virtual_network" "shared" {
  name                = "vnet-shared"
  resource_group_name = "rg-network"
}

resource "azurerm_subnet" "app" {
  name                 = "snet-app"
  resource_group_name  = data.azurerm_virtual_network.shared.resource_group_name
  virtual_network_name = data.azurerm_virtual_network.shared.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

---

### Q38. How do you fetch the current Azure client / subscription details dynamically?
**Answer:** Use the `azurerm_client_config` data source, which returns the tenant ID, subscription ID, and object ID of the identity Terraform is running as. This is commonly used when setting Key Vault access policies:

```hcl
data "azurerm_client_config" "current" {}

# later: object_id = data.azurerm_client_config.current.object_id
```

This avoids hard-coding IDs and keeps the config portable across subscriptions.

---

## 5. State Management

### Q39. Why should you never store state locally for team / production use?
**Answer:** Local state (`terraform.tfstate` on your laptop) causes problems:
- **No collaboration:** Teammates can't see or share your state.
- **No locking:** Two people applying simultaneously can corrupt state.
- **Risk of loss:** A lost laptop = lost state = orphaned resources.
- **Security:** Secrets in plain text sitting on a local disk.

For teams, you use a **remote backend** with locking and encryption.

---

### Q40. How do you configure a remote backend in Azure?
**Answer:** Use the **`azurerm` backend**, which stores state as a blob in an **Azure Storage Account**:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "sttfstateprod001"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

You create the storage account/container **before** init (a chicken-and-egg setup, often bootstrapped manually or by a separate "bootstrap" config). Then `terraform init` migrates/uses that backend.

---

### Q41. How does state locking work with the azurerm backend?
**Answer:** State locking prevents two people/pipelines from modifying state at the same time. With the azurerm backend, locking is **automatic and built-in** — it uses an **Azure Blob lease** on the state blob. When someone runs apply, Terraform acquires the lease (lock); others must wait. Unlike AWS (which needs a separate DynamoDB table), Azure needs **no extra infrastructure** for locking. If a process crashes and leaves a stale lock, you can force-release with `terraform force-unlock <LOCK_ID>` (carefully).

---

### Q42. How do you secure the Terraform state stored in Azure Storage?
**Answer:** Best practices:
- **Encryption at rest** — Azure Storage encrypts blobs by default; optionally use customer-managed keys (CMK) in Key Vault.
- **Enable blob versioning & soft delete** — recover from accidental corruption/deletion.
- **Restrict network access** — private endpoints / firewall rules so only pipelines and approved IPs can reach the storage account.
- **RBAC + minimal access** — only the Terraform identity can read/write the container.
- **Disable public access** on the container.
- **Separate storage accounts per environment** so prod state is isolated.

---

### Q43. How do you migrate Terraform state from local to a remote Azure backend?
**Answer:** Step by step:
1. Create the Azure Storage Account and container.
2. Add the `backend "azurerm"` block to your config.
3. Run `terraform init`. Terraform detects the new backend and asks: *"Do you want to copy existing state to the new backend?"*
4. Answer **yes** (or use `terraform init -migrate-state`).

Terraform copies the local state to the remote backend. Verify with `terraform plan` (should show no changes). Then delete the local `terraform.tfstate`.

---

### Q44. What are the common `terraform state` subcommands?
**Answer:**
- `terraform state list` — list resources in state.
- `terraform state show <addr>` — show details of one resource.
- `terraform state mv <src> <dst>` — rename/move a resource in state (refactoring without recreating).
- `terraform state rm <addr>` — remove a resource from state (Terraform "forgets" it, but the real resource stays).
- `terraform state pull` — output the raw remote state.
- `terraform state push` — overwrite remote state (dangerous — last resort).

---

### Q45. Give a real scenario where you'd use `terraform state mv`.
**Answer:** Suppose you refactor a resource — moving it into a module or renaming its local name from `azurerm_storage_account.sa` to `azurerm_storage_account.data`. Without intervention, Terraform sees the old address as "to be destroyed" and the new one as "to be created," causing **unnecessary recreation**. Running:

```bash
terraform state mv azurerm_storage_account.sa azurerm_storage_account.data
```

tells Terraform they're the same resource, so it just updates the state address and plans **no changes**. (Modern alternative: `moved` blocks — see Q66.)

---

### Q46. When would you use `terraform state rm`?
**Answer:** When you want Terraform to **stop managing** a resource without deleting it from Azure. Examples:
- Handing a resource over to another team/state.
- Removing a resource you imported by mistake.
- Splitting one state into multiple.

After `state rm`, the resource still exists in Azure but Terraform no longer tracks it. If you later run `apply` and the resource is still in your code, Terraform will try to **recreate** it — so usually you also remove it from the code.

---

### Q47. How do you import existing Azure resources into Terraform?
**Answer:** Two approaches:

**1. CLI import (classic):**
```bash
terraform import azurerm_resource_group.rg /subscriptions/<sub-id>/resourceGroups/rg-existing
```
You must first write a matching `resource` block, then import maps the real resource into state.

**2. Import block (declarative, Terraform 1.5+):**
```hcl
import {
  to = azurerm_resource_group.rg
  id = "/subscriptions/<sub-id>/resourceGroups/rg-existing"
}
```
Then `terraform plan -generate-config-out=generated.tf` can even auto-generate the resource configuration. This is reviewable, repeatable, and pipeline-friendly.

---

### Q48. What happens if the state file gets deleted or corrupted? How do you recover?
**Answer:** If state is lost, Terraform no longer knows what it manages and may try to recreate everything (causing conflicts/duplication).

Recovery options:
- **Restore from backup** — Azure blob versioning/soft delete makes this the first and best option.
- **Re-import** every resource with `terraform import` / import blocks (tedious but works).
- For a single corrupted apply, restore the `.tfstate.backup` file Terraform creates automatically.

**Prevention:** remote backend + versioning + soft delete + restricted access. This is why interviewers stress protecting state.

---

### Q49. What is `terraform_remote_state` and when do you use it?
**Answer:** It's a **data source** that reads outputs from *another* Terraform configuration's state. Used to share data across separately-managed stacks — e.g., a "networking" stack exposes the VNet ID, and an "app" stack consumes it:

```hcl
data "terraform_remote_state" "network" {
  backend = "azurerm"
  config = {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "sttfstate001"
    container_name       = "tfstate"
    key                  = "network.terraform.tfstate"
  }
}

# use: data.terraform_remote_state.network.outputs.vnet_id
```

This enables a **layered / modular** architecture without one giant state file.

---

### Q50. What are the pros and cons of splitting state into multiple smaller states?
**Answer:**
**Pros:** Smaller blast radius (a mistake affects only one stack), faster plans/applies, clearer ownership boundaries, parallel team workflows, less locking contention.
**Cons:** More complex cross-stack data sharing (`terraform_remote_state`), more backends to manage, harder to reason about ordering/dependencies, possible duplication.

A common pattern is splitting by **layer** (networking, shared services, app) and by **environment**. The trade-off interviewers look for: balance isolation against operational overhead.

---

## 6. Modules

### Q51. What is a Terraform module?
**Answer:** A **module** is a reusable, self-contained package of Terraform configuration — a folder with `.tf` files (variables, resources, outputs). Every Terraform configuration is technically a module (the **root module**). You create **child modules** to encapsulate and reuse patterns, e.g., a "standard VNet with subnets" module used by every project.

---

### Q52. What is the difference between the root module and child modules?
**Answer:**
- **Root module** — the directory where you run `terraform` commands. It's the entry point.
- **Child module** — any module called by another module using a `module` block. Child modules receive **input variables**, create resources, and return **outputs** to the caller.

```hcl
module "network" {
  source         = "./modules/network"
  vnet_name      = "vnet-app"
  address_space  = ["10.0.0.0/16"]
}
```

---

### Q53. Where can modules be sourced from?
**Answer:** The `source` argument supports many locations:
- **Local path:** `./modules/network`.
- **Terraform Registry (public):** `Azure/network/azurerm`.
- **Private registry:** Terraform Cloud/Enterprise or Azure DevOps.
- **Git:** `git::https://github.com/org/repo.git//modules/vnet?ref=v1.2.0`.
- **HTTP URLs / storage buckets.**

Always **pin a version/ref** (`?ref=v1.2.0` or `version = "x.y.z"`) so module changes don't silently break you.

---

### Q54. How do you pass data into and get data out of a module?
**Answer:**
- **In:** via the module block's arguments, which map to the module's `variable` declarations.
- **Out:** via the module's `output` blocks, accessed by the caller as `module.<name>.<output>`.

```hcl
module "network" {
  source    = "./modules/network"
  vnet_name = "vnet-app"
}

resource "azurerm_subnet" "x" {
  virtual_network_name = module.network.vnet_name   # using module output
}
```

---

### Q55. How do you version Terraform modules and why?
**Answer:** Pin module versions to ensure stability:
- **Registry modules:** `version = "~> 3.0"` in the module block.
- **Git modules:** use a tag ref `?ref=v2.1.0`.

Versioning means upgrading a shared module is a **deliberate, reviewable** action — not something that breaks 20 teams the moment someone pushes to `main`. Follow **SemVer (Semantic Versioning)**: major = breaking, minor = feature, patch = fix.

---

### Q56. How would you structure a large enterprise Terraform repository?
**Answer:** A common layout:
```
├── modules/                 # reusable building blocks
│   ├── network/
│   ├── compute/
│   └── keyvault/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── dev.tfvars
│   ├── staging/
│   └── prod/
└── global/                  # shared/global resources
```
Each environment has its **own state/backend key**, calls the shared modules, and supplies environment-specific variables. This gives reuse (modules) + isolation (per-env state).

---

### Q57. What makes a good, reusable module?
**Answer:**
- **Single responsibility** — does one thing well (e.g., "a storage account with sensible defaults").
- **Sensible defaults** but **configurable** via variables.
- **Strongly typed, validated inputs.**
- **Useful outputs** (IDs, names, connection info).
- **No hard-coded environment specifics** (names, regions, subscription IDs).
- **Documentation** (README, examples) and **pinned provider requirements**.
- **No provider blocks inside** (the caller passes providers) for reusability.

---

### Q58. Should a reusable module contain its own `provider` block?
**Answer:** **No (generally).** Reusable/shared modules should **not** declare `provider` blocks — they should only declare `required_providers` (provider *requirements*). The **root module** configures and passes providers. Hard-coding providers inside a child module breaks reuse (e.g., it can't be used with provider aliases for multi-subscription scenarios). Pass providers explicitly via the `providers` argument when needed.

---

## 7. Meta-Arguments & Expressions

### Q59. What is the `count` meta-argument?
**Answer:** `count` creates multiple instances of a resource based on a number. Each instance is addressed by index (`[0]`, `[1]`, …):

```hcl
resource "azurerm_public_ip" "pip" {
  count               = 3
  name                = "pip-${count.index}"
  location            = var.location
  resource_group_name = var.rg
  allocation_method   = "Static"
}
```

**Caveat:** because instances are index-based, removing one in the middle causes Terraform to shift/recreate the others.

---

### Q60. What is `for_each` and when is it better than `count`?
**Answer:** `for_each` creates instances from a **map or set**, keyed by a stable string instead of a numeric index:

```hcl
resource "azurerm_resource_group" "rg" {
  for_each = toset(["dev", "test", "prod"])
  name     = "rg-${each.key}"
  location = "East US"
}
```

**Why it's often better than `count`:** keys are stable, so adding/removing one item doesn't recreate the others. Use `count` for "N identical copies"; use `for_each` for "a distinct resource per named item."

---

### Q61. What is the `depends_on` meta-argument?
**Answer:** Terraform usually infers dependencies automatically from references between resources. `depends_on` is for **explicit, hidden dependencies** Terraform can't see — e.g., when resource B depends on a side effect of resource A but doesn't directly reference any of A's attributes:

```hcl
resource "azurerm_role_assignment" "ra" {
  # ...
  depends_on = [azurerm_key_vault.kv]
}
```

Use it sparingly; over-using it can slow plans and hide design issues.

---

### Q62. What does the `lifecycle` block do? Name its key arguments.
**Answer:** The `lifecycle` block customizes how Terraform creates/updates/destroys a resource:
- **`create_before_destroy`** — create the replacement *before* destroying the old one (avoids downtime).
- **`prevent_destroy`** — block accidental deletion (errors if a plan would destroy it); great for prod databases.
- **`ignore_changes`** — ignore changes to specified attributes (e.g., tags modified outside Terraform).
- **`replace_triggered_by`** — force replacement when a referenced resource/attribute changes.

```hcl
lifecycle {
  prevent_destroy = true
  ignore_changes  = [tags["last-modified"]]
}
```

---

### Q63. Give a real-world use of `ignore_changes`.
**Answer:** A common case: Azure (or another tool/policy) auto-applies tags or auto-scales instance counts. If Terraform "owns" that attribute, every plan would try to revert it, causing churn. By adding `ignore_changes = [tags]` (or `[sku.0.capacity]` for autoscaling), Terraform leaves that attribute alone after creation, ending the tug-of-war. Another classic: ignoring changes to a VM's admin password or an image reference managed elsewhere.

---

### Q64. What are conditional expressions and dynamic blocks?
**Answer:**
**Conditional expression** (ternary): `condition ? true_value : false_value`
```hcl
sku = var.environment == "prod" ? "Premium" : "Standard"
```

**Dynamic block** — generates nested configuration blocks programmatically, avoiding repetition:
```hcl
dynamic "security_rule" {
  for_each = var.rules
  content {
    name                   = security_rule.value.name
    priority               = security_rule.value.priority
    # ...
  }
}
```
Dynamic blocks are essential for things like NSG (Network Security Group) rules where the count varies by environment.

---

### Q65. What are `for` expressions and the splat operator?
**Answer:**
**`for` expression** transforms one collection into another:
```hcl
locals {
  upper_names = [for n in var.names : upper(n)]
  name_map    = { for n in var.names : n => length(n) }
}
```

**Splat operator (`[*]`)** extracts an attribute from all instances of a resource:
```hcl
output "all_ips" {
  value = azurerm_public_ip.pip[*].ip_address
}
```
Both are heavily used to keep configs DRY (Don't Repeat Yourself).

---

### Q66. What are `moved` blocks?
**Answer:** Introduced in Terraform **1.1**, `moved` blocks let you **refactor resource addresses declaratively** — renaming resources or moving them into modules **without** running `terraform state mv` manually and without recreating the resource:

```hcl
moved {
  from = azurerm_storage_account.old
  to   = azurerm_storage_account.new
}
```

On the next apply, Terraform updates the state mapping and shows **no changes** to the real resource. It's reviewable in Git and works for everyone on the team — superior to manual state surgery.

---

### Q67. What are `import` blocks (Terraform 1.5+)?
**Answer:** They allow **declarative import** of existing resources into state as part of a normal plan/apply, replacing one-off `terraform import` CLI calls:

```hcl
import {
  to = azurerm_resource_group.rg
  id = "/subscriptions/.../resourceGroups/rg-legacy"
}
```

Benefits: reviewable in pull requests, repeatable across environments, and can pair with `-generate-config-out` to scaffold the resource HCL automatically.

---

### Q68. What are common built-in functions you've used and why?
**Answer:** A few practical ones:
- **`lookup(map, key, default)`** — safely read a map value with a fallback.
- **`merge(map1, map2)`** — combine tag maps (e.g., global tags + resource-specific tags).
- **`coalesce(a, b, …)`** — return the first non-empty value.
- **`try(expr, fallback)`** — return a fallback if an expression errors (great for optional nested attributes).
- **`cidrsubnet()`** — calculate subnet ranges from a base CIDR (Classless Inter-Domain Routing).
- **`format()` / `join()` / `split()`** — string building for resource names.
- **`jsonencode()` / `templatefile()`** — generate JSON (e.g., for policies) or render templates (e.g., cloud-init).

---

## 8. Provisioners, Lifecycle & Drift

### Q69. What are provisioners and why are they a "last resort"?
**Answer:** **Provisioners** run scripts/commands on a local or remote machine as part of resource creation/destruction (`local-exec`, `remote-exec`, `file`). HashiCorp explicitly calls them a **last resort** because:
- They break Terraform's declarative model (Terraform can't track what a script did).
- They make plans unpredictable and harder to debug.
- Failures can leave resources "tainted"/half-built.

**Better alternatives:** cloud-init / custom_data for VM bootstrapping, Azure VM extensions, or dedicated config management (Ansible) run separately.

---

### Q70. What's the difference between `local-exec` and `remote-exec`?
**Answer:**
- **`local-exec`** runs a command on the **machine running Terraform** (e.g., calling an external API or writing a file).
- **`remote-exec`** runs commands on the **newly created remote resource** (e.g., SSH into a VM and run setup), requiring a connection block with credentials.

Both should be avoided when a native option exists.

---

### Q71. What is configuration drift and how does Terraform detect it?
**Answer:** **Drift** is when the real infrastructure differs from what's recorded in state/code — usually because someone changed something manually in the Azure Portal or another tool modified it. Terraform detects drift during `plan` (it refreshes state against reality and shows differences). To audit drift without changing anything:

```bash
terraform plan -refresh-only
```

You then decide whether to **revert** (apply your code) or **accept** the change (update code / use `ignore_changes`).

---

### Q72. How do you handle drift in a production environment?
**Answer:** A pragmatic approach:
1. **Detect** drift continuously (scheduled `plan -refresh-only` / `plan` in CI; tools like driftctl or Terraform Cloud drift detection).
2. **Investigate** whether the manual change was legitimate (emergency hotfix) or accidental.
3. **Reconcile:** either codify the change (update Terraform) or revert it (apply).
4. **Prevent recurrence:** restrict portal write access (RBAC), enforce changes through pipelines, and use Azure Policy to block out-of-band changes.

---

### Q73. What does it mean to "taint" a resource, and what replaced it?
**Answer:** Tainting marks a resource for **destruction and recreation** on the next apply (useful when a resource is in a bad state). `terraform taint` is **deprecated** (since Terraform 0.15.2). The modern equivalent is the `-replace` flag:

```bash
terraform apply -replace="azurerm_linux_virtual_machine.vm"
```

This is explicit and shows up in the plan, unlike the old stateful taint.

---

### Q74. What causes Terraform to force-replace a resource instead of updating it in place?
**Answer:** Some attributes are **immutable** in Azure — they can't be changed without recreating the resource. When you change such an attribute, the plan shows `-/+` ("destroy and then create replacement") and a note like *"# forces replacement."* Examples: changing a VM's `location`, certain storage `account_tier` changes, or a resource's `name`. To avoid downtime on forced replacement, use `lifecycle { create_before_destroy = true }`.

---

### Q75. How do you do a zero-downtime replacement of a resource?
**Answer:** Use `create_before_destroy` so the new instance is provisioned and healthy before the old one is removed:

```hcl
resource "azurerm_linux_virtual_machine" "vm" {
  # ...
  lifecycle {
    create_before_destroy = true
  }
}
```

Combine with a load balancer / health probes so traffic only shifts once the new resource is ready. For stateful resources (databases), zero-downtime usually needs an application-level strategy (replication, blue-green), not just Terraform.

---

## 9. Secrets & Security

### Q76. How do you avoid hard-coding secrets in Terraform code?
**Answer:** Never put secrets in `.tf` files (they'd land in Git). Options:
- **Azure Key Vault** — store secrets there and read them with a data source at apply time.
- **Environment variables** (`TF_VAR_*`, `ARM_CLIENT_SECRET`).
- **Pipeline secret stores** — Azure DevOps variable groups, GitHub Actions secrets.
- **OIDC** to avoid client secrets entirely.

Also add `*.tfvars` and `*.tfstate` to `.gitignore` (for any local secrets).

---

### Q77. How do you read a secret from Azure Key Vault in Terraform?
**Answer:** Use the Key Vault data sources:

```hcl
data "azurerm_key_vault" "kv" {
  name                = "kv-shared-prod"
  resource_group_name = "rg-security"
}

data "azurerm_key_vault_secret" "db_password" {
  name         = "db-admin-password"
  key_vault_id = data.azurerm_key_vault.kv.id
}

# use: data.azurerm_key_vault_secret.db_password.value
```

> **Caveat:** the retrieved secret is still written to the **state file in plain text**, so secure the backend (encryption, restricted access) regardless.

---

### Q78. What is a Managed Identity and how does it help Terraform deployments?
**Answer:** A **Managed Identity** is an Azure-managed Service Principal automatically tied to an Azure resource (VM, App Service, DevOps agent). The platform handles credential rotation — **no secrets to store**. Two types:
- **System-assigned** — tied to one resource's lifecycle.
- **User-assigned** — standalone, reusable across resources.

For Terraform, running on an agent with a managed identity means it authenticates to Azure with **zero stored credentials** — a strong security posture.

---

### Q79. The state file contains secrets in plaintext. How do you mitigate this risk?
**Answer:** Layered mitigations:
- **Remote backend with encryption at rest** (Azure Storage default + optional CMK in Key Vault).
- **Restrict access** to the state container via RBAC, private endpoints, and storage firewall.
- **Enable versioning/soft delete** for recovery, but keep access tight.
- **Avoid putting more secrets in state than necessary** (generate passwords in Key Vault, reference by ID where possible).
- **Audit access** via Azure Monitor / storage logs.
- Mark variables/outputs `sensitive` to keep them out of logs.

---

### Q80. How do you generate and manage random secrets (e.g., passwords) in Terraform?
**Answer:** Use the **`random`** provider's `random_password` resource, then often store it in Key Vault:

```hcl
resource "random_password" "db" {
  length  = 24
  special = true
}

resource "azurerm_key_vault_secret" "db" {
  name         = "db-password"
  value        = random_password.db.result
  key_vault_id = azurerm_key_vault.kv.id
}
```

The generated value is stored in state, so secure the backend. `random` resources are stable (they don't regenerate on every apply) unless their inputs (`keepers`) change.

---

### Q81. How do you enforce security/compliance policies on what Terraform can deploy?
**Answer:** Several layers:
- **Azure Policy** — enforced by Azure itself (e.g., "only allowed regions," "storage must use HTTPS"). It blocks non-compliant deployments regardless of how they're made.
- **Sentinel (Terraform Cloud/Enterprise)** or **OPA (Open Policy Agent)** — **policy as code** that evaluates the *plan* before apply (e.g., "no public IPs in prod," "VMs must be tagged").
- **`tfsec` / `checkov` / `terrascan`** — static analysis scanners run in CI to catch insecure configs early.

The pattern: shift-left scanning in CI **plus** guardrails enforced by Azure Policy at deploy time.

---

### Q82. What's the difference between Azure Policy and Sentinel/OPA in a Terraform context?
**Answer:**
- **Azure Policy** runs **inside Azure** and enforces rules at the platform level — it applies to *all* changes (Terraform, portal, CLI). It can audit, deny, or auto-remediate.
- **Sentinel / OPA** run **in the Terraform pipeline**, evaluating the plan **before** anything reaches Azure. They catch issues earlier and can encode org-specific rules with rich logic.

Best practice is **both**: pre-deployment policy-as-code (fast feedback) and Azure Policy as the last line of defense.

---

## 10. CI/CD & Team Collaboration

### Q83. How do you run Terraform in a CI/CD pipeline?
**Answer:** A typical pipeline stage flow:
1. **`terraform fmt -check`** and **`validate`** — linting/syntax.
2. **Security scan** (tfsec/checkov).
3. **`terraform plan -out=tfplan`** — produce a plan artifact.
4. **Manual approval** (especially for prod) — a human reviews the plan.
5. **`terraform apply tfplan`** — apply the exact reviewed plan.

Authentication uses OIDC or a Service Principal; state lives in a remote backend with locking. Tools: Azure DevOps Pipelines, GitHub Actions, or Terraform Cloud/Enterprise.

---

### Q84. Why save the plan as an artifact and apply that exact file?
**Answer:** Because between `plan` and `apply`, the real infrastructure or code could change. Applying a **saved plan file** guarantees Terraform applies **exactly what was reviewed/approved** — nothing more, nothing less. If reality drifted since the plan, Terraform errors out instead of silently applying something different. This makes approvals meaningful and changes auditable.

---

### Q85. How do you prevent two pipelines from corrupting state simultaneously?
**Answer:** **State locking.** With the azurerm backend, the blob lease automatically locks state during apply, so concurrent runs queue/fail rather than clobber each other. Additionally:
- Use **separate state files per environment** so dev and prod pipelines never touch the same state.
- Configure pipeline **concurrency controls** (e.g., one prod deploy at a time).
- Avoid `-lock=false` except for genuine emergencies.

---

### Q86. How do you manage multiple environments (dev/staging/prod)? Compare the approaches.
**Answer:** Two main strategies:

**1. Separate directories per environment** (recommended for most prod setups):
- Each env has its own folder, backend key, and `.tfvars`.
- Strong isolation, explicit, easy to diverge config when needed.

**2. Terraform workspaces:**
- One codebase, multiple named workspaces (`dev`, `prod`), each with isolated state.
- Less code duplication, but easy to accidentally apply to the wrong workspace, and it's harder to have structurally different environments.

Interviewers like hearing: *"Workspaces are great for small/identical environments, but for prod isolation I prefer separate directories/state and pipelines."*

---

### Q87. What are Terraform workspaces and what's a common misconception?
**Answer:** **Workspaces** let one configuration maintain **multiple separate state files** (e.g., `terraform workspace new dev`). Each workspace has isolated state but shares the same code. You reference the current one via `terraform.workspace`.

**Common misconception:** that workspaces are a robust way to separate prod from dev. They share the same backend and code, so it's easy to apply to the wrong workspace, and they don't isolate credentials/permissions. For strong prod isolation, prefer separate state/backends and pipelines.

---

### Q88. How do you handle code review for Terraform changes?
**Answer:** Treat infrastructure like application code:
- All changes via **pull requests** to Git.
- CI runs **fmt/validate/scan** and posts the **`terraform plan` output** to the PR so reviewers see exactly what will change.
- Require **approvals** before merge, and a separate **manual approval** before prod apply.
- Protect main branches; never apply directly from a laptop to prod.

Seeing the plan in the PR is the key practice — reviewers approve actual diffs, not just code.

---

### Q89. What is Terraform Cloud / Enterprise and when would you use it?
**Answer:** **Terraform Cloud (TFC)** / **Terraform Enterprise (TFE)** is HashiCorp's managed platform that adds: remote state with built-in locking, remote plan/apply runs, a private module registry, **Sentinel** policy-as-code, RBAC, run approvals, and VCS integration. You'd use it instead of a homegrown pipeline when you want governance, audit, and collaboration features out of the box — especially in larger orgs. Smaller teams often achieve similar results with Azure DevOps/GitHub Actions + an azurerm backend.

---

## 11. Advanced Topics & Troubleshooting

### Q90. What is the difference between `terraform plan` exit codes and how do you use them in automation?
**Answer:** With `terraform plan -detailed-exitcode`, Terraform returns:
- **0** — no changes (success, nothing to do).
- **1** — error.
- **2** — succeeded, **changes present**.

Automation uses this to decide whether to proceed (e.g., only run apply / send a notification when exit code is 2). It's a clean way to do **drift detection** jobs and conditional pipeline steps.

---

### Q91. How does Terraform build and use its dependency graph?
**Answer:** Terraform parses your configuration and builds a **DAG (Directed Acyclic Graph)** of resources based on references between them. It uses the graph to:
- Determine the correct **order** of operations.
- **Parallelize** independent resources (default up to 10 concurrent operations, tunable with `-parallelism`).
- Compute the right order for destroys (reverse dependency order).

You can visualize it with `terraform graph`. Explicit `depends_on` adds edges the graph can't infer automatically.

---

### Q92. A `terraform apply` failed halfway. What's the state, and how do you recover?
**Answer:** Terraform applies resources incrementally and **updates state as each resource succeeds**, so a mid-apply failure leaves you with a **partial deployment**: some resources created (and in state), others not. Recovery:
1. Read the error — fix the root cause (quota, permission, invalid value, dependency).
2. Re-run `terraform plan` to see what's left to do — Terraform will only create/fix the remaining/failed resources.
3. `terraform apply` again to converge.
4. If a resource was left half-created/corrupt, use `-replace` to recreate it.

Because state is updated as it goes, Terraform is generally **idempotent** and safe to re-run.

---

### Q93. How do you debug a confusing Terraform error?
**Answer:** Tools and techniques:
- **`TF_LOG`** environment variable (`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`) to get detailed logs; `TF_LOG_PATH` to write them to a file.
- **`terraform console`** to test expressions/interpolations.
- **`terraform plan`** carefully — read the "forces replacement" notes.
- **`terraform state show`** to inspect actual stored attributes.
- **`terraform validate`** for syntax issues.
- Reproduce with a **minimal config** to isolate the problem.

---

### Q94. What is the `-parallelism` flag?
**Answer:** It controls how many resource operations Terraform performs concurrently (default **10**). `terraform apply -parallelism=5` lowers it. You reduce it to avoid hitting Azure API **rate limits/throttling** (HTTP 429 errors) on large deployments, or raise it (cautiously) to speed up big applies. It doesn't change correctness — only concurrency.

---

### Q95. How do you handle Azure API throttling / rate limits with large Terraform deployments?
**Answer:** Strategies:
- **Reduce `-parallelism`** to ease API pressure.
- **Split large configurations** into smaller states so each apply touches fewer resources.
- **Retry** — the azurerm provider has built-in retries for transient 429s; pipelines can also retry.
- **Stagger deployments** across resource groups/subscriptions.
- Watch for **provider/version-specific** known throttling issues and upgrade if fixed.

---

### Q96. What is the `null_resource` and the `terraform_data` resource used for?
**Answer:** Both are "do-nothing" resources used as **glue**:
- **`null_resource`** (from the `null` provider) historically paired with `triggers` + provisioners to run actions when certain values change.
- **`terraform_data`** (built-in since Terraform 1.4) is the modern replacement — no extra provider needed. It can store values and trigger replacement via `triggers_replace`, useful for orchestrating provisioners or forcing dependent actions.

Use them sparingly; prefer native resources where possible.

---

### Q97. What is `terraform graph` and when is it helpful?
**Answer:** It outputs the dependency graph in **DOT** format (visualizable with Graphviz). It helps you:
- Understand **why** Terraform orders operations a certain way.
- Spot unexpected dependencies or cycles.
- Debug complex modules.

```bash
terraform graph | dot -Tpng > graph.png
```

---

### Q98. How do you upgrade the azurerm provider across a major version (e.g., 3.x → 4.x) safely?
**Answer:** A careful process:
1. **Read the upgrade guide / changelog** for breaking changes (e.g., v4 made `subscription_id` required, removed deprecated resources/attributes).
2. **Bump the version constraint** and run `terraform init -upgrade`.
3. **Run `terraform plan`** in a **non-prod** environment first; investigate any unexpected replacements.
4. Fix deprecated arguments / renamed attributes.
5. Test thoroughly, then promote through environments (dev → staging → prod).
6. Commit the updated `.terraform.lock.hcl`.

Never upgrade providers blindly straight in prod.

---

### Q99. What's the difference between `terraform state pull/push` and when are they dangerous?
**Answer:**
- **`terraform state pull`** — safely downloads and outputs current remote state (read-only). Useful for inspection/backup.
- **`terraform state push`** — **overwrites** remote state with a local file. This is **dangerous** because a wrong push can corrupt or roll back state, orphaning resources. Use only as a last-resort recovery step, after backing up the current state, and ideally with a lock.

---

### Q100. How would you manage a resource that Terraform doesn't support yet?
**Answer:** Options in order of preference:
1. **`azapi` provider** — directly call the Azure Resource Manager REST API to manage the resource declaratively, including preview features.
2. **`azurerm_resource_group_template_deployment`** — embed an ARM/Bicep template inside Terraform.
3. **`null_resource`/`terraform_data` + `local-exec`** calling Azure CLI (least desirable — not truly declarative).

Then migrate to the native `azurerm` resource once it's released, using `import`/state moves.

---

## 12. Real-World Scenario Questions

### Q101. Scenario: A teammate manually changed a VM's size in the Azure Portal. Your next `plan` wants to revert it. What do you do?
**Answer:** First, decide if the manual change should stay or go:
- **If it was a legitimate change**, update the Terraform code to match (so code = reality) and apply. Going forward, restrict portal write access so changes go through code.
- **If it was accidental/unauthorized**, run `terraform apply` to revert it back to the declared size.
- If this attribute is *intentionally* managed outside Terraform, add `lifecycle { ignore_changes = [size] }` so Terraform stops fighting it.

The key point is **reconciling drift deliberately**, and preventing recurrence with RBAC/Azure Policy.

---

### Q102. Scenario: You need to onboard a large set of existing, manually-created Azure resources into Terraform. How do you approach it?
**Answer:** A structured migration:
1. **Inventory** the existing resources (Azure CLI / Resource Graph).
2. **Write resource blocks** (or use `import` blocks with `-generate-config-out` to scaffold HCL).
3. **Import** each resource into state (CLI import or import blocks).
4. **Run `terraform plan`** repeatedly and refine code until the plan shows **no changes** (meaning code matches reality).
5. Start small (one resource group), validate, then scale up.
6. Optionally use tools like **Azure Export for Terraform (`aztfexport`)** to bulk-generate config + state from existing resources.

The success criterion: a clean `plan` with zero changes after import.

---

### Q103. Scenario: Two engineers ran `apply` at the same time and state looks inconsistent. What happened and how do you prevent it?
**Answer:** Likely **state locking was disabled or absent** (e.g., local state, or someone used `-lock=false`), letting two applies write concurrently and corrupt/race the state.

Fix/prevent:
- Move to a **remote backend** (azurerm) which **auto-locks** via blob lease.
- Never use `-lock=false` outside emergencies.
- If a stale lock remains from a crash, use `terraform force-unlock <ID>` carefully.
- Restore from a state backup/version if corruption occurred.
- Enforce changes through a **pipeline with concurrency controls**.

---

### Q104. Scenario: You must deploy nearly-identical infrastructure to 10 regions. How do you keep the code DRY?
**Answer:** Combine `for_each` with a map of region configs and/or a reusable module:

```hcl
module "regional" {
  for_each = var.regions   # map of region => settings
  source   = "./modules/stack"
  location = each.key
  # per-region overrides from each.value
}
```

This deploys the same module per region, keyed by region name (stable keys via `for_each`), with per-region overrides supplied through the map. Avoids copy-paste and makes adding/removing a region a one-line change.

---

### Q105. Scenario: A `prod` database is in your Terraform state. How do you protect it from accidental deletion?
**Answer:** Layered protection:
- **`lifecycle { prevent_destroy = true }`** on the resource — any plan that would destroy it errors out.
- **Don't use `terraform destroy`** on prod; use targeted, reviewed changes.
- **Azure resource lock** (`CanNotDelete`) at the Azure level as a backstop.
- **Backups / point-in-time restore** enabled on the database.
- Separate prod state and require manual approval for prod applies.

---

### Q106. Scenario: Your `terraform plan` unexpectedly wants to destroy and recreate a critical resource. How do you investigate and avoid downtime?
**Answer:**
1. **Read the plan** — find the attribute marked *"# forces replacement"*; an immutable property changed.
2. **Decide** if the change is necessary. If not, revert the code change.
3. If replacement *is* required, **avoid downtime** with `lifecycle { create_before_destroy = true }` and load-balancer health checks, or use a blue-green approach for stateful systems.
4. For false positives (e.g., provider re-ordering a list), check provider version notes; sometimes `ignore_changes` or a provider upgrade resolves it.

---

### Q107. Scenario: How would you implement a blue-green deployment with Terraform on Azure?
**Answer:** Blue-green keeps two environments (blue = current, green = new) and switches traffic:
- Parameterize the "active" environment via a variable (`active_slot = "blue" | "green"`).
- Deploy the new version into the **idle** slot (e.g., a second VMSS / App Service slot / set of resources).
- **Validate** the green environment.
- **Switch traffic** by updating the load balancer / Traffic Manager / App Service slot swap (often the final Terraform change or an Azure-native swap).
- Keep blue around briefly for instant rollback, then decommission.

Azure App Service **deployment slots** make this especially clean; Terraform manages the slots and the swap configuration.

---

### Q108. Scenario: The state file's storage account was accidentally deleted. What now?
**Answer:**
1. **Try to recover the storage account/blob** — Azure soft delete (if enabled) can restore deleted blobs/containers/accounts within the retention window. This is the fastest path.
2. If a **state backup** exists (blob versioning, a CI-archived copy, or `.tfstate.backup`), restore it to a new backend.
3. If no backup exists, **rebuild state by importing** every resource — slow but recoverable since the real Azure resources still exist.

The lesson the interviewer wants: this is why you **enable versioning + soft delete and back up state**.

---

### Q109. Scenario: A junior engineer added a `count` to a resource list, and now removing one item recreates several others. Why, and how do you fix it?
**Answer:** **Why:** `count` addresses instances by **numeric index**. Removing an item in the middle shifts every later item's index, so Terraform thinks they all changed and recreates them.

**Fix:** Switch from `count` to **`for_each`** keyed by a **stable identifier** (a name/string), so each instance is tied to its key and unaffected by others being added/removed:

```hcl
resource "azurerm_subnet" "s" {
  for_each = var.subnets   # map keyed by subnet name
  # ...
}
```

You may need `terraform state mv` / `moved` blocks to migrate existing indexed instances to the new keyed addresses without recreation.

---

### Q110. Scenario: You need different configurations for dev vs prod (e.g., prod has geo-redundancy, dev doesn't). How do you structure this cleanly?
**Answer:** Keep one set of modules and drive differences through variables and conditionals:
- **Per-environment `.tfvars`** supply sizing, SKUs, replication settings, and feature flags.
- Use **conditionals** for structural differences: `account_replication_type = var.environment == "prod" ? "GRS" : "LRS"` (GRS = Geo-Redundant Storage, LRS = Locally-Redundant Storage).
- For larger structural divergence, use **`count`/`for_each` feature flags** (e.g., `count = var.enable_dr ? 1 : 0` for DR-only resources).
- Separate **state/backends and pipelines** per environment for isolation.

This keeps code DRY while letting environments differ where it matters.

---

### Q111. Scenario: Your team wants to share a standard, compliant "landing zone" pattern across many projects. How do you deliver it?
**Answer:** Build **versioned, reusable modules** published to a **private module registry** (Terraform Cloud/Enterprise or an internal registry/Git tags):
- Encapsulate the compliant patterns (naming, tagging, networking, RBAC, diagnostics) inside the modules with **sensible secure defaults**.
- **Version with SemVer** so teams upgrade deliberately.
- Provide **examples and documentation**.
- Enforce guardrails with **Sentinel/OPA** and **Azure Policy**.
- Optionally align with **Microsoft's Cloud Adoption Framework (CAF)** and the official `Azure/caf-*` modules.

Teams consume the modules rather than reinventing infrastructure, ensuring consistency and compliance.

---

### Q112. Scenario: A `terraform destroy` is hanging or failing on a resource group. What are likely causes and fixes?
**Answer:** Common causes:
- **Resources created outside Terraform** inside the RG, or **dependencies** not fully captured, so deletion order is wrong.
- **Resource locks** (`CanNotDelete`) at the Azure level blocking deletion.
- **`prevent_deletion_if_contains_resources`** provider feature is true.
- **Soft-delete-protected** resources (e.g., Key Vault) or long-running deletes (e.g., some networking resources).

Fixes: remove Azure locks, ensure dependencies are managed/declared, adjust the provider `features` block, handle soft-deleted resources (purge), and check for orphaned resources blocking the RG. Use `TF_LOG=DEBUG` to see which API call is stuck.

---

## 13. Best Practices (Rapid-Fire)

### Q113. What are your top Terraform best practices?
**Answer:**
- **Remote backend** with locking, encryption, versioning, and least-privilege access.
- **Pin** provider and module versions; commit `.terraform.lock.hcl`.
- **Modules** for reuse; **separate state per environment** for isolation.
- **No secrets in code/state more than necessary**; use Key Vault + OIDC.
- **Run `fmt`, `validate`, security scans, and `plan`** in CI; require PR review of plans.
- **Apply a saved plan** behind approvals; never apply prod from a laptop.
- **Use `for_each` over `count`** for stable, named resources.
- **Consistent naming and tagging** conventions.

---

### Q114. Should you commit the `.terraform` directory and `*.tfstate` to Git?
**Answer:** **No.** Add `.terraform/`, `*.tfstate`, `*.tfstate.backup`, and local `*.tfvars` containing secrets to `.gitignore`. The `.terraform` directory is just downloaded plugins/modules (re-creatable via `init`), and state files contain sensitive data and belong in a secure remote backend. **Do** commit `.terraform.lock.hcl`.

---

### Q115. Why use consistent naming conventions and tagging, and how do you enforce them?
**Answer:** Consistent **naming** (e.g., `rg-app-prod-eastus`) and **tagging** (owner, cost-center, environment) improve discoverability, cost allocation, automation, and governance. Enforce via:
- A **naming module/local** that builds names from inputs.
- A **common tags map** merged onto every resource (`tags = merge(local.common_tags, var.extra_tags)`).
- **Azure Policy** to require specific tags.
- **Variable validation** to reject invalid names.

---

### Q116. How do you keep Terraform code DRY (Don't Repeat Yourself)?
**Answer:** Use **modules** for repeated patterns, **`for_each`/`count`** for repeated resources, **`locals`** for computed/shared values, **`merge()`** for shared tags, and **variables + tfvars** for environment differences. Some teams add **Terragrunt** (a wrapper) to reduce backend/provider boilerplate across many environments.

---

### Q117. What is Terragrunt and when is it useful?
**Answer:** **Terragrunt** is a thin wrapper around Terraform that reduces repetition and adds orchestration for **many environments/modules**. It helps **keep backend and provider config DRY**, manage dependencies between modules, and run commands across multiple stacks at once. It's useful in large, multi-environment, multi-account setups; for smaller projects, native Terraform features are usually enough.

---

### Q118. How do you test Terraform code?
**Answer:** Layered testing:
- **`terraform validate` + `fmt`** — syntax/format.
- **Static analysis** — `tfsec`, `checkov`, `tflint`.
- **`terraform plan`** review — catch unintended changes.
- **Native `terraform test`** (built-in test framework, `.tftest.hcl`) — assert plan/apply outcomes.
- **Terratest** (Go-based) — spin up real infrastructure, assert it works, tear it down (integration testing).
- **Policy-as-code** — Sentinel/OPA gates.

---

### Q119. How do you handle large numbers of similar resources without performance/maintenance pain?
**Answer:**
- Drive them from **data structures (maps/lists)** with `for_each` so adding one is a data change, not new code.
- **Split state** to keep plans fast and blast radius small.
- Tune **`-parallelism`** to avoid API throttling.
- Use **modules** to encapsulate the per-item pattern.
- Avoid massive monolithic states that take many minutes to plan.

---

### Q120. What would you put in a Terraform `.gitignore` and a project README for team onboarding?
**Answer:**
**`.gitignore`:** `.terraform/`, `*.tfstate`, `*.tfstate.*`, `crash.log`, `*.tfvars` (if they hold secrets), `override.tf`, `.terraformrc`, `terraform.rc`.

**README should include:** prerequisites (Terraform version, providers), how to authenticate to Azure (OIDC/SP), backend/state details, how to run plan/apply per environment, variable descriptions, module usage examples, and contribution/PR workflow. Good onboarding docs let a new engineer deploy safely on day one.

---

## ✅ Final Tips for the Interview
- **Always explain the *why*, not just the command** — interviewers test understanding, not memorization.
- **Lean on real scenarios:** state corruption, drift, zero-downtime replacement, multi-environment, and secret management come up constantly.
- **Know the recent changes:** azurerm v4 (`subscription_id` required), `import`/`moved` blocks, `taint` → `-replace`, Terraform's BUSL license + OpenTofu.
- **Security & state protection** are the themes most likely to separate a senior answer from a junior one.

> **Good luck — review the scenario questions (Section 12) the night before your interview; they map directly to what real teams hit in production.**
