# Azure DevOps Engineer — Interview Questions & Answers

> A complete, interview-ready preparation guide covering **basic → intermediate → advanced** concepts, real-world scenarios, and hands-on troubleshooting for the **Azure DevOps Engineer** role.

---

## 📊 Total Questions & Answers: **110**

| #  | Topic / Section                                          | Questions    | Count |
|----|---------------------------------------------------------|--------------|-------|
| 1  | DevOps Fundamentals & Culture                           | Q1 – Q10     | 10    |
| 2  | Azure DevOps Platform Overview                          | Q11 – Q17    | 7     |
| 3  | Azure Boards & Agile                                    | Q18 – Q24    | 7     |
| 4  | Azure Repos & Git                                       | Q25 – Q34    | 10    |
| 5  | Branching & Merge Strategies                            | Q35 – Q40    | 6     |
| 6  | Azure Pipelines — Continuous Integration (CI)           | Q41 – Q52    | 12    |
| 7  | Azure Pipelines — Continuous Delivery/Deployment (CD)   | Q53 – Q61    | 9     |
| 8  | YAML Pipelines — Deep Dive                              | Q62 – Q70    | 9     |
| 9  | Azure Artifacts & Package Management                    | Q71 – Q75    | 5     |
| 10 | Infrastructure as Code (IaC)                            | Q76 – Q83    | 8     |
| 11 | Containers & Kubernetes (AKS)                           | Q84 – Q90    | 7     |
| 12 | Security, Secrets & DevSecOps                           | Q91 – Q97    | 7     |
| 13 | Monitoring, Logging & Observability                     | Q98 – Q102   | 5     |
| 14 | Real-World Scenario & Troubleshooting                   | Q103 – Q110  | 8     |
|    | **TOTAL**                                               |              | **110** |

---

## How to use this guide
- Read top-to-bottom for a structured ramp-up, or jump to a section using the table above.
- Each entry follows a simple structure: **Question → Answer** (detailed and beginner-friendly).
- Technical jargon and short forms are expanded in parentheses, e.g. **CI (Continuous Integration)**.
- 🎯 markers highlight **real-world / scenario-based** answers that interviewers love.

---

## 1. DevOps Fundamentals & Culture

### Q1. What is DevOps, and why do organizations adopt it?
**Answer:**
DevOps is a culture, set of practices, and tooling approach that unites **Dev (software development)** and **Ops (IT operations)** so that teams can build, test, and release software faster and more reliably. Instead of developers "throwing code over the wall" to a separate operations team, both groups share ownership of the entire application lifecycle.

Organizations adopt DevOps to achieve:
- **Faster time-to-market** — frequent, smaller releases instead of large risky ones.
- **Higher quality** — automated testing catches bugs early.
- **Reliability & stability** — automation reduces human error in deployments.
- **Collaboration** — shared responsibility breaks down silos between teams.

The four widely tracked metrics (from the **DORA — DevOps Research and Assessment** team) are: **Deployment Frequency**, **Lead Time for Changes**, **Change Failure Rate**, and **MTTR (Mean Time To Recovery)**.

### Q2. What is CI/CD?
**Answer:**
- **CI (Continuous Integration):** Developers frequently merge their code changes into a shared branch. Each merge automatically triggers a **build** and **automated tests**, so integration problems are caught within minutes rather than weeks.
- **CD** can mean two things:
  - **Continuous Delivery:** Every change that passes automated tests is automatically prepared and pushed to a release-ready state, but the final deploy to **production (live environment used by real users)** requires a manual approval/button click.
  - **Continuous Deployment:** Goes one step further — every change that passes the pipeline is **automatically** deployed to production with **no** manual gate.

🎯 In interviews, clarify the difference: *"Continuous Delivery keeps production deployment as a human decision; Continuous Deployment removes that gate entirely."*

### Q3. What is the difference between Continuous Delivery and Continuous Deployment?
**Answer:**
Both automate everything up to production. The single distinction is the **production release gate**:

| Aspect | Continuous Delivery | Continuous Deployment |
|--------|--------------------|-----------------------|
| Deploy to production | Manual approval required | Fully automatic |
| Best for | Regulated industries, business sign-off needed | Mature teams with strong automated testing |
| Risk control | Human checkpoint | Relies entirely on automated quality gates |

### Q4. What is "Infrastructure as Code" (IaC) and why does it matter?
**Answer:**
**IaC (Infrastructure as Code)** is the practice of defining and managing infrastructure (virtual machines, networks, databases, load balancers) using **code/configuration files** instead of manual clicking in a portal. The files are version-controlled in Git, reviewed, and applied automatically.

Benefits:
- **Repeatability & consistency** — the same file produces identical environments (dev, test, prod), eliminating "it works on my machine."
- **Versioning & audit** — every infrastructure change is tracked in Git history.
- **Speed** — spin up an entire environment in minutes.
- **Disaster recovery** — re-create infrastructure from code if it's lost.

Common tools: **ARM (Azure Resource Manager) templates**, **Bicep**, and **Terraform**.

### Q5. What is "Shift Left" in DevOps?
**Answer:**
"Shift Left" means moving activities — especially **testing and security** — **earlier** (to the "left") in the software delivery timeline. Instead of testing or security scanning only at the end, you do it during coding and committing.

Example: Running **SAST (Static Application Security Testing)** and unit tests on every pull request rather than waiting for a pre-release security review. Catching a bug at commit time is far cheaper than catching it in production.

### Q6. What is the difference between Agile and DevOps?
**Answer:**
They are complementary, not competing.
- **Agile** is a project-management and development *methodology* focused on iterative delivery, short sprints, and responding to change (e.g., Scrum, Kanban). It mainly addresses **how developers plan and build** software.
- **DevOps** extends those ideas across the **full lifecycle including operations and deployment**, emphasizing automation, CI/CD, and collaboration between dev and ops.

🎯 Think of it as: *Agile improves how we build software; DevOps improves how we build **and ship and run** software.*

### Q7. What is a "build artifact" and why is it important?
**Answer:**
A **build artifact** is the packaged, deployable output produced by a build process — for example a `.zip`, `.jar`, a **Docker** image, a NuGet package, or compiled binaries. 

The principle "**build once, deploy many**" is key: you build the artifact a single time, then promote that **exact same** artifact through dev → test → production. This guarantees that what you tested is exactly what you ship, avoiding inconsistencies from rebuilding per environment.

### Q8. What is "configuration drift" and how do you prevent it?
**Answer:**
**Configuration drift** happens when environments that should be identical slowly diverge — usually because someone made a manual change directly on a server or in the portal that wasn't captured in code.

Prevention:
- Use **IaC (Infrastructure as Code)** as the single source of truth.
- Disallow manual changes; enforce all changes through pipelines.
- Use **idempotent (running it repeatedly produces the same result)** tooling like Terraform that re-applies the desired state.
- Run periodic drift detection (e.g., `terraform plan` to compare actual vs. declared state).

### Q9. What are blue-green and canary deployment strategies?
**Answer:**
- **Blue-Green Deployment:** You run **two identical production environments** — "Blue" (current live) and "Green" (new version). You deploy and test on Green, then switch all traffic from Blue to Green instantly (via a load balancer or DNS swap). If something breaks, you switch back to Blue immediately. **Pros:** instant rollback. **Cons:** double the infrastructure cost.
- **Canary Deployment:** You release the new version to a **small subset of users first** (e.g., 5%), monitor health/metrics, and gradually increase to 100% if all looks good. **Pros:** limits blast radius of bugs. **Cons:** more complex traffic routing and monitoring.

