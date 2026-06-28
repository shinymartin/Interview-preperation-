# Windows Server Administration — Interview Questions & Answers

> A complete, interview-ready study guide covering basic to advanced Windows Server Administration concepts, with practical real-world scenarios. Cloud-provider-specific topics (AWS/Azure) are intentionally excluded — only general, on-premises and platform-agnostic concepts are covered.

---

## 📊 Total Questions & Answers: **110**

### Breakdown by Sub-Topic
| # | Sub-Topic | Questions |
|---|-----------|-----------|
| 1 | Windows Server Fundamentals | Q1 – Q8 (8) |
| 2 | Active Directory (AD) | Q9 – Q30 (22) |
| 3 | DNS (Domain Name System) | Q31 – Q39 (9) |
| 4 | DHCP (Dynamic Host Configuration Protocol) | Q40 – Q47 (8) |
| 5 | Group Policy (GPO) | Q48 – Q56 (9) |
| 6 | File & Storage Services | Q57 – Q69 (13) |
| 7 | Networking Essentials | Q70 – Q77 (8) |
| 8 | Hyper-V & Virtualization | Q78 – Q85 (8) |
| 9 | Backup & Disaster Recovery | Q86 – Q91 (6) |
| 10 | Security & Hardening | Q92 – Q99 (8) |
| 11 | PowerShell & Administration | Q100 – Q103 (4) |
| 12 | Patch Management & Monitoring | Q104 – Q107 (4) |
| 13 | Real-World Troubleshooting Scenarios | Q108 – Q110 (3) |

---

# 1. Windows Server Fundamentals

### Q1. What is Windows Server, and how does it differ from a desktop version of Windows?
**Answer:**
Windows Server is a server-class operating system (OS) built by Microsoft to provide centralized services to many users and computers on a network — things like authentication, file sharing, email back-ends, websites, and databases.

The main differences from a desktop OS (like Windows 11) are:
- **Roles & Features:** Windows Server can host server **roles** such as Active Directory, DNS, DHCP, IIS, and Hyper-V. Desktop OSes cannot.
- **Scalability:** Server supports far more CPU sockets, cores, and RAM (Random Access Memory). A desktop OS is capped much lower.
- **Concurrent connections:** Server is licensed and designed for many simultaneous users/connections; desktop OSes are limited (e.g., ~20 inbound SMB connections).
- **Stability & uptime:** Server is tuned for reliability and to run 24/7 with minimal reboots.
- **No consumer apps:** Server omits consumer features (Cortana, Microsoft Store apps, etc.) to reduce the **attack surface** (the number of ways a system can be compromised).

**Real-world framing:** A desktop OS serves *one person*. A server OS serves *the whole organization*.

---

### Q2. What are the main editions of Windows Server, and when would you choose each?
**Answer:**
The two primary editions in modern Windows Server (2016/2019/2022/2025) are:
- **Standard Edition:** For physical or lightly virtualized environments. Its licensing includes the rights to run **two** Operating System Environments / virtual machines (VMs) per license set.
- **Datacenter Edition:** For highly virtualized and software-defined datacenters. It allows **unlimited** VMs on the licensed host and unlocks advanced features like **Storage Spaces Direct (S2D)**, **Software-Defined Networking (SDN)**, and **Shielded VMs**.

There is also **Essentials Edition** — a low-cost, single-server option for very small businesses (typically up to 25 users / 50 devices) with simplified setup.

**Choosing guidance:** If you'll run many VMs on a host, Datacenter is usually cheaper and more capable. If you run one or two VMs, Standard is more economical.

---

### Q3. What are the installation options for Windows Server (Desktop Experience vs Server Core vs Nano Server)?
**Answer:**
- **Desktop Experience (GUI):** Full graphical user interface (GUI) with the familiar desktop. Easiest to manage manually, but larger footprint, more updates, and a bigger attack surface.
- **Server Core:** No desktop GUI — just a command line (PowerShell/cmd). Smaller footprint, fewer patches, less attack surface, and lower resource use. Managed remotely via PowerShell, Server Manager, **RSAT** (Remote Server Administration Tools), or **Windows Admin Center**. Microsoft recommends Server Core for most production roles.
- **Nano Server:** An ultra-minimal image meant only for **containers** (lightweight, isolated app packages). It is not a general-purpose install option.

**Real-world tip:** Use Server Core for infrastructure roles (DNS, DHCP, AD, Hyper-V) to harden security and reduce reboots; use Desktop Experience when an app explicitly requires a GUI.

---

### Q4. What is the difference between a Workgroup and a Domain?
**Answer:**
- **Workgroup:** A *peer-to-peer* model. Each computer manages its own local accounts and security. There is no central authority. Good for very small networks (a handful of machines) but does not scale — you'd have to create accounts on every machine separately.
- **Domain:** A *client-server* model managed centrally by **Active Directory** running on **Domain Controllers**. User accounts, computers, and policies are managed in one place. A user can log into any domain-joined machine with a single account (**Single Sign-On**, SSO), and admins can push settings to everyone using Group Policy.

**Real-world framing:** Workgroup = "everyone keeps their own keys." Domain = "one central security office issues and controls all keys."

---

### Q5. What are Roles and Features in Windows Server?
**Answer:**
- A **Role** is a major function that defines the server's primary purpose — for example, **AD DS** (Active Directory Domain Services), **DNS Server**, **DHCP Server**, **Web Server (IIS)**, **Hyper-V**, or **File and Storage Services**.
- A **Feature** is a supporting component that augments a role or the OS — for example, **.NET Framework**, **Failover Clustering**, **BitLocker**, or **Windows Server Backup**.

You install both through **Server Manager → Add Roles and Features**, or via PowerShell with `Install-WindowsFeature`.

**Example:** To stand up a domain controller, you install the *AD DS role* (which often pulls in *DNS*), then promote the server.

---

### Q6. Explain the core-based licensing model in Windows Server.
**Answer:**
Modern Windows Server (since 2016) uses **core-based licensing** instead of the older per-processor model. The key rules:
- You must license **all physical cores** in the server.
- There is a **minimum of 8 core licenses per physical processor** and a **minimum of 16 cores per server**.
- Core licenses are typically sold in **2-core packs**.
- Separately, **CALs** (Client Access Licenses) are required for users or devices that access the server. There are **User CALs** (one per person, any device) and **Device CALs** (one per device, any number of people).

**Real-world note:** Don't forget CALs — they're a common audit gap. Specialized roles like Remote Desktop Services (RDS) require their *own* RDS CALs on top.

---

### Q7. What is Server Manager, and what can you do with it?
**Answer:**
**Server Manager** is the central management console (dashboard) in the Desktop Experience installation. From it you can:
- Add or remove **roles and features**.
- Manage **multiple remote servers** from one console (server groups).
- View **health, events, services, and performance** alerts.
- Launch management tools (DNS Manager, AD Users and Computers, etc.).

For managing many servers or modern/Core deployments, admins increasingly use **Windows Admin Center (WAC)** — a browser-based tool — or **PowerShell** for automation.

---

### Q8. What is the typical boot process / startup sequence of a Windows Server?
**Answer:**
At a high level:
1. **POST (Power-On Self-Test):** Firmware (BIOS/UEFI) checks hardware.
2. **Boot loader:** UEFI/BIOS loads the **Windows Boot Manager** (`bootmgr`), which reads the **BCD** (Boot Configuration Data) store.
3. **Kernel load:** `winload.efi` loads the Windows kernel (`ntoskrnl.exe`) and **HAL** (Hardware Abstraction Layer), then loads boot-critical drivers.
4. **Session Manager (smss.exe):** Initializes the system session and starts core subsystems.
5. **Services & logon:** The Service Control Manager starts services; **winlogon** presents the logon screen.

**Why it matters in interviews:** Knowing this helps you troubleshoot boot failures — e.g., a corrupt **BCD** (fix with `bcdedit`/`bootrec`), a bad boot driver (boot into Safe Mode), or hardware failures at POST.

---

# 2. Active Directory (AD)

### Q9. What is Active Directory (AD), and why is it important?
**Answer:**
**Active Directory (AD)** is Microsoft's **directory service** — a centralized database that stores information about network objects (users, computers, groups, printers, etc.) and provides **authentication** (verifying *who you are*) and **authorization** (deciding *what you can access*).

It's the backbone of most enterprise Windows networks because it enables:
- **Single Sign-On (SSO):** one account works everywhere.
- **Centralized management:** of users, passwords, and permissions.
- **Group Policy:** to enforce configuration and security org-wide.

The role that provides it is **AD DS** (Active Directory Domain Services).

---

### Q10. Explain the difference between a Domain, a Tree, and a Forest.
**Answer:**
These are the logical building blocks of AD, from smallest to largest:
- **Domain:** A single administrative and security boundary that shares one AD database (e.g., `corp.example.com`). All objects in it share a common namespace and policies.
- **Tree:** One or more domains that share a **contiguous namespace** (a parent-child DNS hierarchy), e.g., `example.com`, `sales.example.com`, `eu.sales.example.com`. Domains in a tree are connected by automatic two-way trusts.
- **Forest:** The **top-level container** and the ultimate **security boundary** of AD. It holds one or more trees that may have **different (non-contiguous) namespaces** (e.g., `example.com` and `contoso.com`) but share a common **schema** (the definition of object types) and **Global Catalog**.

**Analogy:** Domain = a building, Tree = a campus of related buildings, Forest = the entire organization that owns all campuses.

---

### Q11. What is a Domain Controller (DC)?
**Answer:**
A **Domain Controller (DC)** is a server running the **AD DS** role that holds a copy of the AD database (`NTDS.dit`) and handles **authentication and authorization** requests for the domain — for example, validating logons via **Kerberos**.

Key points:
- Most DCs are **multi-master** (writable) — changes can be made on any DC and **replicate** to the others.
- Best practice is **at least two DCs per domain** for redundancy. If your only DC dies, no one can log in or access domain resources.

---

### Q12. What is the difference between AD DS and AD LDS?
**Answer:**
- **AD DS (Active Directory Domain Services):** The full directory service that provides domain functionality — authentication, Kerberos, Group Policy, domain join, etc. Requires a domain.
- **AD LDS (Active Directory Lightweight Directory Services):** A trimmed-down **LDAP** (Lightweight Directory Access Protocol) directory that stores application-specific data **without** requiring a domain or domain controllers. It's used by apps that need a directory store but don't need the full domain infrastructure.

**Use case:** AD LDS is handy for a custom application that needs its own directory of users/objects, kept separate from the corporate AD.

