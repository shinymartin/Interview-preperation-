# Azure VM Naming Convention Guide (Beginner Friendly)

This guide teaches you how to "read" an Azure Virtual Machine size name — like `Standard_D4s_v5` or `Standard_E16-8ads_v6` — and instantly know what kind of computer it is, without looking anything up. No prior Azure knowledge is assumed.

---

## 1. The big picture — what's actually in a name?

Think of an Azure VM name as a **label on a box**, similar to a car model name like "Toyota Corolla LE 1.8L Hybrid 2024." Each word or letter in that car name tells you something: the brand, trim level, engine size, fuel type, and year. Azure VM names work exactly the same way — just with letters instead of words.

The full pattern looks like this:

```
Standard_[Family][Vendor?][vCPUs][-Constrained?][Features?]_[GPU?]_v[Version]
```

Don't worry about memorizing that — every piece is explained below, one at a time, with plain-English examples.

---

## 2. What is a vCPU? (the basic building block)

Before decoding names, you need to understand **vCPU**, because almost every VM name is built around this number.

- A physical computer (the "host") in an Azure data center has real, physical CPU cores.
- Azure splits the processing power of those physical cores into smaller shareable units called **vCPUs (virtual CPUs)** — think of it like slicing a pizza. The whole pizza is the physical CPU; each slice is a vCPU handed to a VM.
- Each vCPU usually corresponds to one hardware "thread" on the physical processor, not a full dedicated core.

### Why vCPU count matters

| Aspect | Why it matters |
|---|---|
| **Compute power** | More vCPUs = more parallel processing, useful for busy web servers, batch jobs, multi-threaded apps. |
| **Memory tied to vCPU** | Each VM family has a fixed ratio of RAM per vCPU (e.g., 4 GB per vCPU for D-series). Pick vCPU count, and you're also picking your RAM amount. |
| **Cost** | Price scales with vCPU count — more vCPUs, higher hourly cost. |
| **Software licensing** | Software like SQL Server is often billed **per vCPU**, so extra vCPUs can quietly increase licensing costs. |
| **Network & disk speed limits** | Azure raises network throughput and disk IOPS limits as vCPU count goes up. |
| **Right-sizing** | Too few vCPUs → app slows down under load. Too many → you're paying for idle capacity you don't use. |

**In short:** the number in a VM name isn't just "how many CPUs" — it's a shortcut for the VM's whole performance and cost tier.

---

## 3. Decoding the name step-by-step

Let's break down a real example, piece by piece:

### Example: `Standard_E16-8ads_v6`

| Piece | Value | What it means |
|---|---|---|
| `Standard_` | — | Just a fixed prefix Azure puts on every VM size name. Ignore it. |
| `E` | Family | **Memory optimized** — built for RAM-hungry workloads |
| `16` | Hardware size | This VM is built on 16-vCPU-class hardware |
| `-8` | Constrained cores | Only **8 vCPUs are actually turned on/usable** (explained in detail in Section 6) |
| `a` | Vendor | **AMD** processor |
| `d` | Feature | Has a **local (temporary) disk** |
| `s` | Feature | Supports **Premium SSD / Ultra Disk** storage |
| `v6` | Version | 6th hardware generation of the E-series |

Once you can do this for one name, you can do it for any name — it's always the same recipe.

---

## 4. Family letter — "what job is this VM built for?"

This is the single most important letter. It tells you the VM's purpose, like a car being labeled "SUV" vs "sports car."

| Letter | Family name | What it's good for | Easy way to picture it |
|---|---|---|---|
| `A` | Basic / Entry-level | Dev/test, small non-critical workloads | The "starter" trim |
| `B` | Burstable | Workloads that are idle most of the time but occasionally spike | A car that can sprint occasionally but isn't built for constant high speed |
| `D` | General purpose | Balanced, everyday workloads (web servers, small-medium databases) | The "default family sedan" — good for most needs |
| `E` | Memory optimized | Databases, caches, SAP, anything RAM-hungry | A cargo van — built to carry a lot (of memory) |
| `F` | Compute optimized | CPU-heavy tasks: batch jobs, gaming servers, builds | A sports car — built for raw speed (CPU) |
| `G` / `M` | Memory & storage optimized (very large) | Huge enterprise databases | An 18-wheeler truck — the biggest hauler |
| `L` | Storage optimized | Big data, NoSQL, workloads needing fast local disk | A moving truck — built to carry lots of storage |
| `N` (`NC`/`ND`/`NV`) | GPU accelerated | AI/ML training, video rendering, visualization | Has an "N" because it's powered by an **N**VIDIA GPU |
| `H` | High performance computing (HPC) | Scientific simulation, weather modeling | Literally stands for **H**PC |