🎯 Related: **Rolling deployment** updates instances in batches, and **feature flags (toggles that turn features on/off without redeploying)** allow releasing code dark and enabling it later.

### Q10. What is idempotency, and why does it matter in automation?
**Answer:**
**Idempotency** means an operation can be applied multiple times and the result stays the same after the first successful run. In automation, an idempotent script can be re-run safely without causing duplicate resources or errors.

Example: A non-idempotent script that runs `CREATE` will fail or create duplicates if run twice. An idempotent IaC tool checks current state first and only changes what's needed — re-running it when everything already matches makes **no changes**. This is crucial for reliable, retry-safe pipelines.

---

## 2. Azure DevOps Platform Overview

### Q11. What is Azure DevOps and what are its main services?
**Answer:**
**Azure DevOps** is Microsoft's cloud-based (also available self-hosted as **Azure DevOps Server**) suite of tools that supports the entire software development lifecycle. It has five core services:

1. **Azure Boards** — Agile planning, work-item tracking, backlogs, sprints, Kanban boards.
2. **Azure Repos** — Source control hosting (**Git** and the older **TFVC — Team Foundation Version Control**).
3. **Azure Pipelines** — CI/CD automation for building, testing, and deploying.
4. **Azure Artifacts** — Hosting and sharing packages (NuGet, npm, Maven, Python, Universal).
5. **Azure Test Plans** — Manual and exploratory test management.

### Q12. What is the difference between Azure DevOps Services and Azure DevOps Server?
**Answer:**
- **Azure DevOps Services** — the **cloud-hosted (SaaS — Software as a Service)** version. Microsoft manages the infrastructure, updates, and scaling. Accessed via `dev.azure.com`.
- **Azure DevOps Server** — the **on-premises (installed on your own servers)** version, formerly called **TFS (Team Foundation Server)**. You manage the hardware, backups, and upgrades yourself. Chosen by organizations with strict data-residency or compliance requirements.

### Q13. What is an Organization, Project, and Team in Azure DevOps?
**Answer:**
This is the hierarchy:
- **Organization** — the top-level container (e.g., `dev.azure.com/yourcompany`). It groups related projects and manages billing and global settings.
- **Project** — a container for a product or initiative. It holds its own repos, pipelines, boards, and artifacts. Provides a security/isolation boundary.
- **Team** — a group of people within a project. Each team gets its own backlog, board, and dashboards, enabling multiple squads to work in one project independently.

### Q14. How is access control managed in Azure DevOps?
**Answer:**
Azure DevOps uses **RBAC (Role-Based Access Control)** through:
- **Security Groups** — built-in groups like *Project Administrators*, *Contributors*, *Readers*, plus custom groups.
- **Permissions** — granular Allow/Deny settings on objects (repos, pipelines, branches).
- **Azure AD / Microsoft Entra ID integration** — users and groups are synced from your identity provider for centralized management.
- **PATs (Personal Access Tokens)** — scoped tokens for tools/scripts to authenticate without using a password.

🎯 Best practice: follow **least privilege** — grant only the minimum permissions needed, and prefer group-based assignment over per-user.

### Q15. What are Service Connections in Azure DevOps?
**Answer:**
A **Service Connection** is a stored, secure configuration that lets a pipeline authenticate to and interact with an **external system** — such as an Azure subscription, a Docker registry, a Kubernetes cluster, GitHub, or SonarQube.

For Azure, the most common is the **Azure Resource Manager service connection**, which uses a **Service Principal (a non-human identity / app registration in Azure AD)** or, more securely, **Workload Identity Federation (passwordless, token-based trust)**. This avoids storing long-lived secrets in the pipeline.

### Q16. What is the difference between Microsoft-hosted and self-hosted agents?
**Answer:**
An **agent** is the machine that actually runs your pipeline jobs.
- **Microsoft-hosted agents** — Microsoft provides fresh, fully-managed VMs (Virtual Machines) for each run. They come pre-installed with common tools, are wiped after each job, and require zero maintenance. **Limits:** ~60-minute timeout on free tiers, no persistent caching, fixed specs.
- **Self-hosted agents** — VMs or machines **you** provision and maintain. Use them when you need: custom software, more CPU/RAM, access to a private network/on-prem resources, faster builds via local caching, or longer-running jobs.

🎯 Common real-world reason to use self-hosted: deploying to resources inside a **private VNet (Virtual Network)** that Microsoft-hosted agents can't reach.

### Q17. What is an Agent Pool?
**Answer:**
An **Agent Pool** is a group of agents that share a common purpose, so you don't manage agents one-by-one in each project. Pipelines target a pool (e.g., `Azure Pipelines` for Microsoft-hosted, or a custom `Linux-Build-Pool`), and the system picks an available agent from it. Pools can be shared across multiple projects in an organization, which simplifies scaling and governance.

---

## 3. Azure Boards & Agile