---

### Q13. What are FSMO roles? Name and explain them.
**Answer:**
**FSMO** stands for **Flexible Single Master Operations**. While AD is multi-master, certain operations can't safely happen on multiple DCs at once, so they're assigned to **single-master** role holders. There are **five** FSMO roles:

**Forest-wide (one per forest):**
1. **Schema Master:** The only DC that can make changes to the AD **schema** (object/attribute definitions). Needed for actions like extending the schema for Exchange.
2. **Domain Naming Master:** Controls adding/removing **domains** in the forest.

**Domain-wide (one per domain):**
3. **PDC Emulator (Primary Domain Controller Emulator):** Acts as the authoritative **time source**, handles **password changes** priority, **account lockout** processing, and is the default target for Group Policy edits. Often considered the most critical role.
4. **RID Master (Relative Identifier Master):** Hands out pools of **RIDs** to other DCs so every new security object gets a unique **SID** (Security Identifier).
5. **Infrastructure Master:** Keeps **cross-domain object references** up to date (e.g., group memberships referencing users in other domains).

**Interview tip:** Know that in a single-domain forest, the Infrastructure Master concern about Global Catalog placement largely goes away. You can **transfer** roles gracefully or **seize** them if a DC is permanently dead (using `ntdsutil`).

---

### Q14. What is the Global Catalog (GC)?
**Answer:**
The **Global Catalog (GC)** is a special distributed data store hosted on designated DCs. It holds a **full copy of all objects in its own domain** plus a **partial, read-only copy of every object in every other domain** in the forest (just the most commonly searched attributes).

Its purposes:
- **Forest-wide searches:** Find objects anywhere in the forest quickly.
- **Logon support:** It resolves **Universal Group** memberships during logon.
- **UPN resolution:** Resolves **User Principal Name** logons (`user@domain.com`).

**Real-world note:** If the GC is unavailable, users may fail to log on (especially with Universal Groups) — so place GC servers in every site for resilience.

---

### Q15. What are Organizational Units (OUs), and how do they differ from containers?
**Answer:**
An **Organizational Unit (OU)** is a **container object** in AD used to logically organize users, groups, and computers — usually by department, location, or function (e.g., an "HR" OU, a "Servers" OU).

Why OUs matter:
- You can **link Group Policy Objects (GPOs)** to OUs to apply settings to everything inside them.
- You can **delegate administration** — give specific admins control over just one OU (e.g., a helpdesk team manages only the "Sales" OU).

**OU vs default container:** A default **container** (like the built-in `Users` or `Computers`) **cannot** have a GPO linked to it, while an **OU can**. That's a key reason to create OUs and move objects out of the default containers.

---

### Q16. What is the difference between a Security Group and a Distribution Group?
**Answer:**
- **Security Group:** Can be assigned **permissions** to resources (files, folders, printers) and is used for access control. It has a **SID**. It can also be email-enabled. This is the workhorse for granting access.
- **Distribution Group:** Used **only** for email distribution lists (e.g., in Exchange). It has **no SID** and **cannot** be used to assign permissions.

**Rule of thumb:** If you need to grant access, use a **security group**. If you only need to email a set of people, a distribution group is enough.

---

### Q17. Explain AD group scopes (Domain Local, Global, Universal) and the AGDLP strategy.
**Answer:**
Group **scope** defines where a group can pull members from and where it can be used:
- **Global group:** Members come from the **same domain**; can be used to grant access **anywhere in the forest**. Used to group users by role (e.g., "Sales Users").
- **Domain Local group:** Members from **any domain**; used to grant access to resources in **its own domain only**. Used to control access to a specific resource.
- **Universal group:** Members from **any domain**; usable **anywhere in the forest**. Memberships are stored in the Global Catalog (use carefully to limit replication).

**AGDLP** is Microsoft's best-practice nesting strategy:
**A**ccounts → **G**lobal groups → **D**omain **L**ocal groups → **P**ermissions.
You put **Accounts into Global groups** (by role), put **Global groups into Domain Local groups** (by resource), and assign **Permissions to the Domain Local group**. This keeps permissions clean, scalable, and easy to audit.

---

### Q18. How does Active Directory replication work?
**Answer:**
**Replication** keeps the AD database consistent across all DCs (multi-master model). Key concepts:
- **Within a site (intra-site):** Replication is **fast and uncompressed**, triggered shortly after a change (notification-based), assuming high-speed LAN links.
- **Between sites (inter-site):** Replication is **scheduled and compressed** to conserve bandwidth on slower WAN links, using **site links** and **bridgehead servers**.
- **KCC (Knowledge Consistency Checker):** An automatic process that builds and maintains the **replication topology** (the connections between DCs).
- **USN (Update Sequence Number)** and **high-watermark vectors** track which changes a DC already has, so only deltas replicate.

**Tools:** `repadmin /replsummary` and `repadmin /showrepl` are commonly used to check replication health. Knowing these commands is a strong interview signal.

---

### Q19. What is SYSVOL, and why is it important?
**Answer:**
**SYSVOL** is a shared folder on every DC (default path `C:\Windows\SYSVOL`) that stores domain-wide public files that must be **available and identical on all DCs**. It primarily holds:
- **Group Policy** files (the GPO templates/settings in the "Policies" folder).
- **Logon/logoff and startup/shutdown scripts**.

SYSVOL is replicated between DCs using **DFS-R** (Distributed File System Replication) in modern domains (older domains used **FRS**, File Replication Service, now deprecated). If SYSVOL replication breaks, Group Policy may apply inconsistently across the domain.

---

### Q20. What is a Read-Only Domain Controller (RODC)?
**Answer:**
A **Read-Only Domain Controller (RODC)** is a DC that holds a **read-only** copy of the AD database. Changes cannot be made directly on it; it pulls updates from a writable DC.

Why it exists:
- Designed for **branch offices** or **less physically secure locations** where you can't guarantee the server's safety.
- By default it does **not cache passwords** (controlled by a **Password Replication Policy**), so if it's stolen, the exposure is limited.
- It reduces risk because malware or a malicious change made locally can't replicate back into the domain.

---

### Q21. What is the difference between authentication and authorization?
**Answer:**
- **Authentication** answers **"Who are you?"** — it verifies identity, typically with a username and password (and possibly MFA). In AD this is handled by **Kerberos** or **NTLM**.
- **Authorization** answers **"What are you allowed to do?"** — it decides which resources you can access and what actions you can take, based on permissions (e.g., NTFS ACLs) and group memberships.

**Analogy:** Authentication is showing your ID at the door; authorization is the list of rooms your badge can open once you're inside.

---

