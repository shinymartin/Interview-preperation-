# Interview Prep — Terraform Azure Module Library + CI/CD

Likely interview questions based on this project, grouped by topic, each with a
short answer **grounded in what this repo actually does**. Use the answers as
talking points — say them in your own words, and always tie them back to a
concrete choice you made here.

> Tip: interviewers care less about textbook definitions and more about *why you
> built it this way*. Whenever you can, answer with "In my project I did X
> because Y."

## Contents

1. [Project overview & "walk me through it"](#1-project-overview--walk-me-through-it)
2. [Terraform fundamentals](#2-terraform-fundamentals)
3. [Modules](#3-modules)
4. [Remote state](#4-remote-state)
5. [Azure resources](#5-azure-resources)
6. [CI/CD & GitHub Actions](#6-cicd--github-actions)
7. [The pipeline checks](#7-the-pipeline-checks)
8. [Security](#8-security)
9. [OIDC & authentication](#9-oidc--authentication)
10. [Versioning & releases](#10-versioning--releases)
11. [Branch protection & workflow](#11-branch-protection--workflow)
12. [Cost management](#12-cost-management)
13. [Troubleshooting & debugging](#13-troubleshooting--debugging)
14. [Design decisions & trade-offs](#14-design-decisions--trade-offs)
15. [Scenario & behavioural](#15-scenario--behavioural)
16. [Rapid-fire definitions](#16-rapid-fire-definitions)

---

## 1. Project overview & "walk me through it"

**Q: Give me a 60-second overview of this project.**
A reusable, versioned Terraform module library for Azure, plus a GitHub Actions
CI/CD pipeline that automatically checks every pull request for formatting,
linting, security, the plan, and the cost before it can merge. It demonstrates
how infrastructure is *operated* by a team — modular, tested, reviewed,
versioned — not just deployed once.

**Q: What problem does it solve?**
Teams shouldn't copy-paste Terraform or run `apply` blindly. This gives them
reusable building blocks (modules), pins them to versions so consumers don't
break, and gates every change behind automated quality, security, and cost
checks with human review.

**Q: Who is the "user" of this repo?**
Other engineers/projects that consume the modules by referencing a specific
version, and the team maintaining the library via pull requests.

**Q: Walk me through what happens from a code change to a release.**
Branch → edit a module → test locally (fmt/validate/lint/checkov/infracost) →
open a PR → the pipeline runs the same checks and posts the plan + cost as PR
comments → reviewer approves → merge to `main` → tag a semantic version → other
projects can consume the new version.

---

## 2. Terraform fundamentals

**Q: What is Terraform and why use it?**
An infrastructure-as-code tool that declares desired infrastructure in HCL and
reconciles reality to match it. Benefits: repeatable, version-controlled,
reviewable, and cloud-agnostic in syntax.

**Q: Explain `init`, `plan`, and `apply`.**
`init` downloads providers and configures the backend; `plan` computes the diff
between your config and current state and shows what *would* change; `apply`
executes that diff.

**Q: What is Terraform state and why does it matter?**
A file mapping your configuration to real resources. It lets Terraform know what
already exists so `plan` can compute an accurate diff. Without it, Terraform
couldn't track or update resources it created.

**Q: What is a provider?**
A plugin that knows how to talk to a specific API — here, `azurerm` for Azure. It
translates HCL resources into API calls.

**Q: How do you pin provider versions, and why?**
In `versions.tf` with `required_providers` (e.g. `>= 3.0.0, < 5.0.0`). Pinning
prevents a new provider release from silently changing behaviour or breaking the
build.

**Q: Difference between a variable and an output?**
A variable is an input to a module/config; an output is a value the
module/config exposes to its caller (like a resource ID).

**Q: What does `terraform fmt` do?**
Rewrites files to the canonical style (spacing/indentation). In CI we run
`terraform fmt -check` so badly formatted code fails the build.

**Q: What is the `.terraform.lock.hcl` file? Do you commit it?**
The dependency lock file recording exact provider versions and hashes. Yes — you
commit it so everyone (and CI) resolves identical versions. Gotcha: it records
hashes per platform, so you may need `terraform providers lock -platform=windows_amd64 -platform=linux_amd64`
so the Linux CI runner doesn't fail on a Windows-only lock.

---

## 3. Modules

**Q: What is a Terraform module?**
A reusable folder of Terraform that creates one logical thing, with defined
inputs (variables) and outputs. Other configs *call* it instead of duplicating
code.

**Q: How is each of your modules structured?**
`versions.tf` (required versions), `variables.tf` (inputs), `main.tf`
(resources), `outputs.tf` (outputs), and `README.md` (usage). Consistent layout
across `resource-group`, `storage-account`, `virtual-network`, and `linux-vm`.

**Q: What are the different module `source` types?**
Local path (`../../modules/x`), Git (`git::https://...//subdir?ref=tag`),
Terraform Registry, and others. Your example uses local paths; external
consumers use the Git source pinned to a tag.

**Q: How does a consumer use a specific version of your module?**
`source = "git::https://github.com/<you>/terraform-azure-modules.git//modules/resource-group?ref=v1.1.0"`.
The `//subdir` selects the module folder; `?ref=v1.1.0` pins the version.

**Q: Why reference a tag instead of `main`?**
So an upstream change to `main` can't unexpectedly break consumers. They upgrade
deliberately by bumping the `ref`.

**Q: How did you pass outputs from one module into another?**
In the example, the resource-group module's `name` output feeds the storage and
network modules' `resource_group_name` input, and the VNet's `subnet_ids` output
feeds the VM's `subnet_id`. That's module composition.

**Q: How would you make a module input safer / self-validating?**
Add a `validation` block to the variable (e.g. `account_tier` must be `Standard`
or `Premium`), so a bad value fails at `validate`/`plan` with a clear message
instead of erroring deep in the provider.

**Q: What's `for_each` and where did you use it?**
It creates one resource per item in a map/set. The VNet module uses
`for_each = var.subnets` to create a subnet per entry, keyed by name.

---

## 4. Remote state

**Q: Why store state remotely instead of locally?**
A laptop file can be lost, and CI can't read it. Remote state in Azure Storage is
shared, durable, and accessible to both you and the pipeline.

**Q: How is your remote backend configured?**
The `azurerm` backend in `backend.tf` points at a dedicated resource group,
storage account, container, and a `key` (the state file name).

**Q: What is state locking and why does it matter?**
It prevents two `apply`s running at once and corrupting state. The azurerm
backend locks via a blob lease automatically.

**Q: Can secrets end up in state? How do you handle that?**
Yes — state can contain sensitive values, which is why it must never be committed
(`.gitignore` excludes `*.tfstate`) and is stored in access-controlled Azure
Storage.

**Q: Why a separate resource group/storage account just for state?**
To isolate it from the resources Terraform manages, so a destroy of your app
infrastructure can never wipe the state that tracks it.

---

## 5. Azure resources

**Q: What Azure resources does the library create?**
Resource groups, storage accounts, virtual networks + subnets, and Linux VMs
(with a network interface).

**Q: What is a resource group?**
A logical container that groups related Azure resources for lifecycle and access
management.

**Q: Naming constraints you ran into?**
Storage account names must be globally unique, 3–24 chars, lowercase letters and
numbers only — a common source of "name already taken" errors.

**Q: How does the VM authenticate? Why no password?**
SSH public key only (`disable_password_authentication = true`). Password login is
a common attack vector; key-based auth is the secure default and also satisfies
Checkov.

**Q: Does the VM have a public IP?**
No — by design it's private-only (no public IP on the NIC), which is the secure
default.

**Q: What region and sizing did you use, and why?**
Cheap/free-tier-friendly choices (e.g. `Standard_B1s` VM, `Standard_LRS` disk,
`LRS` storage) to keep the demo low-cost.

---

## 6. CI/CD & GitHub Actions

**Q: What is CI/CD and how does it apply to infrastructure?**
Continuous Integration/Delivery: automatically validating and (optionally)
shipping changes. For IaC it means every change is linted, security-scanned, and
planned automatically before merge.

**Q: Where does your pipeline live and when does it run?**
In `.github/workflows/ci.yml`. It triggers on pull requests that touch
`modules/`, `examples/`, the workflows, or `*.hcl` files.

**Q: What are the jobs in your pipeline?**
Validate & Lint (fmt/validate/tflint), Security Scan (Checkov), Terraform Plan
(posts a PR comment), and Infracost (posts a cost diff comment).

**Q: Why trigger on pull_request rather than push?**
Because the goal is to gate changes *before* they reach `main`. The checks inform
the review and block the merge if they fail.

**Q: Where do GitHub Actions run, and what does that imply?**
On GitHub-hosted runners (here, `ubuntu-latest`) — not your laptop. So the
workflow only does anything once committed and pushed, and the runner is Linux
(relevant to the lock-file gotcha).

**Q: How does the pipeline post comments on the PR?**
The plan and Infracost steps use the `GITHUB_TOKEN`/actions to write a comment
with the plan output and the monthly cost difference, so reviewers see them
inline.

**Q: How would you keep secrets out of the workflow?**
Use GitHub Actions secrets (`AZURE_CLIENT_ID`, etc.) and OIDC for Azure auth — no
long-lived credentials in the YAML or repo.

---

## 7. The pipeline checks

**Q: Walk me through each check and what it catches.**
- `terraform fmt -check` — formatting consistency.
- `terraform validate` — syntax and internal consistency.
- `tflint` — provider best practices and likely errors.
- `Checkov` — security/compliance misconfigurations.
- `terraform plan` — what will actually change (as a PR comment).
- `Infracost` — the monthly cost impact (as a PR comment).

**Q: What's the difference between `validate` and `tflint`?**
`validate` checks that the config is syntactically valid and internally
consistent; `tflint` checks conventions/best practices and provider-specific
issues. They overlap a little but catch different classes of problem.

**Q: tflint didn't catch an invalid value like `Standard0` — why?**
tflint lints structure and conventions, not provider value-enums. Validating that
a value is one the Azure API accepts is the provider's job (caught at
`plan`/`validate`), or you enforce it yourself with a variable `validation` block.

**Q: How do you prove a linter is actually wired up?**
Introduce something it *does* check — e.g. an unused variable triggers
`terraform_unused_declarations` in tflint. If that warning appears, it's working.

---

## 8. Security

**Q: What is Checkov?**
A static analysis (SAST) tool for IaC that scans Terraform against security and
compliance policies and flags misconfigurations.

**Q: What security hardening did you put in the storage module?**
Enforced TLS 1.2, HTTPS-only, disabled public blob access, disabled public
network access, disabled shared-key auth (forcing Azure AD), added a SAS
expiration policy, and enabled soft-delete.

**Q: How do you handle a Checkov finding you don't want to fix?**
Either fix it (preferred) or add a `#checkov:skip=<ID>:<reason>` comment. The key
is documenting *why* — a justified skip silences only that finding while still
catching new ones. I skipped things needing external infra (CMK key vault,
private endpoints) or not relevant to a demo (queue logging, GRS).

**Q: Why is a documented skip better than disabling the scanner?**
It's surgical and auditable — it suppresses one known, justified finding rather
than blinding the whole gate, so future problems still get caught.

**Q: How would you make Checkov block vs. just warn?**
The `soft_fail`/`soft-fail` setting: `false` fails the build on findings (a hard
gate), `true` only reports. I used a hard gate so insecure code can't merge.

**Q: Give an example of catching a real vulnerability.**
Setting `min_tls_version = "TLS1_0"` and opening a PR — Checkov fails the build
and names the weak-TLS rule, so it never merges.

---

## 9. OIDC & authentication

**Q: How does the pipeline authenticate to Azure?**
Via OIDC (OpenID Connect) federated credentials — no stored passwords or client
secrets.

**Q: Explain how OIDC works here, step by step.**
GitHub's token service issues a short-lived signed JWT for the workflow run. Azure
AD has a *federated credential* that trusts that issuer, audience, and a specific
*subject* (e.g. this repo's PRs or `main`). `azure/login` exchanges the JWT for a
short-lived Azure access token. Nothing long-lived is stored.

**Q: Why is OIDC better than a service principal secret?**
No long-lived secret to leak, rotate, or accidentally commit. Tokens are
short-lived and scoped to specific repo/branch/environment subjects.

**Q: What is the "subject" and why did you register several?**
The subject identifies *which* workflow context is allowed. GitHub presents
different subjects for PRs (`...:pull_request`), the main branch
(`...:ref:refs/heads/main`), and environments (`...:environment:production`), so
each needs its own federated credential.

**Q: You hit `AADSTS700213: No matching federated identity record`. What causes it?**
The subject GitHub presented didn't match any registered federated credential —
e.g. adding `environment: production` changes the subject to an environment
subject, which needs its own credential.

**Q: What Azure permissions does the pipeline's identity have, and why two roles?**
`Contributor` on the subscription to manage resources, plus `Storage Blob Data
Contributor` because the storage module disables shared keys — so Terraform must
reach the blob (state/data) over Azure AD, which needs a data-plane role.

**Q: Are the `AZURE_CLIENT_ID`/`TENANT_ID`/`SUBSCRIPTION_ID` secrets sensitive?**
They're identifiers, not passwords — but storing them as Actions secrets is the
clean convention and keeps the workflow tidy.

---

## 10. Versioning & releases

**Q: How do you version the modules?**
Git tags following Semantic Versioning (`vMAJOR.MINOR.PATCH`).

**Q: Explain semantic versioning with examples from this repo.**
PATCH (`v1.0.1`) = bug fix, no behaviour change; MINOR (`v1.1.0`) = new feature,
backwards compatible (I bumped to `v1.1.0` when adding the linux-vm module);
MAJOR (`v2.0.0`) = a breaking change like renaming a variable.

**Q: What makes a change "breaking"?**
Anything that forces consumers to change their code — renaming/removing a
variable or output, or changing default behaviour incompatibly.

**Q: How are releases created?**
A `release.yml` workflow triggers on pushed `v*.*.*` tags and creates a GitHub
Release with auto-generated notes.

**Q: Can you tag any commit?**
A tag labels a specific commit, so I `git pull` first to ensure the merged code is
on `main` before tagging — otherwise the tag would point at code missing the new
module.

---

## 11. Branch protection & workflow

**Q: What is branch protection and what did you enforce?**
Rules on `main`: require a pull request before merging, and require the status
checks (Validate & Lint, Checkov, Plan) to pass first. So nothing broken or
unreviewed reaches `main`.

**Q: Walk me through your daily workflow.**
Create a feature branch → make changes → test locally → push → open a PR → the
pipeline runs and comments → fix anything red → get a review → merge → tag a
version when appropriate.

**Q: Why require PRs even for yourself?**
It enforces the automated gate and review on every change, builds the habit, and
keeps `main` always releasable.

**Q: A required check is failing but the change is urgent — what do you do?**
Fix the underlying issue rather than bypassing the gate. If something is a known,
justified exception, document it (e.g. a Checkov skip with a reason) — don't
disable protection.

---

## 12. Cost management

**Q: What is Infracost and where does it run?**
A tool that estimates monthly cloud cost from a Terraform plan. It runs in the
pipeline and posts the cost difference as a PR comment.

**Q: Why show cost on the PR?**
So reviewers approve changes knowing the financial impact — "this adds ~$8/month"
— instead of discovering it on the bill.

**Q: Which resource made the cost number meaningful?**
The Linux VM — it has a fixed monthly price (a `Standard_B1s` is roughly
$7–8/month), so the Infracost comment shows a real figure rather than $0.

**Q: How did you keep the demo cheap?**
LRS storage instead of GRS, `Standard_B1s` VM, `Standard_LRS` disk, and tearing
resources down after demoing.

---

## 13. Troubleshooting & debugging

**Q: You got `403 Key based authentication is not permitted`. Why and how fixed?**
The storage module sets `shared_access_key_enabled = false`, so Terraform must use
Azure AD for blob operations. Fix: `storage_use_azuread = true` in the provider
**and** the `Storage Blob Data Contributor` role on the identity.

**Q: `git push` failed with `src refspec main does not match any`. Cause?**
Either nothing was committed yet, or the branch was still `master`. Fix: commit
first, and/or `git branch -M main`, then push.

**Q: CI `terraform init` fails on a checksum/hash mismatch. Cause?**
The lock file only had Windows hashes but the runner is Linux. Fix: regenerate the
lock with both platforms recorded and commit it.

**Q: How do you debug a failing pipeline generally?**
Reproduce locally first (run the same fmt/validate/lint/checkov commands), read
the specific failing step's logs, and check OIDC subject/secret config for auth
errors.

---

## 14. Design decisions & trade-offs

**Q: Why is the pipeline plan-only instead of auto-applying?**
For a shared module library, auto-applying on every change is risky. Plan-only is
the safe default; deploys are a separate, manually triggered, optionally
approval-gated workflow.

**Q: Why keep deploy as a separate manual workflow?**
To keep `apply` a deliberate, controlled action while the PR pipeline stays
automatic and safe. Real teams gate production changes behind approval.

**Q: Why an `examples/complete` folder?**
It's runnable Terraform that exercises every module — the pipeline plans and
prices it, and it doubles as living documentation for consumers.

**Q: Why version with Git tags instead of a registry?**
Tags are simple, free, and Git-native; consumers pin with `?ref=`. A private
registry would be the next step at larger scale.

**Q: What would you do differently or add next?**
Auto-deploy on merge with an approval environment; `terraform-docs` to generate
README tables; automated tests (Terratest or `terraform test`); more modules (Key
Vault, AKS).

---

## 15. Scenario & behavioural

**Q: What was the hardest part?**
Getting OIDC right — the federated-credential subjects must match exactly per
context (PR vs branch vs environment), and Azure role propagation adds delay, so
errors like `AADSTS700213` and the storage `403` took methodical debugging.

**Q: What did you learn?**
That IaC is maybe 10% writing Terraform — the real work is versioning, testing,
security, cost, review, and safe authentication around it. The mindset shift:
"how does a team safely change this a hundred times?" not "how do I deploy once?"

**Q: How would this scale to a real team?**
Add more modules, a private module registry, environment-specific state and
workflows, mandatory reviews, and an apply pipeline with approvals per
environment.

**Q: How do you onboard a new engineer to this repo?**
Point them at the `guide/` walkthrough (Phase 0–10) and the per-module READMEs;
the consistent module structure and the example make it self-documenting.

**Q: A teammate's PR fails Checkov but they want to merge anyway. What do you do?**
Discuss the finding: fix it if it's real, or agree on a documented
`#checkov:skip` with a justification if it's a genuine, understood exception —
never disable the gate.

---

## 16. Rapid-fire definitions

- **Terraform** — declarative IaC tool.
- **HCL** — HashiCorp Configuration Language, Terraform's syntax.
- **State** — record mapping config to real resources.
- **Backend** — where state is stored (here, Azure Storage via `azurerm`).
- **Module** — reusable, parameterised unit of Terraform.
- **Provider** — plugin to an API (`azurerm`).
- **Plan / Apply** — preview changes / execute them.
- **OIDC** — passwordless federated auth between GitHub and Azure.
- **Federated credential** — Azure AD trust for a specific GitHub subject.
- **tflint** — Terraform linter (best practices).
- **Checkov** — IaC security scanner.
- **Infracost** — cloud cost estimator.
- **Semantic versioning** — `MAJOR.MINOR.PATCH` versioning scheme.
- **Branch protection** — rules requiring PRs/checks before merge.
- **CI/CD** — automated integration/delivery pipeline.
- **Idempotent** — re-running yields the same end state (Terraform's core property).

---

*Practice tip: pick 8–10 of these you'd most expect, and rehearse saying each
answer out loud in two sentences, ending with the specific choice you made in
this repo.*