---

## 5. Vendor / sub-family letter — "whose chip is inside?"

Right after the family letter, you may see a lowercase letter for the CPU brand:

| Letter | Meaning |
|---|---|
| *(nothing shown)* | Intel processor (this is the default, so Azure doesn't bother labeling it) |
| `a` | AMD processor |
| `p` | ARM-based processor (Microsoft Cobalt or Ampere Altra chips) |

**Simple trick:** if you don't see a letter here, assume Intel.

---

## 6. Understanding "constrained" vCPUs (the `-8` in `E16-8`)

This is a special notation that confuses most beginners, so let's slow down.

**The question:** In a name like `E16-8ads_v6`, does that mean the VM has 16 vCPUs but you can only use 8 of them?

**The answer: Yes, exactly.**

Here's what's really happening:

1. Azure takes the **E16** size (a real, normal 16-vCPU VM with a certain amount of RAM, network bandwidth, and disk speed).
2. It then **disables half the vCPUs**, leaving only 8 active and usable by your operating system.
3. However, you **still keep all the memory, network bandwidth, and disk speed of the full 16-vCPU size.**

**Why would anyone want this?**

Mainly to save money on **per-core software licensing**. Some software (like SQL Server) charges you based on how many vCPUs are active. So:

- If you need a *lot of RAM* (like a database server) but don't need all 16 vCPUs worth of compute,
- You can get an `E16-8` VM: pay for and license only 8 active vCPUs,
- But still enjoy the RAM/network/disk performance that comes with the bigger 16-vCPU size.

**Think of it like this analogy:** Imagine renting a 4-bedroom house but the landlord locks 2 bedrooms so you only pay "2-bedroom" rent — yet you still get the house's full backyard, garage, and driveway (the "extra" resources that scale with house size, not bedroom count).

You'll see this constrained-core pattern on several families (D, E, F, M, etc.) — always written as `[FullSize]-[ActiveCores]`, like `D64-32s_v5` (64-vCPU hardware, only 32 active).

---

## 7. Feature letters — "extra capabilities bolted on"

After the vCPU number, you may see one or more lowercase letters. A name can have **zero, one, or several** of these — they just get added on:

| Letter | Feature | Plain-English meaning |
|---|---|---|
| `a` | AMD CPU | Same as the vendor letter — AMD-based |
| `d` | Local disk present | Has fast temporary disk built into the physical host |
| `s` | Premium storage support | Can attach Premium SSD / Ultra Disk (fast, production-grade storage) |
| `i` | Isolated | Runs on a dedicated physical server, not shared with other customers |
| `l` | Low memory | Lower RAM-per-vCPU than the standard version of that series |
| `m` | High memory | The highest RAM-per-vCPU option in that series |
| `p` | ARM CPU | Same as vendor letter — ARM-based |
| `t` | Tiny local disk | A very small local disk variant |
| `C` | Confidential computing | Extra hardware-level data encryption/security |
| `B` | Block storage only | Optimized specifically for block storage performance |

**Easy memory trick — the word "SAID":**
- **S** = SSD support
- **A** = AMD chip
- **I** = Isolated hardware
- **D** = Disk (local) present

If a VM name "SAID" one of these letters, that's the feature it has.

**Golden rule for beginners:** If you don't see an `s` in the name, that VM **cannot** use Premium SSD or Ultra Disk — only slower standard storage. For any production or database workload, always look for the `s`.

---

## 8. "Local disk" (`d`) vs. "Data disk" — a common point of confusion

This is one of the most misunderstood parts of Azure VM naming, so it deserves its own detailed section.

### The short answer

The `d` letter in a VM name does **not** mean "a data disk is attached to this VM." It means something much narrower: **this VM size comes with a built-in local (temporary) disk on the physical host.** That's a very different thing from the data disks you attach yourself.

### What the `d` (local disk) actually is

- It's physical storage that lives on the same physical server hardware the VM is running on.
- Azure provisions it automatically as part of the VM size — you never had to create it, and you can't remove it or resize it independently.
- It's meant for **temporary, disposable data**: the OS page/swap file, temp files, cached data, or anything an application can afford to lose.
- **It is not persistent.** Its contents can be wiped whenever the VM is stopped/deallocated, moved to different host hardware, or resized to a different VM size. Azure gives no guarantee this data survives any of those events.
- Windows commonly maps this to the `D:` drive by default; Linux typically mounts it at `/mnt` or `/mnt/resource`. That's simply a default mount point — it doesn't make it a "real" persistent disk.

**Rule of thumb:** never store your database files, application data, or anything you care about on the local disk. Treat it like a scratchpad that can be erased at any time.

### What a "data disk" actually is

- A data disk is a **separate managed disk resource** (Standard HDD, Standard SSD, Premium SSD, or Ultra Disk) that you explicitly create and attach to the VM, in addition to its OS disk.
- You choose its size, performance tier, and how many you want to attach.
- It is fully **persistent** — it survives VM reboots, deallocation, resizing, and even VM deletion (if you detach it instead of deleting it, or if you configure it to not auto-delete with the VM).
- This is where real workloads belong: databases, application files, logs you need to keep, user uploads, etc.
- Every Azure VM size can attach at least some tier of data disk — the `s` feature letter (see Section 7) only tells you whether that size can attach the **Premium SSD / Ultra Disk** tiers specifically, not whether data disks are possible at all.

### Side-by-side comparison

| | `d` — Local (temporary) disk | Data disk (managed disk you attach) |
|---|---|---|
| Who creates it | Azure, automatically, as part of the VM size | You, manually, as a separate resource |
| Shown in the name by | `d` | Indirectly via `s` (tells you if Premium/Ultra tiers are supported) |
| Persistent across reboot? | No | Yes |
| Persistent across deallocation/resize/host move? | No — can be wiped | Yes |
| Typical use | Page file, swap, temp/cache files | Databases, application data, anything important |
| Can you resize/choose it? | No — fixed size tied to the VM size | Yes — you choose size and performance tier |
| Cost | Included in the VM's price | Billed separately, based on size/tier |

### Why this distinction matters in practice

If you see a VM name with `d` in it and assume "great, this VM has storage for my database," that's a mistake — you could lose your data the next time Azure moves your VM to different hardware (which can happen during maintenance, resizing, or certain restarts). Always provision a proper **data disk** for anything that needs to persist, regardless of whether the VM name has a `d` in it.

Conversely, a VM name **without** `d` simply means that particular size doesn't come with a local temp disk at all — this is common in some newer, storage-optimized-for-persistence series where Azure intentionally omits local disk to reduce cost and encourage using remote managed disks instead. It has nothing to do with your ability to attach data disks.

### Windows Server gotcha: the `D:` drive letter is already taken

This trips up a lot of admins in practice, so it deserves a callout.

- On Windows Server VMs, Azure **automatically assigns the local temporary disk to drive letter `D:` by default**, every time the VM boots. This is done automatically by the Azure VM agent — you didn't configure it, and you can't easily stop it from happening.
- Because `D:` is already occupied by that temporary disk, if you attach a persistent **data disk** and try to also assign it the letter `D:`, Windows won't allow two volumes to share the same letter — you'll need to pick a different one.

**What to do instead:**
- Attach your data disk as usual (Azure portal, CLI, or PowerShell).
- When you initialize/format it in Windows Disk Management, give it a **different, unused letter** — commonly `F:`, `G:`, or similar. (`E:` is sometimes reserved for the DVD-ROM drive on certain images, so many admins skip straight to `F:`.)
- **Never repurpose `D:` for persistent data**, even if you manage to force it. On every reboot, resize, or host migration, Azure's agent will try to remount the local temp disk back onto `D:` — which can silently break application paths or cause a drive-letter conflict.

**Watch out for legacy applications:** some older software (including some legacy SQL Server installs) is hardcoded to expect data on `D:`. If you inherit a VM configured that way, don't assume it's safe just because it "has always worked" — verify whether `D:` is actually the ephemeral local disk before trusting it with anything important.

## 9. GPU naming — a special case

GPU VMs (family starts with `N`) insert the actual GPU model name into the string, separated by underscores:

```
Standard_NC40ads_H100_v5
```

Here, `H100` tells you it has an NVIDIA H100 GPU attached. Other common GPU codes you might see: `A100`, `V100`, `MI300X`. This is the easiest part to read — the GPU name is spelled out for you.

---

## 10. Version number — "how new is the hardware?"

The last part of the name, `_v2`, `_v5`, `_v6`, etc., tells you the **generation** of that series.

- Higher number = newer hardware = generally better price-for-performance, faster networking, and better efficiency.
- If there's **no version number at all** (e.g., plain `B-series`, `HB-series`), it just means that series has only ever had one generation — so there's nothing to distinguish.

**Beginner tip:** when deploying something new, always pick the **highest version number available** for that family — it's rarely more expensive and is usually the better deal.

---

## 11. Memory tricks — recall the whole system without memorizing

You don't need to memorize any table. Use these shortcuts instead:

**Family letters — remember with a sentence:**
> "**A**lex **B**uys **D**onuts **E**very **F**riday"
> A = Basic/Entry, B = Burstable, D = Default/general purpose, E = Extra memory, F = Fast/compute

**Other family letters:**
- `G`/`M` = **G**iant / **M**assive (biggest machines)
- `L` = **L**ots of storage
- `N` = **N**Vidia (GPU)
- `H` = **H**PC (literally the acronym)

**Feature letters — remember the word "SAID":**
- S = SSD, A = AMD, I = Isolated, D = Disk

**Vendor letters — remember "Apple Pie":**
- **a** = AMD, **p** = ARM ("Pie"). No letter shown = Intel (the default).

**Constrained cores — remember the "locked bedroom" analogy:**
- `16-8` = 16-vCPU-size house, but only 8 bedrooms (vCPUs) unlocked for use. You still get the full house's yard (memory/network/disk).

---

## 12. Fully worked examples (practice reading names yourself)

**`Standard_D4s_v5`**
- `D` → General purpose
- *(no letter)* → Intel CPU
- `4` → 4 vCPUs
- `s` → Premium SSD supported
- `v5` → 5th generation

→ A balanced, everyday, Intel-based 4-vCPU VM with fast storage support. Good for a small web app.

**`Standard_E32as_v6`**
- `E` → Memory optimized
- `a` → AMD CPU
- `32` → 32 vCPUs
- `s` → Premium SSD supported
- `v6` → 6th generation

→ A big, RAM-heavy, AMD-based VM — good for a large database or in-memory cache.

**`Standard_F16s_v2`**
- `F` → Compute optimized
- `16` → 16 vCPUs
- `s` → Premium SSD supported
- `v2` → 2nd generation

→ A CPU-heavy VM good for batch processing, builds, or game servers.

**`Standard_E16-8ads_v6`**
- `E` → Memory optimized
- `16` → Built on 16-vCPU hardware
- `-8` → Only 8 vCPUs active/usable (see Section 6)
- `a` → AMD CPU
- `d` → Local disk present
- `s` → Premium SSD supported
- `v6` → 6th generation

→ A memory-optimized VM that gives you 16-vCPU-class memory/network/disk performance, but only bills/licenses 8 active vCPUs — great for RAM-hungry, per-core-licensed database workloads.

**`Standard_NC24ads_A100_v4`**
- `NC` → GPU, compute-focused family
- `24` → 24 vCPUs
- `a` → AMD CPU
- `d` → Local disk present
- `s` → Premium SSD supported
- `A100` → NVIDIA A100 GPU attached
- `v4` → 4th generation

→ A GPU VM with an A100 accelerator — built for AI/ML training workloads.

---

## 13. Quick-reference cheat sheet

```
Standard_<Family><Vendor?><vCPU>[-ActiveCores]<Features?>[_<GPU>]_v<Version>

Family:    A, B, D, E, F, G, H, L, M, N (NC/ND/NV)
Vendor:    (blank)=Intel, a=AMD, p=ARM
Features:  d=local disk, s=premium storage, i=isolated,
           l=low memory, m=high memory, C=confidential, t=tiny disk
Constrained cores: "16-8" = 16-vCPU hardware, only 8 active
Version:   v1 (implicit) through the current generation
```

---

## 14. Sources & further reading

- Microsoft Learn — [Virtual machine sizes overview](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview)
- Microsoft Learn — [Azure VM sizes naming conventions](https://learn.microsoft.com/en-us/azure/virtual-machines/vm-naming-conventions)

*Note: Azure occasionally introduces new families, feature letters, or generations. If you come across a letter or pattern not covered here, check the official naming conventions page linked above.*