### Q22. Compare Kerberos and NTLM authentication.
**Answer:**
- **Kerberos** is the **default and preferred** authentication protocol in modern AD. It uses a **ticket-based** system and a trusted third party — the **KDC** (Key Distribution Center, which runs on DCs). The user gets a **TGT** (Ticket-Granting Ticket) at logon, then requests **service tickets** to access resources. It supports **mutual authentication** (both sides verify each other) and avoids sending passwords over the network. It is **time-sensitive** — clocks must be within ~5 minutes (which is why the PDC Emulator's time role matters).
- **NTLM (NT LAN Manager)** is the older **challenge-response** protocol. It's used as a **fallback** — for example, when accessing a resource by IP address instead of name, in workgroups, or when Kerberos isn't possible. It is weaker and a common target for attacks (e.g., pass-the-hash), so organizations work to minimize its use.

**Interview gold:** Mentioning that Kerberos failures are often **clock skew** or **SPN (Service Principal Name)** issues shows real depth.

---

### Q23. What is a trust relationship in Active Directory? Name the main types.
**Answer:**
A **trust** lets users in one domain or forest access resources in another. Types include:
- **Two-way vs One-way:** Two-way means both domains trust each other; one-way means trust flows in a single direction.
- **Transitive vs Non-transitive:** Transitive trust extends through the chain (if A trusts B and B trusts C, then A trusts C); non-transitive does not.
- **Parent-child & Tree-root trusts:** Created **automatically** when you add domains within a forest (two-way, transitive).
- **External trust:** Non-transitive trust to a domain in **another forest** or an older NT domain.
- **Forest trust:** Transitive trust between **two entire forests** (common after mergers/acquisitions).
- **Shortcut trust:** Optimizes authentication paths between domains in a complex forest.

---

### Q24. How do you transfer vs seize FSMO roles, and when would you do each?
**Answer:**
- **Transfer (graceful):** Used when the current role holder is **online and healthy** — for example, during planned maintenance or decommissioning. Both DCs coordinate the handoff. You can transfer via the AD GUI consoles, `ntdsutil`, or PowerShell `Move-ADDirectoryServerOperationMasterRole`.
- **Seize (forceful):** Used **only** when the role holder is **permanently offline/dead** and won't come back. You forcibly assign the role to another DC using `ntdsutil` (or PowerShell with `-Force`).

**Critical caution:** After seizing a role, the **old DC must never come back online** — doing so can cause AD corruption/conflicts. It should be cleanly removed (metadata cleanup) before any rebuild.

---

### Q25. What are AD Sites and Site Links, and why do they matter?
**Answer:**
**Sites** in **Active Directory Sites and Services** represent **physical network locations** (defined by IP subnets), typically corresponding to offices connected by WAN links.

They matter because:
- **Replication efficiency:** Inter-site replication is scheduled and compressed to save WAN bandwidth (via **site links**).
- **Client locality:** Clients prefer DCs, DFS targets, and other services **in their own site**, reducing latency and WAN traffic. For example, a user authenticates against a local DC rather than one across the country.

**Real-world example:** A company with offices in New York and London defines two sites; site links control how often AD changes replicate across the expensive transatlantic link.

---

### Q26. Walk through what happens when a user logs into a domain-joined computer.
**Answer:**
A simplified flow:
1. The user enters credentials at the **winlogon** screen.
2. The computer locates a DC using **DNS** (specifically `SRV` records for the domain) and **site** awareness to find a *local* DC.
3. The DC's **KDC** authenticates the user via **Kerberos**, issuing a **TGT** (Ticket-Granting Ticket).
4. The user's **group memberships and SIDs** are gathered into an **access token**.
5. **Group Policy** is processed and applied (computer policies at startup, user policies at logon).
6. The user's profile loads, logon scripts run, and the desktop appears.

**Troubleshooting hooks:** Logon problems often trace back to **DNS misconfiguration**, **DC unavailability**, **time skew (Kerberos)**, or **GPO processing errors**.

---

### Q27. What is LDAP, and how does it relate to Active Directory?
**Answer:**
**LDAP (Lightweight Directory Access Protocol)** is the **protocol** used to **query and modify** directory services. Active Directory uses LDAP as the way clients and applications **read and write** directory data.

Key points:
- Default ports: **389** for LDAP, **636** for **LDAPS** (LDAP over SSL/TLS, encrypted). Global Catalog uses **3268** (and **3269** for SSL).
- Objects are referenced by a **Distinguished Name (DN)**, e.g., `CN=John Smith,OU=Sales,DC=corp,DC=example,DC=com`, where **CN** = Common Name, **OU** = Organizational Unit, **DC** = Domain Component.

**Security note:** Microsoft has pushed for **LDAP signing/channel binding** (and LDAPS) to prevent unencrypted directory traffic and relay attacks.

---

### Q28. What are domain and forest functional levels?
**Answer:**
**Functional levels** determine which **AD features** are available and set the **minimum Windows Server version** allowed for DCs.
- **Domain Functional Level (DFL):** Applies to one domain; all its DCs must run at least that OS version.
- **Forest Functional Level (FFL):** Applies forest-wide; it can't be higher than the lowest DFL.

**Example:** Raising the level to Windows Server 2016 unlocks certain security features but means you can no longer add a 2012 R2 DC. Raising functional levels is generally **one-way** (you can't easily roll back), so plan carefully.

---

### Q29. What is the AD Recycle Bin, and how is it different from tombstones?
**Answer:**
- **Tombstone:** When an object is deleted, AD doesn't immediately purge it; it becomes a **tombstone** (most attributes stripped) for a set **tombstone lifetime** (default 180 days) so the deletion replicates everywhere. Restoring from a tombstone alone loses most attributes.
- **AD Recycle Bin:** A feature (introduced in 2008 R2) that, once enabled, lets you **restore deleted objects with all their attributes and group memberships intact**, without an authoritative restore from backup. Restores can be done via the **AD Administrative Center** GUI or PowerShell (`Restore-ADObject`).

**Note:** Enabling the AD Recycle Bin is **irreversible** but highly recommended.

---

### Q30. What is the `NTDS.dit` file, and where are its supporting files?
**Answer:**
`NTDS.dit` is the **actual Active Directory database file** — it stores all directory objects (users, groups, computers, etc.). It typically lives in `C:\Windows\NTDS`.

Supporting files include:
- **Transaction logs** (`edb.log`) — like a database journal, ensuring changes are durable.
- **Checkpoint file** (`edb.chk`) — tracks which log entries are committed.

**Why it matters:** Protecting and backing up `NTDS.dit` and SYSVOL is core to AD disaster recovery. The **DSRM (Directory Services Restore Mode)** password — set during DC promotion — is needed to perform authoritative restores and offline maintenance, so it must be documented.

---

# 3. DNS (Domain Name System)

### Q31. What is DNS, and why is it critical to Active Directory?
**Answer:**
**DNS (Domain Name System)** translates human-friendly names (like `server01.corp.example.com`) into **IP addresses** (like `10.0.0.5`) and back. It's basically the **phonebook of the network**.

It's **critical to AD** because clients use special DNS **`SRV` (Service) records** to **locate domain controllers** and services (LDAP, Kerberos, Global Catalog). If DNS is broken or misconfigured, AD logons, replication, and Group Policy all fail. The saying among admins is: *"It's always DNS."*

---

### Q32. What is the difference between a Forward Lookup Zone and a Reverse Lookup Zone?
**Answer:**
- **Forward Lookup Zone:** Resolves a **name → IP address** (the most common direction). Example: `web01.corp.com` → `10.0.0.20`.
- **Reverse Lookup Zone:** Resolves an **IP address → name**, using special **`PTR` (Pointer)** records and the `in-addr.arpa` namespace. Example: `10.0.0.20` → `web01.corp.com`.

**Why reverse zones matter:** They're needed for things like **email anti-spam checks**, **logging/auditing** (showing names instead of raw IPs), and some applications that perform reverse lookups for verification.

---

### Q33. Explain common DNS record types.
**Answer:**
| Record | Full Name | Purpose |
|--------|-----------|---------|
| **A** | Address | Maps a hostname to an **IPv4** address |
| **AAAA** | Quad-A | Maps a hostname to an **IPv6** address |
| **CNAME** | Canonical Name | An **alias** pointing one name to another name |
| **PTR** | Pointer | Reverse lookup — maps an **IP to a name** |
| **MX** | Mail Exchanger | Identifies the **mail server** for a domain |
| **NS** | Name Server | Lists the **authoritative DNS servers** for a zone |
| **SOA** | Start of Authority | Holds zone metadata (primary server, serial number, refresh timers) |
| **SRV** | Service | Locates servers for specific **services** (used heavily by AD for DCs, Kerberos, LDAP) |
| **TXT** | Text | Holds text data, often for **SPF/DKIM** email validation or domain verification |

---

### Q34. What is the difference between a Forwarder and a Conditional Forwarder?
**Answer:**
- **Forwarder:** A DNS server you designate to handle **all** external/unknown queries that your server can't resolve locally. Commonly, internal DNS servers forward to the ISP's DNS or a public resolver (e.g., for general internet lookups).
- **Conditional Forwarder:** Forwards queries for a **specific domain** to a **specific DNS server**. For example, all queries for `partner.com` go to the partner company's DNS server, while everything else uses the normal forwarder.

**Use case:** Conditional forwarders are common between **trusted forests/partner domains** so each side can resolve the other's names without hosting full zones.

---

### Q35. What are Primary, Secondary, and Stub zones?
**Answer:**
- **Primary zone:** The **master, writable** copy of a zone. Changes are made here.
- **Secondary zone:** A **read-only copy** of a primary zone, kept in sync via **zone transfer**. It provides redundancy and load distribution. You can't edit records directly on it.
- **Stub zone:** A small zone containing only the **NS, SOA, and A records** needed to identify the **authoritative name servers** for a domain — not the full record set. It's used to keep a parent aware of a child zone's name servers efficiently.

---

### Q36. What is an Active Directory-Integrated Zone, and what are its advantages?
**Answer:**
An **AD-Integrated zone** stores its DNS data **inside the Active Directory database** rather than in a flat text file. It's only possible on DCs that also run DNS.

Advantages:
- **Multi-master updates:** Any DC running DNS can accept changes; they **replicate via AD replication** (secure and efficient) instead of separate zone transfers.
- **Secure dynamic updates:** Only authenticated, authorized computers can register/update their records, preventing rogue entries.
- **Built-in redundancy & fault tolerance:** No single primary server to fail.

This is the recommended setup for AD environments.

---

### Q37. What is DNS scavenging (aging), and why is it useful?
**Answer:**
**Scavenging** automatically **removes stale (outdated) dynamic DNS records** — for example, records left behind by devices that were removed or that changed IP addresses.

How it works: each dynamically registered record gets a **timestamp**; aging settings (no-refresh and refresh intervals) define how long a record can live without being refreshed. When scavenging runs, records older than the threshold are deleted.

**Why it matters:** Without scavenging, DNS fills up with **stale records**, causing clients to connect to wrong/old IPs and creating troubleshooting headaches. Enabling it (carefully, with sensible intervals) keeps DNS clean.

---

### Q38. Explain the difference between recursive and iterative DNS queries.
**Answer:**
- **Recursive query:** The client asks the DNS server for an answer and expects the server to **do all the work** and return a final answer (or an error). The client says, "Give me the full answer." This is what a typical PC does when it queries its configured DNS server.
- **Iterative query:** The DNS server queries other DNS servers, each returning the **best referral** it has ("I don't know, but ask this server"), and the resolver follows the chain. This is typically how DNS servers talk to each other up the hierarchy (root → TLD → authoritative).

**Combined flow:** Client makes a *recursive* request to its local DNS server; that server then makes *iterative* requests across the DNS hierarchy to find the answer.

---

### Q39. What are Root Hints, and how do they relate to forwarders?
**Answer:**
**Root Hints** are a built-in list of the **root DNS servers** (the top of the global DNS hierarchy). When a DNS server can't resolve a query locally and has **no forwarder configured**, it uses root hints to start the resolution process from the top down.

Relationship to forwarders:
- If a **forwarder** is configured, the server sends unresolved queries there first.
- If forwarders fail (or none are set), the server can fall back to **root hints** to resolve recursively itself.

**Real-world tip:** In secure/isolated networks, root hints may be removed and only forwarders used, to fully control where external queries go.

---

# 4. DHCP (Dynamic Host Configuration Protocol)

### Q40. What is DHCP, and explain the DORA process.
**Answer:**
**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP configuration (IP address, subnet mask, default gateway, DNS servers, etc.) to devices, removing the need for manual configuration.

The lease is obtained through the **DORA** process:
1. **Discover:** The client broadcasts a request looking for a DHCP server ("Is there a DHCP server out there?").
2. **Offer:** DHCP server(s) respond, offering an available IP address.
3. **Request:** The client broadcasts acceptance of one offer (formally requesting that IP).
4. **Acknowledge (ACK):** The server confirms and finalizes the lease, sending the full configuration.

**Memory aid:** **D-O-R-A** = Discover, Offer, Request, Acknowledge.

---

### Q41. What is a DHCP Scope, and what's the difference between Reservations and Exclusions?
**Answer:**
- **Scope:** A defined **range of IP addresses** that a DHCP server can lease out for a particular subnet (e.g., `10.0.0.100`–`10.0.0.200`), along with options like gateway and DNS.
- **Reservation:** Ties a **specific IP address to a specific device's MAC address** (Media Access Control — the hardware address). The device always gets the **same IP** via DHCP. Used for printers, servers, or devices that need consistent addressing but still managed centrally.
- **Exclusion:** A range of addresses **within the scope** that the server will **never lease** (e.g., reserving `.100`–`.110` for statically assigned servers).

**Difference in one line:** A reservation **always assigns** a set IP to one device; an exclusion **prevents** certain IPs from being handed out at all.

---

### Q42. What is a DHCP lease, and how does renewal work?
**Answer:**
A **lease** is the period of time a client is allowed to use an assigned IP address (e.g., 8 days by default for wired clients).

Renewal timers:
- At **50% (T1)** of the lease, the client tries to **renew** directly with the original DHCP server (unicast).
- At **87.5% (T2)**, if renewal failed, the client tries to renew from **any** DHCP server (broadcast/rebind).
- If the lease fully expires without renewal, the client must start the **DORA** process again.

**Tuning tip:** Short leases suit highly mobile/guest networks (addresses recycle fast); longer leases suit stable office networks (less DHCP traffic).

---

### Q43. What is a DHCP Relay Agent (IP Helper), and when do you need it?
**Answer:**
DHCP **Discover** messages are **broadcasts**, and routers normally **don't forward broadcasts** between subnets. So a DHCP server on one subnet won't directly hear clients on another subnet.

A **DHCP Relay Agent** (often configured on a router as an **IP Helper Address**) forwards those DHCP broadcasts to a DHCP server on another subnet (as unicast), so one central DHCP server can serve **multiple subnets/VLANs**.

**When you need it:** Any time clients and the DHCP server are on **different subnets** and you don't want a DHCP server in every subnet.

---

### Q44. Explain DHCP Failover modes (Load Balance vs Hot Standby).
**Answer:**
DHCP **Failover** (built into Windows Server 2012+) lets two DHCP servers share scope information for redundancy. Two modes:
- **Load Balance:** Both servers are **active** and split the client load (e.g., 50/50). If one fails, the other handles all requests. Good for distributing load.
- **Hot Standby:** One server is **active (primary)** handling all leases; the other is a **standby** that takes over only if the primary fails. Good for branch-to-central designs.

**Benefit:** Either way, clients keep getting IP addresses even if one DHCP server goes down — no single point of failure.

---

### Q45. What is the difference between Server Options, Scope Options, and Reservation Options?
**Answer:**
DHCP options (like DNS servers, gateway, domain name) can be set at different levels, applied in a **hierarchy** where the more specific level wins:
- **Server Options:** Apply to **all scopes** on the server (broadest).
- **Scope Options:** Apply to **one specific scope** and override server options for that scope.
- **Reservation Options:** Apply to a **single reserved device** and override scope/server options.

**Example:** You might set a default gateway at the scope level (since it differs per subnet) but DNS servers at the server level (since they're the same network-wide).

---

### Q46. What is a DHCP Superscope, and when is it used?
**Answer:**
A **Superscope** groups **multiple scopes** together so a single physical network segment can serve clients from **more than one IP subnet** (a configuration called **multinetting**).

**Use case:** When a subnet is **running out of addresses**, you can add a second IP range and bind both scopes into a superscope so the DHCP server hands out addresses from either range on the same physical LAN, without re-IP-ing the whole network immediately.

---

### Q47. What is APIPA, and what does it indicate?
**Answer:**
**APIPA (Automatic Private IP Addressing)** is a fallback feature where a Windows client that is set to obtain an IP automatically but **can't reach a DHCP server** assigns itself an address in the range **169.254.0.0 – 169.254.255.255**.

**What it tells you:** If a machine has a `169.254.x.x` address, it means **DHCP failed** — possible causes include a down DHCP server, an exhausted scope, a broken relay agent, a disconnected/faulty network cable, or a VLAN/switch issue. It's a quick visual diagnostic that "the client never got a real lease."

---

# 5. Group Policy (GPO)

### Q48. What is Group Policy, and what is a GPO?
**Answer:**
**Group Policy** is a Windows feature that lets administrators **centrally manage and enforce configuration** for users and computers in a domain. A **GPO (Group Policy Object)** is the container holding those settings.

Examples of what you can control:
- Password and account lockout policies
- Desktop settings, drive mappings, and printers
- Software installation
- Security settings (firewall, audit policies)
- Startup/shutdown and logon/logoff scripts

GPOs are linked to **sites, domains, or OUs**, and they apply to the users/computers within that scope.

---

### Q49. Explain the order in which Group Policies are applied (LSDOU).
**Answer:**
GPOs apply in a specific order, summarized as **LSDOU**:
1. **L** – **Local** group policy (on the machine itself)
2. **S** – **Site** level
3. **D** – **Domain** level
4. **OU** – **Organizational Unit** level (and nested OUs, parent before child)

**Key principle:** Later policies **override earlier** ones when settings conflict — so the **OU-linked GPO closest to the object usually wins**. Exceptions are **Enforced** GPOs and **Block Inheritance** (covered next).

---

### Q50. What do "Block Inheritance" and "Enforced" do in Group Policy?
**Answer:**
- **Block Inheritance:** Applied to an **OU**, it stops GPOs from **higher levels** (parent OUs, domain) from flowing down into that OU. Useful when an OU needs an isolated configuration.
- **Enforced (formerly "No Override"):** Applied to a **GPO link**, it forces that GPO to apply **regardless of Block Inheritance** and gives it priority over conflicting lower-level GPOs.

**Interaction rule:** **Enforced beats Block Inheritance.** So a critical security GPO marked Enforced at the domain level will still apply even to an OU that blocks inheritance. This is a favorite interview "gotcha."

---

### Q51. What is the difference between `gpupdate` and `gpresult`?
**Answer:**
- **`gpupdate`:** **Applies/refreshes** Group Policy on the local machine. `gpupdate /force` reapplies **all** policies (not just changed ones). Used after editing a GPO to make changes take effect immediately rather than waiting for the default ~90-minute refresh.
- **`gpresult`:** **Reports** which policies are actually applied to a user/computer — a diagnostic (RSoP, Resultant Set of Policy) tool. `gpresult /r` gives a summary; `gpresult /h report.html` generates a detailed HTML report.

**In short:** `gpupdate` = *do it now*; `gpresult` = *show me what applied and why*.

---

### Q52. What is Group Policy Loopback Processing, and when do you use it?
**Answer:**
Normally, **user** settings come from GPOs linked to the **user's** OU, regardless of which computer they log into. **Loopback processing** changes this so that **user settings are determined by the computer's location** instead.

Two modes:
- **Replace:** The user settings from the **computer's** GPOs **replace** the user's normal settings.
- **Merge:** The computer's user settings are **combined** with the user's own (computer's win on conflict).

**Use case:** Shared or kiosk computers — like a **lab, conference room, or reception PC** — where you want a consistent locked-down user experience **no matter who logs in**.

---

### Q53. What is Security Filtering and WMI Filtering in Group Policy?
**Answer:**
By default, a GPO applies to **all authenticated users/computers** in its linked scope. Filtering narrows that:
- **Security Filtering:** Restricts a GPO to apply **only to specific users, groups, or computers** by adjusting which security principals have the **"Apply Group Policy"** permission. Example: apply a GPO only to the "Finance" security group within an OU.
- **WMI Filtering:** Uses a **WMI (Windows Management Instrumentation)** query to apply a GPO **only if certain conditions are true** — e.g., only on machines running a specific OS version, or only laptops (based on chassis type).

**Combined power:** You can link one GPO broadly but use filters so it only hits the exact targets you intend.

---

### Q54. What is the difference between the Computer Configuration and User Configuration sections of a GPO?
**Answer:**
Every GPO has two halves:
- **Computer Configuration:** Settings that apply to the **machine**, regardless of who logs in. They process at **startup** and on the background refresh. Example: firewall rules, services, machine-wide security settings.
- **User Configuration:** Settings that apply to the **user**, regardless of which machine they use. They process at **logon** and on refresh. Example: drive mappings, desktop wallpaper, Start menu layout.

**Tip:** If a GPO only contains settings in one half, you can **disable the unused half** to speed up processing.

---

### Q55. What are GPO Preferences, and how do they differ from Policies?
**Answer:**
- **Policies (Administrative Templates etc.):** **Enforced** settings. The UI is often **greyed out** for the end user, and if the GPO stops applying, the setting **reverts**. Policies "win" and are mandatory.
- **Group Policy Preferences (GPP):** **Configurable defaults** that users **can change**. They are applied but **not strictly enforced** by default (they "tattoo" the setting and can persist). GPP is used for things like **mapping drives, creating shortcuts, configuring printers, setting registry values, and managing local users/scheduled tasks**.

**Key flexibility:** Preferences support **Item-Level Targeting (ILT)** — apply a preference only if conditions match (OU, group, OS, IP range, etc.), giving very granular control.

---

### Q56. A GPO isn't applying to a user. How would you troubleshoot it?
**Answer:**
A structured approach:
1. **Confirm the link & scope:** Is the GPO **linked** to the correct OU/domain/site, and is the user/computer object actually **inside** that scope?
2. **Run `gpresult /r` (or `/h`):** See what's applied, what's filtered out, and any errors (RSoP).
3. **Check filters:** Verify **Security Filtering** (does the user/group have "Apply Group Policy"?) and any **WMI filter** that might exclude them.
4. **Check inheritance:** Is **Block Inheritance** or a higher **Enforced** GPO interfering? Is there a conflicting GPO with higher precedence?
5. **Computer vs User:** Is the setting in the right half (e.g., a user setting won't apply if it's in Computer Configuration)? Consider **loopback** if it's a per-machine scenario.
6. **Replication/SYSVOL:** Ensure the GPO replicated to the DC the client used (check `dcgpofix`/`repadmin`, SYSVOL health).
7. **`gpupdate /force`** and re-test; check the **Event Viewer** (Group Policy operational logs) for errors.

Mentioning this logical flow demonstrates real troubleshooting maturity in an interview.

---

# 6. File & Storage Services

### Q57. Compare NTFS, FAT32, and ReFS file systems.
**Answer:**
- **FAT32 (File Allocation Table):** Old, simple, broadly compatible. **No permissions/security**, max file size **4 GB**, max volume ~**32 GB** (practical Windows limit). Used mainly for USB drives and firmware/boot partitions.
- **NTFS (New Technology File System):** The standard Windows server file system. Supports **permissions (ACLs)**, **encryption (EFS)**, **compression**, **quotas**, **auditing**, **journaling** (for crash recovery), and very large files/volumes. This is what you use for most server data.
- **ReFS (Resilient File System):** A newer file system focused on **resiliency and scale** — automatic **integrity checking and self-healing** (with Storage Spaces), protection against data corruption, and huge capacities. Great for **large data volumes, backups, and virtualization storage**, but it lacks some NTFS features (e.g., no EFS encryption, no compression, can't be a boot volume).

---

### Q58. What is the difference between Share permissions and NTFS permissions?
**Answer:**
- **Share permissions:** Apply only when a folder is accessed **over the network** (via the share). They are simpler (Read, Change, Full Control) and **don't apply** when logged in locally.
- **NTFS permissions:** Apply **both locally and over the network**, are far more **granular** (Read, Write, Modify, Read & Execute, List, Full Control, plus special permissions), and follow the file/folder even if shared differently.

**Best practice:** Set **Share permissions** broadly (e.g., "Authenticated Users = Full Control" or "Change") and control actual access with **NTFS permissions**, which are more precise and apply everywhere.

---

### Q59. When Share and NTFS permissions conflict, which one wins?
**Answer:**
When accessing a folder **over the network**, **both** permission sets are evaluated, and the **most restrictive (effective) permission applies** — i.e., the **intersection** of the two.

**Example:** If Share permission = **Read** but NTFS = **Full Control**, the user gets **Read** over the network (the more restrictive of the two). Locally (not via the share), only **NTFS** applies, so they'd have Full Control.

**Also remember:** An explicit **Deny** generally **overrides Allow**. So a Deny anywhere usually wins.

---

### Q60. What is DFS, and what's the difference between DFS Namespaces and DFS Replication?
**Answer:**
**DFS (Distributed File System)** simplifies and adds resiliency to file shares across multiple servers.
- **DFS Namespaces (DFS-N):** Presents shares from **different servers** under a **single, unified folder path** (e.g., `\\corp.com\Shared\HR` regardless of which physical server hosts HR). Users don't need to know or care which server the data is on, and admins can move data between servers transparently.
- **DFS Replication (DFS-R):** **Replicates folder contents** between servers (multi-master, using **RDC — Remote Differential Compression**, which copies only the changed blocks). It keeps copies in sync for redundancy and for branch-office scenarios.

**Combined:** DFS-N + DFS-R gives you a single path **and** redundant, synchronized copies, so if one file server fails, clients are seamlessly served from another.

---

### Q61. What is RAID, and explain common RAID levels.
**Answer:**
**RAID (Redundant Array of Independent Disks)** combines multiple physical disks for **performance, redundancy, or both**. Common levels:
- **RAID 0 (Striping):** Splits data across disks for **speed**. **No redundancy** — one disk fails, all data is lost. (Capacity = all disks.)
- **RAID 1 (Mirroring):** **Duplicates** data on two disks. Survives one disk failure. (Usable capacity = 50%.)
- **RAID 5 (Striping with parity):** Stripes data + **distributed parity** across 3+ disks. Survives **one** disk failure. Good balance of capacity and redundancy. (Loses one disk's worth of capacity.)
- **RAID 6 (Double parity):** Like RAID 5 but survives **two** simultaneous disk failures (4+ disks). Safer for large arrays.
- **RAID 10 (1+0):** Mirrored pairs that are then striped — combines **speed and redundancy**, common for databases. (Usable capacity = 50%.)

**Interview note:** RAID is **not a backup** — it protects against disk failure, not deletion, corruption, or ransomware.

---

### Q62. What is the difference between Basic and Dynamic disks, and between MBR and GPT?
**Answer:**
**Basic vs Dynamic disks:**
- **Basic disk:** Standard disk using simple partitions. Most common.
- **Dynamic disk:** Supports software-based volume types like spanned, striped, mirrored, and RAID-5 volumes managed by Windows. (Largely superseded by **Storage Spaces** in modern setups.)

**MBR vs GPT (partition styles):**
- **MBR (Master Boot Record):** Older style. Max disk size **~2 TB**, max **4 primary partitions**. Uses legacy BIOS boot.
- **GPT (GUID Partition Table):** Modern style. Supports disks **far larger than 2 TB**, up to **128 partitions**, includes redundancy of the partition table, and is required for **UEFI** booting. Recommended for new servers.

---

### Q63. What is VSS (Volume Shadow Copy Service)?
**Answer:**
**VSS (Volume Shadow Copy Service)** creates **point-in-time snapshots** of a volume, even while files are open/in use. This enables:
- **"Previous Versions":** Users can right-click a file/folder and restore an earlier version without calling IT (when Shadow Copies are enabled on a share).
- **Consistent backups:** Backup software uses VSS to back up open/locked files (like databases) in a consistent state.

**Real-world value:** It's a quick **self-service recovery** mechanism for accidental edits/deletes — but it's a convenience feature, **not** a replacement for real backups (snapshots live on the same volume).

---

### Q64. What is FSRM (File Server Resource Manager)?
**Answer:**
**FSRM (File Server Resource Manager)** is a role service for managing and controlling data on file servers. Its main capabilities:
- **Quota Management:** Limit how much space a folder/volume can use (hard or soft quotas, with email alerts at thresholds).
- **File Screening:** **Block specific file types** (e.g., prevent `.mp3`/`.exe` from being saved in certain folders) or alert on them.
- **Storage Reports:** Generate reports on disk usage, large files, duplicate files, files by owner, etc.
- **File Classification & File Management Tasks:** Tag files and apply actions (e.g., expire/move old data).

**Real-world example:** Use file screening + quotas to stop a department's share from filling up with personal media and exceeding its allocation — and get alerted before it does.

---

### Q65. What is the difference between a hard quota and a soft quota?
**Answer:**
- **Hard quota:** **Strictly enforced** — once the limit is reached, the user **cannot** save more data to that folder. It physically prevents going over.
- **Soft quota:** **Not enforced** — the user **can exceed** the limit, but the system **logs and/or alerts** administrators. Used for **monitoring and reporting** rather than blocking.

**Use case:** Soft quotas help you understand usage trends before imposing hard limits; hard quotas guarantee a folder never exceeds its allocation.

---

### Q66. What are the standard NTFS permission levels?
**Answer:**
The common (basic) NTFS permissions, from least to most powerful:
- **Read:** View files/folders and their contents and attributes.
- **Read & Execute:** Read plus run executable files.
- **List Folder Contents:** See folder contents (folders only).
- **Write:** Create new files/folders and write data.
- **Modify:** Read, write, **and delete** files/folders (Read & Execute + Write + delete).
- **Full Control:** Everything, including **changing permissions** and **taking ownership**.

There are also **Special/Advanced permissions** for fine-grained control (e.g., "Delete subfolders and files," "Change permissions," "Take ownership"). **Permissions are inherited** from parent folders by default, and **Deny overrides Allow**.

---

### Q67. What is permission inheritance, and how do you handle "broken" inheritance?
**Answer:**
By default, **child files and folders inherit** NTFS permissions from their **parent folder** — so you set permissions once at the top and they flow down. This keeps administration simple.

**Breaking inheritance:** You can **disable inheritance** on a subfolder to give it a unique permission set (Windows offers to **convert inherited permissions to explicit** ones or remove them). This is sometimes necessary but should be **documented**, because heavily customized/broken inheritance makes permissions hard to audit and troubleshoot.

**Best practice:** Use **groups** (not individual users) on folders, set permissions high up, and **avoid excessive inheritance breaks**. When access seems wrong, use the **Effective Access** tab to see a user's true resulting permissions.

---

### Q68. What is Storage Spaces (and Storage Spaces Direct)?
**Answer:**
- **Storage Spaces:** A Windows storage virtualization feature that pools multiple physical disks into a **storage pool**, from which you carve out **virtual disks (spaces)** with chosen resiliency: **Simple** (striping, no redundancy), **Mirror** (copies for fault tolerance), or **Parity** (efficient redundancy like RAID-5). It's flexible, software-defined, and easy to expand by adding disks.
- **Storage Spaces Direct (S2D):** A **Datacenter-edition** feature that pools **local disks across multiple servers** (a cluster) into shared, highly available storage — the foundation of **Hyper-Converged Infrastructure (HCI)**, where compute and storage run on the same nodes.

---

### Q69. What is deduplication in Windows Server, and when is it useful?
**Answer:**
**Data Deduplication** is a role service that **identifies and removes duplicate chunks** of data on a volume, storing one copy and using pointers for the rest. This can **dramatically reduce disk usage**.

**Good use cases:** File shares with lots of similar documents, **VHD/VHDX libraries**, software deployment shares, and **backup target volumes** — where duplication is high, savings of 50–90% are possible.

**Cautions:** It's a background process (needs scheduling/resources) and isn't recommended for constantly changing, high-I/O workloads like live databases. Always ensure backups are dedup-aware.

---

# 7. Networking Essentials

### Q70. What is subnetting, and what is CIDR notation?
**Answer:**
**Subnetting** divides a large IP network into smaller **subnets** to improve organization, security (segmentation), and efficient use of addresses, and to reduce broadcast traffic.

**CIDR (Classless Inter-Domain Routing)** notation expresses the **subnet mask** as a slash and a number indicating how many bits are the network portion:
- `/24` = `255.255.255.0` = 256 addresses (254 usable) — a typical small office subnet.
- `/16` = `255.255.0.0` = 65,536 addresses.
- `/30` = `255.255.255.252` = 4 addresses (2 usable) — common for point-to-point links.

**Quick math:** Usable hosts = 2^(32 − prefix) − 2 (subtracting network and broadcast addresses).

---

### Q71. What are the private IP address ranges?
**Answer:**
Defined by **RFC 1918**, these ranges are **not routable on the public internet** and are used internally (with NAT to reach the internet):
- **10.0.0.0 – 10.255.255.255** (`10.0.0.0/8`)
- **172.16.0.0 – 172.31.255.255** (`172.16.0.0/12`)
- **192.168.0.0 – 192.168.255.255** (`192.168.0.0/16`)

Also worth knowing: **127.0.0.0/8** is **loopback** (e.g., `127.0.0.1` = "this machine"), and **169.254.0.0/16** is **APIPA** (link-local, indicates DHCP failure).

---

### Q72. What is the difference between TCP and UDP?
**Answer:**
Both are transport-layer protocols:
- **TCP (Transmission Control Protocol):** **Connection-oriented** and **reliable**. It establishes a connection (the **three-way handshake**: SYN, SYN-ACK, ACK), guarantees **ordered delivery**, and **retransmits** lost packets. Used where accuracy matters — web (HTTP/HTTPS), email, file transfer, AD/LDAP.
- **UDP (User Datagram Protocol):** **Connectionless** and **fast** but **unreliable** (no guaranteed delivery or ordering, no handshake). Used where speed matters more than perfection — DNS queries, **DHCP**, streaming, VoIP, and some monitoring (SNMP).

**One-liner:** TCP = reliable phone call; UDP = quick postcard.

---

### Q73. What is NIC Teaming, and why use it?
**Answer:**
**NIC Teaming** (Load Balancing/Failover, **LBFO**) combines **multiple physical network adapters (NICs)** into one logical interface.

Benefits:
- **Fault tolerance:** If one NIC or cable/switch port fails, traffic continues over the others — no outage.
- **Increased throughput:** Combined bandwidth via load distribution.

**Real-world use:** On critical servers (file servers, Hyper-V hosts), teaming two NICs to two different switches removes a single point of failure in network connectivity. (Note: Hyper-V also offers SET — Switch Embedded Teaming — for virtual switches.)

---

### Q74. Explain default gateway, subnet mask, and DNS server in IP configuration.
**Answer:**
- **Subnet mask:** Defines which portion of an IP is the **network** vs the **host**, so the device knows whether a destination is **local** (same subnet) or **remote**.
- **Default gateway:** The IP of the **router** the device uses to reach **other networks/the internet**. Traffic destined outside the local subnet is sent here.
- **DNS server:** The server used to **resolve names to IPs**. In an AD environment, clients must point to **internal DNS** (the DCs) so they can find domain services — pointing clients at a public DNS in a domain breaks AD functionality.

**Common misconfig:** Wrong DNS (or pointing to an ISP DNS in a domain) is a frequent cause of "can't log in / can't find domain" issues.

---

### Q75. What are some common TCP/UDP port numbers an admin should know?
**Answer:**
| Port | Protocol/Service | Notes |
|------|------------------|-------|
| 53 | **DNS** | UDP (queries) / TCP (zone transfers) |
| 67/68 | **DHCP** | UDP |
| 80 | **HTTP** | Web (unencrypted) |
| 443 | **HTTPS** | Web (TLS/SSL encrypted) |
| 88 | **Kerberos** | AD authentication |
| 389 | **LDAP** | Directory queries (636 = LDAPS) |
| 3268 | **Global Catalog** | (3269 = GC over SSL) |
| 445 | **SMB** | File/printer sharing |
| 135 | **RPC Endpoint Mapper** | AD/management |
| 3389 | **RDP** | Remote Desktop |
| 25 / 587 | **SMTP** | Email sending |
| 5985/5986 | **WinRM** | PowerShell remoting (HTTP/HTTPS) |

Knowing these helps with **firewall rules** and troubleshooting connectivity.

---

### Q76. What is the difference between IPv4 and IPv6?
**Answer:**
- **IPv4:** **32-bit** addresses (e.g., `192.168.1.10`), ~4.3 billion total — now **exhausted**, which is why NAT and private ranges are heavily used.
- **IPv6:** **128-bit** addresses (e.g., `2001:0db8:85a3::8a2e:0370:7334`), providing a virtually unlimited address space. It also has built-in features like **stateless address autoconfiguration (SLAAC)** and no need for NAT.

**Admin note:** Windows enables IPv6 by default and AD/Windows components rely on parts of it — Microsoft generally recommends **leaving IPv6 enabled** rather than disabling it, even if you primarily run IPv4.

---

### Q77. What tools would you use to troubleshoot network connectivity on a Windows Server?
**Answer:**
A practical toolkit:
- **`ping`** – tests basic reachability (ICMP).
- **`ipconfig /all`** – shows full IP configuration, DNS, gateway, MAC.
- **`tracert`** – traces the hops/route to a destination to find where it breaks.
- **`nslookup` / `Resolve-DnsName`** – tests DNS resolution.
- **`Test-NetConnection`** (PowerShell) – tests a port/connection (modern replacement combining ping + port test + route info).
- **`netstat -ano`** – shows active connections and listening ports (with PIDs).
- **`pathping`** – combines ping and tracert with per-hop loss stats.
- **`route print`** – views the routing table.

**Approach:** Work the **OSI/TCP-IP stack** bottom-up — link/cable → IP config → gateway → DNS → target service/port.

---

# 8. Hyper-V & Virtualization

### Q78. What is Hyper-V, and what is a hypervisor?
**Answer:**
A **hypervisor** is software that lets a single physical host run multiple **virtual machines (VMs)** by abstracting and sharing the underlying hardware (CPU, RAM, storage, network).

**Hyper-V** is Microsoft's built-in **Type 1 (bare-metal) hypervisor** — it runs directly on the hardware (the management OS itself becomes a privileged "parent" partition). This makes it efficient and suitable for production. It's a **role** you add to Windows Server.

**Two hypervisor types:**
- **Type 1 (bare-metal):** Runs directly on hardware (Hyper-V, VMware ESXi) — better performance, for servers.
- **Type 2 (hosted):** Runs as an app on top of an OS (VirtualBox, VMware Workstation) — for desktops/testing.

---

### Q79. What is the difference between Generation 1 and Generation 2 VMs in Hyper-V?
**Answer:**
- **Generation 1:** Emulates **legacy BIOS** hardware. Broadly compatible — supports older guest OSes and 32-bit OSes. Boots from emulated IDE controllers.
- **Generation 2:** Uses modern **UEFI firmware**. Supports **Secure Boot**, larger boot volumes (**GPT**), **PXE boot over a standard network adapter**, and hot-add of some resources. Requires a **64-bit, modern guest OS**. Faster boot and install.

**Choosing:** Use **Gen 2** for modern 64-bit OSes (better security and features); use **Gen 1** only if you need to run an older/32-bit OS or for compatibility reasons. You **cannot convert** between generations after creation.

---

### Q80. What is the difference between VHD and VHDX virtual disk formats?
**Answer:**
- **VHD (Virtual Hard Disk):** Older format, **max 2 TB**, no protection against corruption from power loss.
- **VHDX:** Newer format (Windows Server 2012+), **up to 64 TB**, includes a **logging mechanism** that protects against corruption during power failures, better performance, and supports larger block sizes.

**Recommendation:** Use **VHDX** for virtually everything modern; only use VHD for compatibility with very old platforms.

Also note disk **provisioning types**:
- **Dynamically expanding:** Grows as data is added (saves space, slight overhead).
- **Fixed size:** Allocates full size upfront (best performance/predictability).
- **Differencing:** A child disk that stores only changes relative to a parent (used for templates/test labs).

---

### Q81. What are Hyper-V Virtual Switch types?
**Answer:**
A **Virtual Switch (vSwitch)** connects VMs to networks. Three types:
- **External:** Binds to a **physical NIC**, giving VMs access to the **physical/external network** (and internet). Used in production for VMs that must reach the LAN.
- **Internal:** Allows communication **among VMs and between VMs and the host**, but **not** to the external physical network. Useful for isolated test setups that still need host access.
- **Private:** Allows communication **only among VMs on the same host** — completely isolated from the host and physical network. Used for sandboxed lab environments.

---

### Q82. What are Checkpoints (Snapshots) in Hyper-V, and what's the caution?
**Answer:**
A **Checkpoint** captures the **state, data, and configuration** of a VM at a point in time, so you can **revert** to it later — handy before applying updates or risky changes.

Types:
- **Standard checkpoint:** Captures running memory state too (application-level snapshot).
- **Production checkpoint:** Uses **VSS** inside the guest for a **data-consistent, application-supported** snapshot (recommended for production servers).

**Major caution:** Checkpoints are **NOT backups** and should not be kept long-term. They create **differencing (AVHDX) files** that grow over time, consuming disk and degrading performance. Leaving checkpoints on **domain controllers or databases** can cause serious issues (USN rollback on DCs). Always merge/delete them promptly.

---

### Q83. What is Dynamic Memory in Hyper-V?
**Answer:**
**Dynamic Memory** lets Hyper-V **automatically adjust the RAM allocated to a VM** based on its actual demand, within configured **minimum, startup, and maximum** values. Unused memory can be reclaimed and given to other VMs that need it.

**Benefit:** Higher **VM density** — you can run more VMs on a host by not statically over-allocating RAM to idle machines.

**Caution:** Some workloads (e.g., certain database servers like SQL Server) prefer **fixed/static memory** for predictable performance, so check vendor recommendations before enabling it.

---

### Q84. What is Live Migration, and what does it require?
**Answer:**
**Live Migration** moves a **running VM** from one Hyper-V host to another **with no downtime** (the VM keeps running and users stay connected). It's used for **load balancing** and **host maintenance** (patching/hardware work without an outage).

Requirements typically include:
- Hosts in the **same domain** (or with proper trust) and compatible CPUs.
- A **network for migration traffic** and proper **authentication** (Kerberos with constrained delegation, or CredSSP).
- Shared storage **or** Shared-Nothing Live Migration (which also moves the storage).
- Often part of a **Failover Cluster** with **Cluster Shared Volumes (CSV)** for high availability.

**Related:** **Quick Migration** involves a brief pause/save-state (some downtime), whereas Live Migration is seamless.

---

### Q85. What is Failover Clustering, and how does it relate to virtualization?
**Answer:**
**Failover Clustering** groups multiple servers (**nodes**) so that if one node fails, its workloads (called **clustered roles**) **automatically restart on another node**, providing **high availability (HA)**.

In virtualization, a **Hyper-V failover cluster** means VMs are clustered roles. If a host (node) crashes, its VMs **automatically restart on surviving nodes** — minimizing downtime.

Key components:
- **Shared storage** (SAN, or **Cluster Shared Volumes / Storage Spaces Direct**) accessible by all nodes.
- A **Quorum** mechanism (witness disk/file share/cloud witness) so the cluster can decide which nodes stay active and **avoid "split-brain"** (two halves both thinking they're in charge).
- A **heartbeat** network for nodes to monitor each other.

**HA vs Fault Tolerance:** Clustering provides **high availability** (quick automatic recovery with brief interruption), not zero-downtime fault tolerance.

---

# 9. Backup & Disaster Recovery

### Q86. Explain the difference between Full, Incremental, and Differential backups.
**Answer:**
- **Full backup:** Backs up **everything** every time. Simplest to restore (one set), but slowest and uses the most storage.
- **Incremental backup:** Backs up only data **changed since the last backup of any type** (full or incremental). **Fastest backups, least storage**, but **restore is slower** — you need the last full plus **every** incremental in order.
- **Differential backup:** Backs up all data **changed since the last full backup**. Backups grow over the week, but **restore is simpler** — just the last full **plus the latest differential**.

**Trade-off summary:** Incremental = fast backup / slow restore; Differential = slower-growing backup / faster restore.

---

### Q87. What is the difference between RTO and RPO?
**Answer:**
These define your disaster-recovery targets:
- **RTO (Recovery Time Objective):** The **maximum acceptable downtime** — how **quickly** you must restore service after an outage. ("We must be back up within 4 hours.")
- **RPO (Recovery Point Objective):** The **maximum acceptable data loss** — how much data (measured in **time**) you can afford to lose. ("We can lose at most 1 hour of data," meaning you must back up at least hourly.)

**Driving decisions:** A tight RPO means more frequent backups/replication; a tight RTO means faster recovery infrastructure (e.g., clustering, hot standby). These shape backup frequency and DR design — and budget.

---

### Q88. What is the 3-2-1 backup rule?
**Answer:**
A widely used best practice for resilient backups:
- **3** copies of your data (the original + 2 backups).
- **2** different **media/storage types** (e.g., local disk and tape/NAS).
- **1** copy stored **offsite** (and ideally **offline/immutable** to survive ransomware, fire, or theft).

**Modern extension (3-2-1-1-0):** add **1 offline/air-gapped/immutable** copy and verify **0 errors** by regularly testing restores. The key lesson: **a backup you've never tested restoring is not a backup you can trust.**

---

### Q89. What is Windows Server Backup, and what is Bare Metal Recovery?
**Answer:**
- **Windows Server Backup (WSB):** A built-in **feature** for backing up files, volumes, system state, or the full server to disk or network share. It uses **VSS** for consistent backups and supports scheduled jobs. Suitable for smaller environments (enterprises typically use dedicated backup solutions).
- **Bare Metal Recovery (BMR):** A backup that captures everything needed to restore the **entire server onto new/empty hardware** (or a new VM) — OS, system state, applications, and data. You boot from Windows installation/recovery media and restore the BMR image. It's the go-to for **total hardware failure** scenarios.

Related: **System State backup** captures critical OS components (registry, boot files, and on a DC, **Active Directory/SYSVOL**), which is essential for **AD recovery**.

---

### Q90. How do you back up and restore Active Directory?
**Answer:**
- **Backup:** Perform a **System State backup** (or full server/BMR) of at least one **Domain Controller** regularly. System State on a DC includes **AD database (NTDS.dit), SYSVOL, registry, and boot files**.
- **Restore types:**
  - **Non-authoritative restore:** Restore the DC, then let normal **replication** bring it up to date from other DCs. Used when you just need to recover a failed DC.
  - **Authoritative restore:** Restore specific objects (e.g., an accidentally deleted OU full of users) and **mark them authoritative** (`ntdsutil`) so they **replicate out** and overwrite the deletion on other DCs.
- **DSRM:** Authoritative restores are done in **Directory Services Restore Mode (DSRM)**, using the DSRM password set at promotion.

**Modern alternative:** If the **AD Recycle Bin** is enabled, recovering deleted objects is far easier (no DSRM/restore needed).

---

### Q91. What's the difference between High Availability, Fault Tolerance, and Disaster Recovery?
**Answer:**
- **High Availability (HA):** Minimizes downtime with **quick automatic recovery** — e.g., **failover clustering** restarts a service on another node within seconds/minutes. A brief interruption may occur.
- **Fault Tolerance (FT):** **Zero downtime** — redundant components run in lockstep so a failure is **invisible** to users (e.g., RAID mirroring, redundant power supplies, FT VMs). More expensive.
- **Disaster Recovery (DR):** The plan and capability to **recover after a major event** (site loss, fire, flood, ransomware) — typically involving **offsite backups/replication** and a documented **DR plan** with defined RTO/RPO.

**Relationship:** HA/FT keep you running through **local** failures; DR gets you back after a **catastrophic** event. You generally need all three layered appropriately.

---

# 10. Security & Hardening

### Q92. What is the Principle of Least Privilege, and how is it applied?
**Answer:**
The **Principle of Least Privilege (PoLP)** means giving users, accounts, and services **only the minimum access they need** to do their job — nothing more.

How it's applied in Windows environments:
- Don't make everyone a **local admin** or **Domain Admin**; use standard accounts for daily work.
- Admins use **separate privileged accounts** for admin tasks (not their everyday login).
- Use **role-based groups** and **delegation** (e.g., let helpdesk reset passwords only).
- Implement **tiered administration** and **Just-In-Time/Just-Enough Admin** where possible.

**Why it matters:** It **limits the blast radius** — if an account is compromised, the attacker gains far less. It's a foundational security control auditors look for.

---

### Q93. What is UAC (User Account Control)?
**Answer:**
**UAC (User Account Control)** is a Windows security feature that **prevents unauthorized changes** by prompting for consent/credentials before allowing actions that require **administrative privileges** (like installing software or changing system settings).

How it works: Even an admin runs with a **standard-user token** by default; when an elevated action is needed, UAC prompts to **"elevate"** (the consent dialog). This is **"Admin Approval Mode."**

**Security value:** It stops malware (and accidental changes) from silently making system-level modifications. Disabling UAC is generally discouraged as it weakens defenses.

---

### Q94. What is BitLocker, and what is a TPM?
**Answer:**
**BitLocker** is Windows' built-in **full-disk encryption** feature. It encrypts entire volumes so that if a disk or server is **physically stolen**, the data is unreadable without the key.

**TPM (Trusted Platform Module)** is a secure hardware chip on the motherboard that **stores encryption keys** and verifies boot integrity. BitLocker commonly uses the TPM to **automatically and securely unlock** the drive at boot **only if** the boot environment hasn't been tampered with.

**Recovery:** A **BitLocker recovery key** is generated and should be **escrowed** safely (e.g., in AD, a key management system, or a secured store) so you can unlock the drive if the TPM/hardware changes.

---

### Q95. What is PKI, and what role does a Certificate Authority (CA) play?
**Answer:**
**PKI (Public Key Infrastructure)** is the framework of policies, hardware, software, and certificates that manages **digital certificates** and **public/private key** encryption — enabling secure identity, encryption, and signing.

A **Certificate Authority (CA)** is the trusted entity that **issues, validates, and revokes** digital certificates. In Windows, **AD CS (Active Directory Certificate Services)** lets you run an internal CA.

Typical uses:
- **HTTPS/TLS** for internal websites and services.
- **Smart card / certificate-based authentication.**
- **Code signing**, **email signing/encryption (S/MIME)**, **VPN/Wi-Fi (802.1X)**, and **LDAPS**.

**Hierarchy:** Best practice is a **two-tier PKI** — an **offline Root CA** plus an online **Issuing (Subordinate) CA** — to protect the root. **CRLs (Certificate Revocation Lists)** publish which certificates are no longer valid.

---

### Q96. What is the difference between Windows Defender Firewall inbound and outbound rules?
**Answer:**
**Windows Defender Firewall with Advanced Security** filters network traffic based on rules:
- **Inbound rules:** Control traffic **coming into** the server. By default, **most inbound traffic is blocked** unless a rule allows it (e.g., open port 3389 for RDP, 445 for SMB).
- **Outbound rules:** Control traffic **leaving** the server. By default, **most outbound traffic is allowed**, but you can restrict it for tighter security (e.g., block a server from reaching the internet).

Rules can be scoped by **profile** — **Domain**, **Private**, or **Public** — so a server behaves differently depending on the network it's on. Firewall settings are commonly managed at scale via **Group Policy**.

---

### Q97. What are some key Windows Server hardening best practices?
**Answer:**
Common hardening measures:
- **Patch promptly** (OS and apps); use **WSUS** or a patch tool.
- **Remove unneeded roles/features/software** to shrink the attack surface; prefer **Server Core**.
- **Apply least privilege**; rename/disable default accounts where appropriate; use strong **password and lockout policies**.
- **Use separate admin accounts** and limit **Domain Admins**; implement **tiered admin** and **LAPS** for local admin passwords.
- **Configure the firewall** (allow only required ports).
- **Disable legacy/insecure protocols** (SMBv1, old TLS, and minimize NTLM).
- **Enable auditing/logging** and forward logs to a central **SIEM (Security Information and Event Management)** system.
- **Encrypt** sensitive data (BitLocker, LDAPS, TLS) and follow a **security baseline** (e.g., Microsoft Security Baselines / CIS Benchmarks) applied via GPO.

---

### Q98. What is LAPS (Local Administrator Password Solution)?
**Answer:**
**LAPS** automatically **manages and rotates the local Administrator password** on domain-joined machines, storing each machine's **unique, random password** securely (in AD attributes, with the newer **Windows LAPS** also supporting Entra ID).

**Why it matters:** Without LAPS, many organizations use the **same local admin password everywhere**, which lets an attacker who compromises one machine **move laterally** to all others (pass-the-hash). LAPS ensures **every machine has a different, regularly rotated password**, and only authorized admins can read it. It's a high-impact, low-cost security control.

---

### Q99. What is the difference between a service account and a managed service account (gMSA)?
**Answer:**
- **Standard service account:** A regular user account used to run a service/application. Drawback: its **password must be managed manually** (and is often set to never expire or shared/known), which is a security and maintenance risk.
- **Managed Service Account (MSA) / Group Managed Service Account (gMSA):** Special AD accounts where **Windows automatically manages and rotates the password** (no human knows it). A **gMSA** can be used across **multiple servers** (e.g., a web farm or cluster), unlike a standalone MSA which is tied to one machine.

**Benefit:** gMSAs eliminate manual password handling, support automatic rotation, and reduce credential-theft risk — best practice for service identities where supported.

---

# 11. PowerShell & Administration

### Q100. What is PowerShell, and how is it structured (cmdlets)?
**Answer:**
**PowerShell** is Microsoft's **command-line shell and scripting language** built for automation and configuration management. Unlike older shells that pass plain text, PowerShell passes **objects** (structured data) through the **pipeline**, making it very powerful for filtering and processing.

**Cmdlets** (command-lets) follow a consistent **Verb-Noun** naming convention:
- `Get-Service`, `Stop-Service`, `New-ADUser`, `Set-DnsServerForwarder`, `Restart-Computer`.
- Common verbs: **Get** (retrieve), **Set** (change), **New** (create), **Remove** (delete), **Start/Stop/Restart**.

**Example:** `Get-Service | Where-Object {$_.Status -eq 'Stopped'}` lists all stopped services. This predictable structure makes commands easy to guess and learn.

---

### Q101. What is PowerShell Remoting, and how does it work?
**Answer:**
**PowerShell Remoting** lets you run commands on **remote computers** as if you were local. It's built on **WinRM (Windows Remote Management)**, which uses the **WS-Management** protocol over ports **5985 (HTTP)** / **5986 (HTTPS)**.

Common uses:
- **One-to-one:** `Enter-PSSession -ComputerName Server01` for an interactive remote session.
- **One-to-many (fan-out):** `Invoke-Command -ComputerName Server01,Server02 -ScriptBlock { Get-Service }` runs a command on **many servers at once**.

**Real-world value:** You can patch, configure, or query **hundreds of servers** from one console — essential for managing **Server Core** and large fleets. WinRM must be enabled (`Enable-PSRemoting`) and is often pre-enabled on Windows Server.

---

### Q102. Give examples of useful PowerShell commands for daily server administration.
**Answer:**
A handful that frequently come up:
- **Roles/Features:** `Install-WindowsFeature -Name DNS` / `Get-WindowsFeature`
- **Services:** `Get-Service`, `Restart-Service -Name Spooler`
- **AD (RSAT/AD module):** `Get-ADUser`, `New-ADUser`, `Get-ADComputer`, `Unlock-ADAccount`, `Search-ADAccount -LockedOut`
- **Network:** `Get-NetIPAddress`, `Test-NetConnection -ComputerName host -Port 443`, `Resolve-DnsName`
- **Disk/Storage:** `Get-Volume`, `Get-PhysicalDisk`, `Get-Disk`
- **Events/Troubleshooting:** `Get-EventLog`/`Get-WinEvent`, `Get-Process | Sort CPU -Descending`
- **Updates:** `Get-HotFix`
- **Help:** `Get-Help <cmdlet> -Examples`, `Get-Command *AD*`

**Tip for interviews:** Knowing how to **discover** commands (`Get-Command`, `Get-Help`, `Get-Member`) is as valuable as memorizing them.

---

### Q103. What is the difference between RSAT, Windows Admin Center, and Server Manager for remote administration?
**Answer:**
- **RSAT (Remote Server Administration Tools):** A set of tools (consoles like AD Users and Computers, DNS Manager, plus PowerShell modules) installed on a **management workstation** to manage server roles **remotely** — so you don't log into servers directly. Ideal for managing **Server Core** boxes.
- **Server Manager:** The built-in dashboard (Desktop Experience) for adding roles/features and monitoring **multiple servers** from one console.
- **Windows Admin Center (WAC):** A modern, **browser-based** management platform that consolidates many tools (server config, certificates, files, Hyper-V, clusters, updates) into one web UI. It's increasingly the go-to for managing both GUI and Core servers, on-prem and hybrid.

**Trend:** Microsoft steers admins toward **WAC + PowerShell** for scalable, GUI-light management.

---

# 12. Patch Management & Monitoring

### Q104. What is WSUS, and why use it?
**Answer:**
**WSUS (Windows Server Update Services)** is a role that lets you **centrally manage and distribute Microsoft updates** to computers in your network.

How it helps:
- Downloads updates **once** from Microsoft, then distributes them internally — **saving internet bandwidth**.
- Lets admins **approve/decline** specific updates and **test** them on pilot groups before broad rollout.
- Uses **computer groups** to stage deployments (e.g., test → pilot → production) and provides **reporting** on compliance.

**Real-world value:** Instead of every machine pulling updates from the internet uncontrolled, WSUS gives you **control, testing, and visibility** — reducing the risk of a bad patch taking down production. (Larger orgs may use **SCCM/Configuration Manager** or modern cloud-based tools, but the WSUS concept underpins many of them.)

---

### Q105. What is Patch Tuesday, and how should you approach patch management?
**Answer:**
**Patch Tuesday** is the **second Tuesday of each month**, when Microsoft releases its regular security and quality updates. (Critical out-of-band patches can come anytime.)

A sound patch strategy:
1. **Inventory & categorize** systems by criticality.
2. **Test** patches on a **pilot/non-production group** first.
3. **Stage the rollout** in rings (test → pilot → broad production).
4. **Schedule** during maintenance windows; ensure **backups exist** before patching.
5. **Have a rollback plan** (snapshots/backups) in case a patch causes issues.
6. **Verify and report** compliance afterward.

**Why it matters:** Unpatched systems are a top breach cause, but **blindly applying patches** can break production — so balance **speed (security)** with **testing (stability)**.

---

### Q106. What is Event Viewer, and which logs matter most?
**Answer:**
**Event Viewer** is the central tool for viewing **logs** about system, security, and application events — essential for troubleshooting and auditing.

Key logs:
- **System:** OS and hardware/driver events (service failures, disk errors, boot issues).
- **Application:** Events from installed applications.
- **Security:** **Audit events** — logons/logoffs, account lockouts, privilege use, object access (vital for security investigations).
- **Setup:** Installation/role configuration events.
- **Applications and Services Logs:** Role-specific operational logs (DNS, DFS Replication, Group Policy, Hyper-V, etc.).

**Event levels:** Information, Warning, Error, Critical. **Event IDs** are highly searchable (e.g., **4625** = failed logon, **4740** = account lockout). Forwarding logs to a **SIEM** centralizes monitoring across many servers.

---

### Q107. How would you troubleshoot a server with high CPU or memory usage?
**Answer:**
A structured approach:
1. **Identify the culprit:** Use **Task Manager**, **Resource Monitor**, or `Get-Process | Sort-Object CPU -Descending` to find the top process(es) consuming CPU/RAM.
2. **Determine what it is:** Map the process to a service/application. Is it expected (a busy SQL/IIS) or rogue (malware, a runaway process, a memory leak)?
3. **Check Event Viewer** for related errors/warnings around the timeframe.
4. **Use Performance Monitor (PerfMon):** Set up **counters** (CPU %, Available MBytes, disk queue length, page faults) and **Data Collector Sets** to capture trends and rule out a **bottleneck** elsewhere (disk, memory paging).
5. **Take action:** Restart the offending service, apply updates/patches for known leaks, scale resources, or rebalance workloads (e.g., live-migrate a VM).
6. **Prevent recurrence:** Set up **monitoring/alerts** and right-size the server.

**Key tools to name:** Task Manager, **Resource Monitor (resmon)**, **Performance Monitor (perfmon)**, Event Viewer, and PowerShell.

---

# 13. Real-World Troubleshooting Scenarios

### Q108. Scenario: Users in one branch office suddenly can't log in, but other offices are fine. How do you troubleshoot?
**Answer:**
Work methodically, starting from what's unique to that branch:
1. **Scope it:** All users at that branch affected, but other sites fine → likely a **branch-local** infrastructure issue (DC/DNS/network), not a domain-wide problem.
2. **Check the local DC:** Is the branch **Domain Controller** (or RODC) **online and healthy**? If it's down and there's no local fallback, logons fail or are slow. Check services (`AD DS`, `Netlogon`, `DNS`), and `dcdiag`.
3. **Check DNS:** Are clients pointing to a **reachable DNS server**? Bad/unreachable DNS breaks DC location (`SRV` records). Verify with `nslookup` / `Test-NetConnection`.
4. **Check the WAN link:** Is connectivity to other sites/DCs up? A **WAN outage** can isolate the branch — `ping`, `tracert`, check the router/firewall.
5. **Check time sync:** Large **clock skew** breaks **Kerberos** (logon failures). Verify `w32tm /query /status`.
6. **Check replication:** If the branch DC is isolated, it may have **stale credentials** (especially an RODC without cached passwords). `repadmin /showrepl`.
7. **Event logs:** Look at **Security** and **System** logs on the DC and clients for clues (lockouts, Netlogon, Kerberos errors).

**Likely culprits:** Branch DC down, **DNS misconfiguration**, **WAN/link failure**, or **time skew**.

---

### Q109. Scenario: A file share is performing slowly and users complain about access. What's your approach?
**Answer:**
Investigate across the layers:
1. **Clarify the symptom:** Slow to **open files**, slow to **list folders**, or **intermittent disconnects**? For everyone or specific users/folders?
2. **Server health:** Check **CPU, memory, and especially disk** performance on the file server (Resource Monitor / PerfMon — watch **disk queue length** and latency). A saturated disk or failing RAID array is a common cause.
3. **Network:** Check the server's NIC for errors/saturation; consider **NIC teaming** status. Test latency between client and server (`Test-NetConnection`, file copy benchmarks). Look for a **duplex mismatch** or a flaky switch port.
4. **Storage:** Check the underlying volume — fragmentation (on HDDs), low free space, RAID rebuild in progress, or a degraded array.
5. **Permissions/structure:** Excessive **broken inheritance** or huge folders with many files/ACLs can slow enumeration. Antivirus scanning the share can also add latency — check **AV exclusions**.
6. **SMB/protocol:** Ensure modern **SMB** is in use (SMBv1 disabled), and check for **SMB signing** overhead or **opportunistic locking** issues.
7. **DFS:** If it's a DFS share, verify clients are hitting the **correct/nearest target** (site costing) and that a **referral isn't pointing across a slow WAN link**.
8. **Logs & monitoring:** Event Viewer for disk/SMB errors; set up monitoring to catch recurrence.

**Common root causes:** Disk/storage bottleneck, network saturation, antivirus scanning, or a misrouted DFS referral.

---

### Q110. Scenario: You need to grant a new "Accounting" team access to a shared folder following best practices. Walk me through it.
**Answer:**
Apply the **AGDLP** model and least privilege:
1. **Create a Global group** for the role, e.g., `G-Accounting-Users`, and **add the user accounts** to it. (Accounts → Global.)
2. **Create a Domain Local group** for the resource, e.g., `DL-AccountingShare-Modify`, describing the access it grants. (Global → Domain Local.)
3. **Nest** the Global group **inside** the Domain Local group (`G-Accounting-Users` → member of `DL-AccountingShare-Modify`).
4. **Assign NTFS permissions** on the folder to the **Domain Local group** (e.g., **Modify**), not to individual users. (Domain Local → Permissions.)
5. **Set Share permissions** appropriately (e.g., Authenticated Users = Change/Full at the share level), letting **NTFS be the precise control** — remembering the **most restrictive of share + NTFS** wins over the network.
6. **Verify** with the **Effective Access** tab for a sample user, and document the design.

**Why this way:** It's **scalable and auditable** — onboarding/offboarding is just adding/removing a user from the Global group, permissions live in one place, and you never assign rights to individual accounts. This answer signals you understand **enterprise permission design**, not just clicking "Add."

---

## ✅ Final Preparation Tips

- **Master the "big four" roles:** Active Directory, DNS, DHCP, and Group Policy — these dominate Windows Server interviews.
- **Always relate concepts to real scenarios** ("When the DC went down, here's what happened…") — interviewers reward practical thinking.
- **Know your troubleshooting tools** and a logical, layered methodology (it's often more valued than memorized trivia).
- **Be ready for "why," not just "what"** — e.g., *why* AGDLP, *why* Server Core, *why* least privilege.
- **Practice explaining clearly** — being able to teach a concept simply shows true understanding.
- **Remember the admin mantras:** *"It's always DNS,"* *"RAID is not a backup,"* and *"a backup you haven't tested isn't a backup."*

> **You've now covered 110 questions spanning fundamentals to advanced, real-world scenarios. Review these, practice articulating them aloud, and you'll be well-prepared to face Windows Server Administration interviews with confidence.**

---
*Good luck with your interview! 🚀*