### Q18. What are Work Items in Azure Boards?
**Answer:**
A **Work Item** is a trackable unit of work. Types include **Epic** (large body of work), **Feature** (a shippable capability), **User Story / Product Backlog Item (PBI)** (a small requirement from a user's perspective), **Task** (a granular to-do), and **Bug** (a defect). Work items have fields like assignee, state, priority, and can be linked to each other (parent/child) and to commits, pull requests, and builds for full traceability.

### Q19. What is the difference between the Basic, Agile, Scrum, and CMMI process templates?
**Answer:**
A **process template** defines which work-item types and workflows your project uses:
- **Basic** — simplest: Epics, Issues, Tasks. Great for small teams getting started.
- **Agile** — uses User Stories and Bugs; states like New → Active → Resolved → Closed.
- **Scrum** — uses Product Backlog Items (PBIs) and Bugs; states like New → Approved → Committed → Done. Aligns with Scrum terminology.
- **CMMI (Capability Maturity Model Integration)** — most formal; adds Requirements, Change Requests, and Risk tracking for organizations needing heavy process and auditing.

### Q20. What is a Sprint and how is it managed in Azure Boards?
**Answer:**
A **Sprint** (also called an **iteration**) is a fixed time-box (commonly 2 weeks) during which a team commits to completing a set of work items. In Azure Boards you define **Iteration Paths** (the sprint timeline) and **Area Paths** (which team/component owns the work). The **Sprint Backlog** and **Taskboard** let the team plan capacity, track progress, and visualize remaining work.

### Q21. What is a Burndown chart?
**Answer:**
A **Burndown chart** shows the amount of remaining work (in hours or story points) versus time over a sprint. The ideal line slopes down to zero by sprint end. If the actual line stays flat or above the ideal, the team is behind. It helps detect scope creep and forecast whether the sprint goal will be met. The related **Burnup chart** shows completed work rising toward the total scope.

### Q22. What is a Kanban board and what is WIP limit?
**Answer:**
A **Kanban board** visualizes work as cards moving through columns (e.g., To Do → In Progress → Done). It emphasizes continuous flow rather than fixed sprints.

A **WIP (Work In Progress) limit** caps how many items can sit in a column at once. This prevents overload, exposes bottlenecks, and forces the team to *finish* work before starting new work — improving throughput.

### Q23. How do you link a commit or pull request to a work item?
**Answer:**
In Azure DevOps you reference a work item by its ID using `#` in a commit message or PR (Pull Request) description — for example, `Fixed login bug #1234`. Azure automatically creates a link between the code change and the work item. This gives **end-to-end traceability**: from a planned requirement → the exact code → the build → the deployment that delivered it, which is invaluable for audits and debugging.

### Q24. What are Queries and Dashboards in Azure Boards?
**Answer:**
- **Queries** are saved searches over work items using filters (e.g., "all active bugs assigned to me, priority 1"). They can be flat lists, tree (parent/child), or direct-links views, and can be shared with the team.
- **Dashboards** are customizable pages of **widgets** (charts, query tiles, build/release status, burndown) that give stakeholders an at-a-glance view of project health without digging into individual items.

---

## 4. Azure Repos & Git

### Q25. What is the difference between Git and TFVC?
**Answer:**
Both are version-control systems offered by Azure Repos:
- **Git** — a **distributed** version control system (DVCS). Every developer has a full local copy of the repository and its history, enabling offline work, fast branching, and flexible workflows. This is the modern default.
- **TFVC (Team Foundation Version Control)** — a **centralized** system. There's one server copy; developers check out/check in files. Better suited to very large monorepos or teams that need fine-grained file-level locking, but largely legacy now.

🎯 Recommendation in interviews: choose **Git** for almost all new projects.

### Q26. Explain the basic Git workflow (clone, add, commit, push, pull).
**Answer:**
- **clone** — copy a remote repository to your local machine.
- **add** (`git add`) — stage changed files, marking them to be included in the next commit.
- **commit** (`git commit`) — save a snapshot of staged changes to your local history with a message.
- **push** (`git push`) — upload your local commits to the remote repository.
- **pull** (`git pull`) — fetch and merge the latest changes from the remote into your local branch (it's a `git fetch` + `git merge` combined).

### Q27. What is the difference between `git merge` and `git rebase`?
**Answer:**
Both integrate changes from one branch into another, but differently:
- **`git merge`** — creates a new "merge commit" that ties the two branch histories together. History is preserved exactly as it happened (non-linear, with branch lines). Safe and non-destructive.
- **`git rebase`** — moves your branch's commits to *replay* on top of the latest target branch, producing a **linear, clean history** as if you'd written your code after the latest changes. It **rewrites commit history**, so you should never rebase commits that others have already pulled (shared branches).

🎯 Rule of thumb: *rebase your local/private branch to stay clean; merge into shared branches.*

### Q28. What is the difference between `git fetch` and `git pull`?
**Answer:**
- **`git fetch`** downloads new commits/branches from the remote but **does not** change your working files — it just updates your local knowledge of the remote. You can review before integrating.
- **`git pull`** does a `fetch` **and immediately merges** (or rebases) those changes into your current branch. It's a convenience but can cause surprise conflicts, so cautious developers often `fetch` then review before merging.

### Q29. What is a Pull Request (PR) and why is it important?
**Answer:**
A **Pull Request (PR)** is a request to merge changes from one branch into another (e.g., a feature branch into `main`). It's the central place for **code review**: teammates inspect the diff, leave comments, and approve or request changes. PRs enable:
- Quality control via peer review.
- Automated checks (build + tests must pass before merge).
- Discussion and knowledge sharing.
- An audit trail of who approved what.

### Q30. What are Branch Policies in Azure Repos?
**Answer:**
**Branch policies** are rules that protect important branches (like `main`) by enforcing conditions before code can merge:
- **Require a minimum number of reviewers** (e.g., 2 approvals).
- **Check for linked work items** — ensures traceability.
- **Require comment resolution** — all PR comments must be addressed.
- **Build validation** — a CI build/tests must pass.
- **Status checks** — external tools (e.g., security scanners) must report success.
- **Limit merge types** (e.g., only squash merge).

🎯 This is a very common interview topic — branch policies are how you enforce quality *automatically*.

### Q31. What is a `.gitignore` file?
**Answer:**
A `.gitignore` file lists patterns of files and folders that Git should **not track** — such as build outputs (`bin/`, `obj/`), dependency folders (`node_modules/`), secrets (`.env`), and IDE settings. This keeps the repository clean, smaller, and prevents accidentally committing sensitive or machine-specific files.

### Q32. What is a merge conflict and how do you resolve it?
**Answer:**
A **merge conflict** occurs when two branches change the **same lines** of the same file (or one edits a file the other deletes), and Git can't automatically decide which version to keep. Git marks the conflicting region with markers (`<<<<<<<`, `=======`, `>>>>>>>`). 

To resolve: open the file, manually edit it to the correct final state, remove the markers, then `git add` the resolved file and complete the merge/commit. Good practices to minimize conflicts: pull frequently, keep branches short-lived, and make small focused commits.

### Q33. What are the different merge strategies when completing a PR in Azure Repos?
**Answer:**
- **Merge (no fast-forward)** — keeps all commits from the branch plus a merge commit; preserves full history.
- **Squash merge** — combines all the branch's commits into a single clean commit on the target. Keeps `main` history tidy.
- **Rebase and fast-forward** — replays the branch commits onto the target with no merge commit (linear history).
- **Semi-linear merge (rebase + merge commit)** — rebases first, then adds a merge commit for a hybrid of clean + traceable history.

### Q34. How do you revert a bad commit that's already been pushed?
**Answer:**
Use **`git revert <commit-hash>`**. Unlike `git reset` (which rewrites history and is dangerous on shared branches), `git revert` creates a **new commit that undoes** the changes of the bad commit, preserving history. This is the safe approach for shared/remote branches. For a not-yet-pushed local mistake, `git reset` is acceptable.

---

## 5. Branching & Merge Strategies

### Q35. What is a branching strategy and why is it needed?
**Answer:**
A **branching strategy** is an agreed-upon convention for how a team creates, names, and merges branches. It prevents chaos when many developers work simultaneously, defines where releases are cut from, and keeps the mainline stable. Popular strategies include **Git Flow**, **GitHub Flow**, and **Trunk-Based Development**.

### Q36. Explain Git Flow.
**Answer:**
**Git Flow** uses multiple long-lived branches:
- **`main`/`master`** — always production-ready.
- **`develop`** — integration branch for the next release.
- **`feature/*`** — branched from `develop` for new work, merged back to `develop`.
- **`release/*`** — created from `develop` to stabilize a release, then merged to `main` and `develop`.
- **`hotfix/*`** — branched from `main` to fix urgent production bugs.

**Pros:** very structured, good for scheduled releases. **Cons:** complex, many long-lived branches, slower flow — less ideal for continuous delivery.

### Q37. Explain Trunk-Based Development.
**Answer:**
In **Trunk-Based Development**, all developers commit to a single shared branch (the "trunk", usually `main`) at least daily, using very **short-lived feature branches** (hours to a couple of days) that merge quickly. Incomplete features are hidden behind **feature flags**.

**Pros:** minimizes merge conflicts, enables true CI/CD, fast feedback. **Cons:** demands strong automated testing and discipline. 🎯 It's the strategy most aligned with high-performing DevOps teams.

### Q38. Explain GitHub Flow.
**Answer:**
**GitHub Flow** is a lightweight middle ground: `main` is always deployable; you create a descriptively-named branch off `main` for any change, open a PR, get it reviewed and tested, then merge back to `main` and deploy. There's no separate `develop` or `release` branch. It's simple and works well for web apps with continuous deployment.

### Q39. What is a feature flag (feature toggle) and how does it relate to branching?
**Answer:**
A **feature flag** is a configuration switch that turns a piece of functionality on or off **at runtime without redeploying code**. It lets teams merge unfinished features into `main` (kept "off") and enable them later for specific users or environments. This reduces long-lived branches and supports trunk-based development, canary releases, and quick rollback (just flip the flag off) without a code deploy.

### Q40. When would you choose Git Flow vs. Trunk-Based Development?
**Answer:**
🎯 Scenario answer:
- Choose **Git Flow** when you have **scheduled, versioned releases** (e.g., enterprise software, mobile apps with app-store review cycles), multiple versions in support, and need strong release isolation.
- Choose **Trunk-Based Development** when you want **fast, frequent deployments** (e.g., SaaS web apps), have mature automated testing, and aim for continuous delivery/deployment.

The trend in modern DevOps strongly favors trunk-based development for its speed and reduced merge pain.

---

## 6. Azure Pipelines — Continuous Integration (CI)

### Q41. What is Azure Pipelines?
**Answer:**
**Azure Pipelines** is the CI/CD service in Azure DevOps that automatically **builds, tests, and deploys** code. It supports any language/platform, runs on Linux, Windows, and macOS agents, integrates with Azure Repos, GitHub, Bitbucket, etc., and can be defined either via **YAML (a configuration-as-code file)** or the older **Classic visual editor**.

### Q42. What is the difference between YAML pipelines and Classic pipelines?
**Answer:**
- **YAML pipelines** — defined in a `azure-pipelines.yml` file stored **in the repo alongside your code**. This means the pipeline is **version-controlled, reviewable in PRs, and portable**. It's the modern, recommended approach.
- **Classic pipelines** — configured through a **drag-and-drop graphical UI** in the portal. Easier for beginners but not stored in source control, harder to review/audit, and harder to replicate across projects.

🎯 Always prefer YAML for "**pipeline as code**" benefits.

### Q43. Explain the structure of a YAML pipeline (stages, jobs, steps).
**Answer:**
A YAML pipeline has a hierarchy:
- **Trigger** — what starts the pipeline (e.g., a push to `main`).
- **Stage** — a major phase like *Build*, *Test*, *Deploy*. Stages run sequentially by default and are good logical/approval boundaries.
- **Job** — a unit of work that runs on a single agent. Jobs in a stage can run in **parallel**.
- **Step** — an individual action inside a job — either a **task** (pre-packaged action) or a **script** (shell/PowerShell command).

```
trigger → stages → jobs → steps (tasks/scripts)
```

### Q44. What are triggers in Azure Pipelines? Name the types.
**Answer:**
A **trigger** defines what automatically starts a pipeline:
- **CI trigger (push trigger)** — runs when code is pushed to specified branches/paths.
- **PR trigger** — runs when a pull request is opened/updated (great for build validation).
- **Scheduled trigger** — runs on a cron schedule (e.g., nightly builds).
- **Pipeline (resource) trigger** — runs when another pipeline completes (pipeline chaining).
- **Manual trigger** — started by a person on demand.

You can also use **path filters** (only trigger when certain folders change) and **branch filters**.

### Q45. What is a build task vs. a script step?
**Answer:**
- A **task** is a reusable, pre-built unit of automation provided by Microsoft or the marketplace (e.g., `DotNetCoreCLI@2`, `Docker@2`, `PublishBuildArtifacts@1`). You configure it with inputs.
- A **script step** runs raw commands directly: `script:` (default shell), `bash:`, `pwsh:` (PowerShell Core), or `powershell:`. Use scripts for custom logic not covered by an existing task.

### Q46. What are pipeline variables and how do they differ from parameters?
**Answer:**
- **Variables** — name/value pairs resolved at **runtime**, used to avoid hardcoding (e.g., `buildConfiguration: 'Release'`). They can be defined inline, in **variable groups**, or come from **Key Vault**.
- **Parameters** (in YAML templates) — typed inputs resolved at **compile/template-expansion time** (before the run starts). They can change the *structure* of the pipeline (e.g., conditionally add a stage), which variables cannot.

🎯 Key distinction: *parameters are evaluated first (template time) and can shape the pipeline; variables are evaluated later (runtime) and supply values.*

### Q47. What are Variable Groups and how are they linked to Key Vault?
**Answer:**
A **Variable Group** is a set of variables stored in the **Library** that can be shared across multiple pipelines (e.g., shared connection strings, environment names). This avoids duplicating variables.

A variable group can be **linked to Azure Key Vault**, so secret values (passwords, API keys) are pulled securely from Key Vault at runtime rather than stored as plain text — improving security and centralizing secret management.

### Q48. How do you secure secrets in a pipeline?
**Answer:**
- Mark variables as **secret** (they're masked in logs and encrypted).
- Store secrets in **Azure Key Vault** and reference them via a linked variable group.
- Use **Service Connections** with **Workload Identity Federation** instead of long-lived credentials.
- Never echo secrets in logs; Azure auto-masks known secrets but custom handling is needed for derived values.
- Restrict who can edit pipelines and variable groups via permissions.

### Q49. What is a build artifact in Azure Pipelines and how do you publish/consume it?
**Answer:**
A **build artifact** is the output of a build stage that you want to pass to later stages or releases. You **publish** it with the `PublishPipelineArtifact@1` (or `PublishBuildArtifacts`) task, giving it a name. Later stages/jobs **download** it with `DownloadPipelineArtifact@2`. This implements "build once, deploy many" — the same artifact flows through every environment.

### Q50. What is pipeline caching and why use it?
**Answer:**
**Caching** stores reusable files (like downloaded dependencies — `node_modules`, NuGet/Maven packages) between runs so the pipeline doesn't re-download them every time. Using the `Cache@2` task with a key (often a hash of the lock file) can dramatically **speed up builds**. If the key matches a previous run, the cache is restored; otherwise it's rebuilt and saved.

### Q51. What is the difference between a multi-stage pipeline and a single-stage pipeline?
**Answer:**
- A **single-stage pipeline** does everything (build/test) in one stage — typical for pure CI.
- A **multi-stage pipeline** separates concerns into multiple stages (e.g., *Build → Deploy-Dev → Deploy-Test → Deploy-Prod*), enabling **CI and CD in one YAML file**, with **approvals and gates** between stages. This is the modern way to do full CI/CD as code.

### Q52. How do you run jobs in parallel and define dependencies between them?
**Answer:**
By default, jobs within a stage run in **parallel** (subject to available parallel agents). To control order, use **`dependsOn`** — a job/stage waits for the listed ones to finish. You can also add **`condition`** (e.g., run only if the previous stage succeeded, or even if it failed). Example: a *Deploy* stage `dependsOn: Build` and `condition: succeeded()`. This builds a dependency graph (DAG — Directed Acyclic Graph) controlling execution flow.

---

## 7. Azure Pipelines — Continuous Delivery / Deployment (CD)

### Q53. What is an Environment in Azure Pipelines?
**Answer:**
An **Environment** is a named collection of resources you deploy to (e.g., `dev`, `staging`, `production`). It provides:
- **Deployment history** — what was deployed when and by which run.
- **Approvals and checks** — gate deployments to that environment.
- **Resource targeting** — for Kubernetes namespaces or VMs registered to the environment.

It's the modern replacement for the "stages" concept in classic Release pipelines, and key to safe CD.

### Q54. What are Approvals and Checks?
**Answer:**
- **Approvals** — a manual gate where designated people must click "approve" before a deployment to a protected environment proceeds (e.g., a manager approves production releases).
- **Checks** — automated gates on an environment or service connection, such as: **Invoke Azure Function**, **Query Azure Monitor alerts** (block deploy if alerts are firing), **business hours** check, or **required template**. 

🎯 Together they enforce governance — you can't accidentally ship to prod without sign-off and healthy metrics.

### Q55. What is a deployment job and what strategies does it support?
**Answer:**
A **deployment job** (`deployment:` keyword) is a special job type that targets an Environment, records deployment history, and supports built-in **deployment strategies**:
- **runOnce** — deploys once, simplest.
- **rolling** — updates instances in batches to limit downtime.
- **canary** — deploys to a small percentage first, then increases.

Each strategy has lifecycle hooks (`preDeploy`, `deploy`, `routeTraffic`, `postRouteTraffic`, `on: failure/success`) for fine control.

### Q56. How do you implement a blue-green deployment in Azure?
**Answer:**
🎯 A common approach using **Azure App Service deployment slots**:
1. Deploy the new version to a **staging slot** (the "green" slot) — this doesn't affect live traffic.
2. Run smoke tests and warm-up against the staging slot.
3. Perform a **slot swap** — Azure swaps the staging and production slots, instantly routing live traffic to the new version with near-zero downtime.
4. If problems appear, **swap back** to roll back instantly.

For AKS or VMs, you'd use load balancer/traffic-manager routing between two environments instead of slots.

### Q57. What is a rollback strategy and how do you implement one?
**Answer:**
A **rollback** returns the system to the last known-good state after a failed deployment. Approaches:
- **Slot swap back** (App Service) — instant reverse.
- **Redeploy previous artifact** — pipelines store prior artifacts, so re-run the deploy stage with the last good version.
- **Blue-green switch** — route traffic back to the old environment.
- **Database migrations** — must be backward-compatible (expand/contract pattern) so rolling back code doesn't break the DB.
- **Feature flags** — disable the problematic feature without redeploying.

### Q58. What are deployment gates in Azure Pipelines?
**Answer:**
**Gates** are automated, time-based health checks evaluated **before or after** a deployment without human involvement. Examples: query **Azure Monitor / Application Insights** for error rates, check that work items are in the right state, call a REST API, or ensure no active incidents. If the gate condition isn't met within a timeout, the deployment is paused or rejected — protecting production from unhealthy releases.

### Q59. How do you deploy to multiple environments with promotion?
**Answer:**
Use a **multi-stage pipeline** where each environment is a stage chained with `dependsOn`: Build → Deploy-Dev (auto) → Deploy-Test (auto after dev) → Deploy-Prod (requires approval). The **same artifact** is promoted through each stage. Approvals/checks guard the later environments. This gives a controlled, auditable promotion path with increasing scrutiny toward production.

### Q60. What is the difference between a release pipeline (classic) and a multi-stage YAML pipeline?
**Answer:**
- **Classic Release pipeline** — a separate, UI-based artifact-driven deployment tool with stages, gates, and approvals configured visually. Not stored in source control.
- **Multi-stage YAML pipeline** — combines build and deploy stages in one version-controlled YAML file, with environments, approvals, and checks. It's the recommended modern approach because the entire CI/CD process lives as code in the repo.

### Q61. How do you handle database deployments in a CI/CD pipeline?
**Answer:**
🎯 Database changes are riskier than code because data is stateful. Best practices:
- Use **migration-based tooling** (e.g., **DACPAC** with SqlPackage, EF Core Migrations, Flyway, or Liquibase) to apply versioned, repeatable schema changes.
- Make changes **backward-compatible** using the **expand/contract pattern**: first add new columns/tables (expand), deploy code, then later remove old ones (contract) — so old and new code can coexist during rollout.
- Always **back up** before production migrations.
- Test migrations in lower environments first.
- Keep schema scripts in source control alongside app code.

---

## 8. YAML Pipelines — Deep Dive

### Q62. What are templates in Azure Pipelines and why use them?
**Answer:**
**Templates** are reusable YAML files that you reference from multiple pipelines to avoid duplication (DRY — Don't Repeat Yourself). Types:
- **Stage / Job / Step templates** — reuse blocks of pipeline logic.
- **Variable templates** — reuse variable definitions.

You pass **parameters** to customize them. Templates standardize practices (e.g., a single "build-and-scan" template used by every team) and make organization-wide changes in one place.

### Q63. What is the difference between compile-time and runtime expressions?
**Answer:**
- **Compile-time / template expressions** — written as `${{ }}`. Evaluated **before** the pipeline runs, during YAML expansion. Used with parameters to conditionally include stages/jobs or loops (`each`).
- **Runtime expressions** — written as `$[ ]`. Evaluated **while** the pipeline runs, useful for values not known until runtime (e.g., another job's output variable).
- **Macro syntax** — `$(varName)`. Simple variable substitution at runtime.

🎯 Knowing which expression runs when is a frequent advanced interview question.

### Q64. How do you pass output from one job/stage to another?
**Answer:**
Set an **output variable** in a step using the logging command:
```bash
echo "##vso[task.setvariable variable=myVar;isOutput=true]someValue"
```
Then reference it in a dependent job via `dependencies`:
```yaml
$[ dependencies.JobA.outputs['stepName.myVar'] ]
```
For stages, use `stageDependencies`. The producing job must be in `dependsOn` of the consumer. This enables data flow across the pipeline.

### Q65. What are conditions in YAML pipelines? Give examples.
**Answer:**
**Conditions** control whether a stage/job/step runs. Built-in functions include:
- `succeeded()` — run only if previous succeeded (default).
- `failed()` — run only if something failed (useful for cleanup/notification).
- `always()` — run regardless of outcome.
- `eq(variables['Build.SourceBranch'], 'refs/heads/main')` — run only on the `main` branch.

Example: deploy to prod only on main branch AND only if build succeeded:
```yaml
condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

### Q66. What is the `each` keyword / loops in YAML?
**Answer:**
The `${{ each item in parameters.list }}` syntax iterates at **template-expansion time** to generate repeated blocks — for example, creating one deployment job per environment from a parameter array. It reduces copy-paste and keeps multi-environment pipelines maintainable.

### Q67. What are matrix and parallel strategies in jobs?
**Answer:**
- **Matrix** — runs the **same job multiple times with different variable combinations** in parallel (e.g., test across Node 16, 18, 20, or Windows + Linux). Defined under `strategy: matrix:`.
- **parallel** (maxParallel / slicing) — splits work (like tests) into N parallel agents to finish faster.

These speed up testing across configurations and large test suites.

### Q68. How do you reuse pipelines across projects (extends and required templates)?
**Answer:**
- **`extends`** — a pipeline can extend a base template that defines the allowed structure. Combined with the **"Require a specific template" check** on an environment, this **enforces governance**: deployments are only allowed if they extend an approved, security-reviewed template. This prevents teams from bypassing required steps (like security scans).

### Q69. What are resources in a YAML pipeline?
**Answer:**
The `resources` section declares external things the pipeline depends on:
- **repositories** — other repos (e.g., shared templates) to check out.
- **pipelines** — other pipelines to trigger from or consume artifacts of.
- **containers** — container images to run jobs inside.
- **packages** — package resources.

This enables cross-repo templates, pipeline chaining, and containerized jobs.

### Q70. How do you debug a failing pipeline?
**Answer:**
🎯 Practical steps:
1. Read the **failed step's logs** top-to-bottom for the actual error.
2. Enable **system diagnostics** — set variable `system.debug = true` for verbose logging.
3. Reproduce locally where possible (same commands the script runs).
4. Check **agent capabilities** vs. demands (missing tool/version).
5. Verify **service connection** auth and permissions.
6. Check variable/secret values are resolving (watch for empty secrets).
7. Use **`condition: always()`** steps to print environment/diagnostic info.
8. For flaky tests, isolate and re-run; check for race conditions or environment state.

---

## 9. Azure Artifacts & Package Management

### Q71. What is Azure Artifacts?
**Answer:**
**Azure Artifacts** is a service for hosting, storing, and sharing **packages** — including **NuGet (.NET)**, **npm (Node.js)**, **Maven/Gradle (Java)**, **Python (PyPI)**, and **Universal Packages** (any file blob). It lets teams publish their own packages and consume public ones through a single feed, with versioning and access control.

### Q72. What is a Feed in Azure Artifacts?
**Answer:**
A **Feed** is a container/repository for packages. You publish packages to a feed and configure your build/dev tools to restore from it. Feeds have **permissions** (who can read/publish) and **views** (`@local`, `@prerelease`, `@release`) so you can promote a package from prerelease to release-quality. Feeds can be scoped to a project or shared across the organization.

### Q73. What are Upstream Sources?
**Answer:**
**Upstream sources** let a single feed transparently proxy and cache packages from **public registries** (like nuget.org or npmjs.com) alongside your private packages. Benefits:
- One feed URL for both internal and public packages.
- **Caching** improves reliability (builds don't break if the public registry is down) and can speed restores.
- Captured versions are retained, protecting against a public package being removed.

### Q74. How do you version packages, and what is semantic versioning?
**Answer:**
**Semantic Versioning (SemVer)** uses `MAJOR.MINOR.PATCH` (e.g., `2.4.1`):
- **MAJOR** — incompatible/breaking changes.
- **MINOR** — new backward-compatible features.
- **PATCH** — backward-compatible bug fixes.
Pre-release suffixes like `-beta` or `-rc1` indicate non-final builds. Consistent versioning lets consumers safely choose updates and lets pipelines auto-increment versions on each build.

### Q75. How do you consume an Azure Artifacts feed in a pipeline?
**Answer:**
Use the appropriate restore task with the feed configured — e.g., `NuGetCommand@2` or `DotNetCoreCLI@2` with `restore` and the feed referenced (often via a `nuget.config` pointing to the feed, with authentication handled by the built-in pipeline identity or the `NuGetAuthenticate@1` task). For npm, the `npmAuthenticate@0` task injects credentials for the feed's `.npmrc`.

---

## 10. Infrastructure as Code (IaC)

### Q76. What are ARM templates?
**Answer:**
**ARM (Azure Resource Manager) templates** are **JSON files** that declaratively define Azure resources (the desired end state). You submit the template to Azure Resource Manager, which figures out how to create/update the resources. They support parameters, variables, and outputs. **Declarative** means you describe *what* you want, not the step-by-step *how*. The downside is JSON is verbose and hard to read at scale — which led to Bicep.

### Q77. What is Bicep and how does it relate to ARM?
**Answer:**
**Bicep** is a **DSL (Domain-Specific Language)** that compiles down to ARM JSON. It's Microsoft's modern IaC language for Azure with much cleaner, more concise syntax than raw ARM, better tooling (IntelliSense), modules, and no need to manage JSON braces. Anything ARM can do, Bicep can do — it **transpiles** to ARM at deploy time. 🎯 For Azure-only IaC, Bicep is the recommended choice today.

### Q78. What is Terraform and how does it differ from ARM/Bicep?
**Answer:**
**Terraform** (by HashiCorp) is a popular **cloud-agnostic** IaC tool using its own language **HCL (HashiCorp Configuration Language)**. Key differences:
- **Multi-cloud** — works with Azure, AWS, GCP, and hundreds of providers (ARM/Bicep are Azure-only).
- **State file** — Terraform tracks deployed resources in a **state file** (which ARM/Bicep don't keep separately; Azure itself is the source of truth).
- **Plan/Apply workflow** — `terraform plan` previews changes before `terraform apply`.

🎯 Choose Terraform for multi-cloud or if your org already standardizes on it; choose Bicep for pure Azure with tightest day-one feature support.

### Q79. What is Terraform state and why does remote state matter?
**Answer:**
The **state file** (`terraform.tfstate`) is Terraform's record of what it has provisioned, mapping config to real resources. It's critical and sensitive (can contain secrets). 

**Remote state** stores this file in a shared backend (e.g., an **Azure Storage Account** with a blob container) instead of locally. This enables:
- **Team collaboration** — everyone uses the same state.
- **State locking** — prevents two people applying at once (avoiding corruption).
- **Durability & security** — backed up and access-controlled.

### Q80. What does idempotency mean in Terraform, and what does `terraform plan` do?
**Answer:**
Terraform is **idempotent**: it compares your declared configuration against the current state and only makes the changes needed to reconcile them. Running `apply` when everything already matches results in **no changes**.

**`terraform plan`** produces a **dry-run preview** showing what will be added, changed, or destroyed — without touching anything. Reviewing the plan (often in a PR) is a key safety step before `apply`.

### Q81. How do you securely run Terraform in an Azure DevOps pipeline?
**Answer:**
🎯 Practical setup:
- Authenticate via a **Service Connection (Service Principal or Workload Identity Federation)** — no secrets in code.
- Store **remote state** in an Azure Storage Account with locking.
- Keep secrets in **Key Vault**, injected at runtime.
- Pipeline flow: `terraform init` → `terraform validate` → `terraform plan` (publish plan output for review) → manual **approval** → `terraform apply` using the saved plan file.
- Use environments + approvals so production applies require sign-off.

### Q82. What are Terraform modules?
**Answer:**
A **module** is a reusable, parameterized package of Terraform configuration (e.g., a "standard web app" module that creates an App Service, plan, and Key Vault together). Modules promote reuse, consistency, and DRY principles — teams call the same module with different inputs instead of copying resource blocks. They can be stored locally, in a registry, or in a Git repo.

### Q83. What is the difference between declarative and imperative IaC?
**Answer:**
- **Declarative** — you describe the **desired end state**, and the tool figures out the steps (e.g., ARM, Bicep, Terraform). Easier to reason about and naturally idempotent.
- **Imperative** — you specify the **exact sequence of commands** to reach the state (e.g., a shell script with `az cli` create commands). More control but harder to make idempotent and maintain.

🎯 Modern IaC strongly favors the declarative model.

---

## 11. Containers & Kubernetes (AKS)

### Q84. What is a container and how does it differ from a virtual machine?
**Answer:**
A **container** packages an application with its dependencies into a single, portable, isolated unit that runs consistently anywhere. 

- **Containers** share the **host operating system's kernel** and isolate at the process level — they're **lightweight**, start in seconds, and use less resource.
- **VMs (Virtual Machines)** each run a **full guest operating system** on a hypervisor — heavier, slower to start, but with stronger isolation.

🎯 Containers solve "it works on my machine" by bundling the runtime environment.

### Q85. What is Docker and what is a Dockerfile?
**Answer:**
**Docker** is the most popular platform for building and running containers. A **Dockerfile** is a text file of instructions to build a container **image** (the template), e.g.:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```
Each instruction creates a cached **layer**. A running instance of an image is a **container**.

### Q86. What is a multi-stage Docker build?
**Answer:**
A **multi-stage build** uses multiple `FROM` statements in one Dockerfile. An early "build" stage compiles the app (with all the heavy build tools/SDKs), and a final lightweight stage copies only the compiled output. This produces a **much smaller, more secure final image** because build tools and source aren't shipped. 🎯 It's a best practice interviewers like to hear.

### Q87. What is Kubernetes and what problem does it solve?
**Answer:**
**Kubernetes (K8s)** is an open-source **container orchestration** platform. When you run many containers across many machines, you need automated **scheduling, scaling, self-healing, networking, and rollouts** — Kubernetes provides all of this. It keeps the desired number of container replicas running, restarts failed ones, load-balances traffic, and enables zero-downtime updates.

### Q88. What is AKS and what does it manage for you?
**Answer:**
**AKS (Azure Kubernetes Service)** is Microsoft's **managed Kubernetes** offering. Azure manages the **control plane (the Kubernetes "brain" — API server, scheduler, etcd)** for free, handling its availability and upgrades. You manage and pay only for the **worker nodes (the VMs that run your containers)**. AKS integrates with Azure AD, Azure Monitor, Key Vault, and virtual networks, reducing operational overhead.

### Q89. Explain key Kubernetes objects: Pod, Deployment, Service, Ingress.
**Answer:**
- **Pod** — the smallest deployable unit; wraps one (or a few tightly-coupled) containers sharing network/storage.
- **Deployment** — manages a set of identical Pods, handling **replicas, rolling updates, and rollbacks** to a desired state.
- **Service** — gives a stable network endpoint/IP and load-balances traffic across Pods (since Pods are ephemeral with changing IPs).
- **Ingress** — manages **external HTTP/HTTPS access** with routing rules (host/path-based) and TLS termination, typically via an ingress controller (e.g., NGINX, Application Gateway).

### Q90. How do you deploy to AKS from an Azure DevOps pipeline?
**Answer:**
🎯 Typical flow:
1. **Build** the Docker image and **push** it to a registry — usually **ACR (Azure Container Registry)** — tagged with the build ID.
2. Use a **Service Connection** to the Kubernetes cluster (or **Environment** with a Kubernetes resource).
3. **Deploy** using `kubectl apply` with manifests, or **Helm** charts (`HelmDeploy@0`), or the `KubernetesManifest@1` task (which supports canary/blue-green strategies).
4. Store secrets via Key Vault + CSI driver. Use **imagePullSecrets** or AKS-ACR integration for registry auth.

---

## 12. Security, Secrets & DevSecOps

### Q91. What is DevSecOps?
**Answer:**
**DevSecOps** integrates **security** into every stage of DevOps rather than treating it as a final gate. Security becomes a shared responsibility, automated into the pipeline (the "shift-left" of security). This includes automated vulnerability scanning, secrets detection, dependency checks, and compliance-as-code — catching issues early when they're cheap to fix.

### Q92. What is Azure Key Vault and how is it used in pipelines?
**Answer:**
**Azure Key Vault** is a managed service for securely storing and accessing **secrets** (passwords, connection strings), **keys** (encryption), and **certificates**. In pipelines you reference it by **linking a variable group to Key Vault** or using the `AzureKeyVault@2` task, so secrets are fetched at runtime and never stored in the YAML or repo. Access is controlled via Azure RBAC / access policies and audited.

### Q93. What are SAST, DAST, and SCA?
**Answer:**
Three complementary security scan types:
- **SAST (Static Application Security Testing)** — analyzes **source code** (without running it) for vulnerabilities like SQL injection patterns. Runs early (on commit/PR). Tools: SonarQube, Checkmarx.
- **DAST (Dynamic Application Security Testing)** — tests the **running application** from the outside, simulating attacks. Tools: OWASP ZAP.
- **SCA (Software Composition Analysis)** — scans **third-party/open-source dependencies** for known vulnerabilities (CVEs). Tools: Dependabot, Snyk, WhiteSource/Mend.

🎯 A mature pipeline runs all three.

### Q94. How do you prevent secrets from being committed to a repo?
**Answer:**
- Add sensitive files to **`.gitignore`** (e.g., `.env`).
- Use **secret-scanning tools** / pre-commit hooks (e.g., GitLeaks, git-secrets) and enable **push protection**.
- Store all real secrets in **Key Vault**, not in code.
- If a secret leaks, **rotate it immediately** (assume it's compromised) — removing it from history isn't enough.
- Educate the team and enforce PR reviews.

### Q95. What is the principle of least privilege and how do you apply it in Azure DevOps?
**Answer:**
**Least privilege** means granting every user, service, or pipeline only the **minimum permissions** required to do its job — nothing more. In Azure DevOps:
- Assign permissions to **groups**, not individuals.
- Scope **service connections** to specific subscriptions/resource groups.
- Use **scoped PATs** with short expiry.
- Limit who can edit pipelines, approve releases, and access secrets.
- Prefer **Workload Identity Federation** over standing credentials.

### Q96. What is a Service Principal vs. Managed Identity vs. Workload Identity Federation?
**Answer:**
- **Service Principal (SP)** — a non-human identity (app registration) in Azure AD with a **secret or certificate** used to authenticate automation to Azure. Secrets must be rotated.
- **Managed Identity** — an identity automatically managed by Azure for a resource (like a VM or App Service); **no credentials to manage** — Azure handles it. (System-assigned or user-assigned.)
- **Workload Identity Federation** — lets external systems (like Azure DevOps or GitHub Actions) authenticate to Azure using **short-lived federated tokens** instead of stored secrets — the most secure, "passwordless" option for pipelines.

### Q97. How do you implement compliance and auditing in Azure DevOps?
**Answer:**
🎯 Approaches:
- **Branch policies + required reviewers** enforce code review (segregation of duties).
- **Required templates (extends)** ensure security steps can't be skipped.
- **Approvals/checks** on environments create an audit trail of who authorized deployments.
- **Audit logs** in Azure DevOps record administrative actions.
- **Azure Policy** enforces governance on the infrastructure side (e.g., "no public storage accounts").
- Link work items to commits/PRs for full traceability from requirement to release.

---

## 13. Monitoring, Logging & Observability

### Q98. What is the difference between monitoring and observability?
**Answer:**
- **Monitoring** — tracking **known** metrics and alerting when they cross thresholds (e.g., CPU > 80%). It answers "is the system healthy by the measures we predefined?"
- **Observability** — the ability to **understand the internal state** of a system from its outputs, so you can investigate **unknown/unexpected** problems. It's built on the **three pillars: metrics, logs, and traces**.

🎯 Monitoring tells you *that* something is wrong; observability helps you find out *why*.

### Q99. What are Azure Monitor and Application Insights?
**Answer:**
- **Azure Monitor** — the umbrella platform that collects, analyzes, and acts on telemetry (metrics and logs) from Azure resources and applications. It powers alerts, dashboards, and autoscale.
- **Application Insights** — an **APM (Application Performance Monitoring)** feature of Azure Monitor focused on **applications**. It tracks request rates, response times, failure rates, dependencies, exceptions, and provides distributed tracing and live metrics — helping you find performance bottlenecks and errors in code.

### Q100. What is Log Analytics and KQL?
**Answer:**
**Log Analytics** is the tool/workspace in Azure Monitor where logs and metrics are stored and queried. **KQL (Kusto Query Language)** is the read-only query language used to analyze that data — e.g., filtering errors, aggregating request counts, joining tables. A simple KQL example:
```kql
requests
| where timestamp > ago(1h)
| where success == false
| summarize count() by name
```
🎯 KQL fluency is increasingly expected of Azure DevOps engineers.

### Q101. What are the "three pillars of observability"?
**Answer:**
1. **Metrics** — numeric measurements over time (CPU, request rate, latency). Cheap to store, good for dashboards/alerts.
2. **Logs** — timestamped, detailed event records (text/structured). Great for deep investigation of specific events.
3. **Traces** — the end-to-end path of a single request across multiple services (**distributed tracing**), showing where time is spent and where failures occur in a microservices architecture.

### Q102. What is MTTR and how do you reduce it?
**Answer:**
**MTTR (Mean Time To Recovery/Repair)** is the average time to restore service after an incident — a key reliability metric. Reduce it by:
- **Good observability** — find the root cause fast.
- **Automated alerts** that fire quickly and route to the right people.
- **Fast rollback mechanisms** (slot swaps, feature flags).
- **Runbooks** — documented response procedures.
- **Practiced incident response** and blameless postmortems to prevent recurrence.

---

## 14. Real-World Scenario & Troubleshooting

### Q103. 🎯 Scenario: A deployment to production failed halfway and the app is now broken. What do you do?
**Answer:**
1. **Stabilize first** — initiate **rollback** immediately (slot swap back, redeploy last good artifact, or disable the feature via flag) to restore service. Recovery beats root-causing in the moment.
2. **Communicate** — notify stakeholders and update the incident channel/status page.
3. **Investigate** — check pipeline logs, Application Insights, and recent changes to find what failed (bad migration? config? dependency?).
4. **Verify recovery** — confirm metrics/health are back to normal.
5. **Postmortem** — run a **blameless** review, identify the root cause, and add safeguards (better tests, gates, smoke tests, approvals) so it can't recur.

### Q104. 🎯 Scenario: Builds are taking 40 minutes and developers are frustrated. How do you speed them up?
**Answer:**
- **Caching** — cache dependencies (`node_modules`, NuGet, Maven) with `Cache@2`.
- **Parallelize** — split tests across multiple agents (matrix/slicing) and run independent jobs concurrently.
- **Self-hosted agents** with more CPU/RAM and warm caches if Microsoft-hosted is the bottleneck.
- **Incremental/path filters** — only build/test what changed (especially in monorepos).
- **Multi-stage Docker builds** and layer caching for container builds.
- **Remove redundant steps**, fail fast on quick checks (lint/unit) before slow ones.
- **Profile** the pipeline to find the actual slow steps before optimizing.

### Q105. 🎯 Scenario: A developer accidentally committed an API key to the repo. What's your response?
**Answer:**
1. **Treat it as compromised** — immediately **rotate/revoke** the key. Removing it from Git history is *not* enough because it may already be cloned/scraped.
2. **Remove from history** if required (e.g., `git filter-repo` or BFG) and force-push — coordinate with the team.
3. **Audit** for any unauthorized use of the key.
4. **Prevent recurrence** — add **secret scanning / push protection**, move the secret to **Key Vault**, update `.gitignore`, and educate the developer.

### Q106. 🎯 Scenario: How would you design a CI/CD pipeline for a microservices application?
**Answer:**
- **Independent pipelines per service** so services deploy independently (with **path filters** in a monorepo or separate repos).
- **Build** → containerize each service → push to **ACR** with immutable tags.
- **Automated tests** at multiple levels (unit, integration, contract tests between services).
- **Deploy to AKS** using Helm, with **canary or rolling** strategies per service.
- **Environments** (dev/test/prod) with approvals before prod.
- **Shared templates** to enforce consistent build/security steps across teams.
- **Observability** — centralized logging and distributed tracing across services.
- **Versioning & backward compatibility** so services can deploy without breaking consumers.

### Q107. 🎯 Scenario: The same code works in dev but fails in production. How do you troubleshoot?
**Answer:**
This usually points to **environment differences**. Check:
- **Configuration/variables** — different connection strings, feature flags, or missing env vars in prod.
- **Secrets** — Key Vault access or a missing/expired secret.
- **Infrastructure differences** — scaling, networking/firewall rules, permissions (RBAC), DNS.
- **Data differences** — prod data volume/edge cases not present in dev.
- **Configuration drift** — manual changes made to prod.
- **Dependencies/versions** — different runtime or package versions.

🎯 The fix long-term: use **IaC** and the **same artifact** across environments to keep them identical, and externalize all config.

### Q108. 🎯 Scenario: Your team wants zero-downtime deployments. How do you achieve it?
**Answer:**
- Use **blue-green** (slot swap) or **rolling/canary** deployment strategies so old instances serve traffic while new ones come up.
- Ensure **health probes/readiness checks** so traffic only routes to healthy instances.
- Make **database changes backward-compatible** (expand/contract) so both versions coexist during rollout.
- Use **load balancer draining** to finish in-flight requests before removing old instances.
- **Feature flags** to decouple deploy from release.
- Automate **smoke tests** post-deploy and auto-rollback on failure.

### Q109. 🎯 Scenario: How do you manage configuration across dev, test, and production environments?
**Answer:**
- **Never hardcode** environment-specific values in code.
- Use **variable groups** per environment and/or **environment-specific config files** transformed at deploy time.
- Store **secrets in Key Vault**, referenced per environment.
- Keep infrastructure in **IaC** with per-environment parameter files.
- Use **the same build artifact** everywhere — only inject configuration at deploy time ("build once, configure per environment").
- Control access so only authorized people can view/edit production config.

### Q110. 🎯 Scenario: Management asks you to prove your DevOps practices are improving. What metrics do you present?
**Answer:**
Present the **DORA (DevOps Research and Assessment) metrics**, which correlate with high performance:
1. **Deployment Frequency** — how often you ship to production (higher is better).
2. **Lead Time for Changes** — time from commit to running in production (lower is better).
3. **Change Failure Rate** — percentage of deployments causing a failure (lower is better).
4. **MTTR (Mean Time To Recovery)** — how fast you recover from incidents (lower is better).

🎯 Supplement with build success rate, test coverage trends, cycle time, and mean PR review time. Show **trends over time**, not just snapshots, to demonstrate improvement.

---

## ✅ Final Tips for the Interview
- **Explain the "why," not just the "what."** Interviewers want reasoning, not memorized definitions.
- **Bring real examples.** Tie answers to things you've built or fixed; the 🎯 scenarios above are great patterns to adapt.
- **Always mention security and automation.** Modern Azure DevOps roles expect DevSecOps awareness.
- **Know the trade-offs.** YAML vs. Classic, Bicep vs. Terraform, Git Flow vs. Trunk-Based — be ready to justify a choice.
- **Be honest about gaps,** then explain how you'd find the answer (docs, experimentation, asking the team).

*Good luck — you've got this! 🚀*
