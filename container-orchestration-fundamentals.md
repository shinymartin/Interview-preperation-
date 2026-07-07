# Container & Kubernetes Fundamentals

A foundational guide to Containers, Docker, and Kubernetes — and how this knowledge maps onto managed cloud services like **AKS** (Azure Kubernetes Service) and **EKS** (Amazon Elastic Kubernetes Service).

---

## Table of Contents

1. [Why This Stack Exists](#1-why-this-stack-exists)
2. [Containers — The Foundation](#2-containers--the-foundation)
3. [Docker — Building & Running Containers](#3-docker--building--running-containers)
4. [Why We Need Orchestration](#4-why-we-need-orchestration)
5. [Kubernetes — The Orchestrator](#5-kubernetes--the-orchestrator)
6. [Kubernetes Core Objects](#6-kubernetes-core-objects)
7. [Kubernetes Architecture Diagram](#7-kubernetes-architecture-diagram)
8. [Managed Kubernetes: AKS, EKS, GKE](#8-managed-kubernetes-aks-eks-gke)
9. [AKS vs EKS — Quick Comparison](#9-aks-vs-eks--quick-comparison)
10. [Glossary](#10-glossary)
11. [Suggested Learning Path](#11-suggested-learning-path)

---

## 1. Why This Stack Exists

Before containers, teams ran apps directly on servers or virtual machines (VMs). This caused two classic problems:

- **"It works on my machine"** — different environments (dev, test, prod) had different OS versions, library versions, or configs, so apps behaved inconsistently.
- **Heavy, slow scaling** — VMs each need a full OS, so spinning up new capacity was slow and resource-hungry.

The industry solved this in layers, each layer solving the limitation of the one below it:

```
Problem: inconsistent environments        → Solved by: Containers
Problem: managing containers by hand      → Solved by: Docker (tooling)
Problem: running many containers reliably → Solved by: Kubernetes (orchestration)
Problem: running/maintaining Kubernetes   → Solved by: Managed services (AKS/EKS/GKE)
```

By the end of this document you should see how each layer solves the problem of the one before it.

---

## 2. Containers — The Foundation

A **container** is a lightweight, standalone package that bundles an application together with everything it needs to run: code, runtime, system libraries, and settings.

### Key idea: isolation without a full OS

Unlike a VM, a container does **not** include its own operating system kernel. All containers on a machine share the host's kernel, but are isolated from each other using two Linux kernel features:

| Feature | What it does |
|---|---|
| **Namespaces** | Give each container its own isolated view of processes, network, filesystem, hostname, etc. (so it "thinks" it's alone on the machine) |
| **cgroups** (control groups) | Limit how much CPU, memory, and disk I/O each container can use |

### Containers vs Virtual Machines

```
 Virtual Machines                     Containers
 ┌─────────┐ ┌─────────┐             ┌─────────┐ ┌─────────┐
 │  App A  │ │  App B  │             │  App A  │ │  App B  │
 ├─────────┤ ├─────────┤             ├─────────┤ ├─────────┤
 │ Guest OS│ │ Guest OS│             │  (libs) │ │  (libs) │
 ├─────────┴─┴─────────┤             ├─────────┴─┴─────────┤
 │      Hypervisor      │             │   Container Engine   │
 ├───────────────────────┤             ├───────────────────────┤
 │      Host OS          │             │      Host OS          │
 ├───────────────────────┤             ├───────────────────────┤
 │      Hardware          │             │      Hardware          │
 └───────────────────────┘             └───────────────────────┘

 Heavy: each app carries a full OS     Light: apps share one OS kernel
 Boots in minutes                      Starts in seconds
```

**Takeaway:** Containers are fast, lightweight, and portable because they skip duplicating the OS.

---

## 3. Docker — Building & Running Containers

Docker didn't invent containers (the Linux kernel features existed before), but it made them **easy to build, share, and run**. Docker is the most common *container runtime and tooling ecosystem*.

### Core Docker concepts

- **Dockerfile** — a text file with step-by-step instructions to build an image (e.g., "start from Ubuntu, install Python, copy my code, run this command").
- **Image** — a read-only, packaged snapshot of an app + its dependencies, built from a Dockerfile. Think of it as a "class" in programming.
- **Container** — a running instance of an image. Think of it as an "object" instantiated from that class.
- **Registry** — a storage/distribution point for images (e.g., Docker Hub, Azure Container Registry, Amazon ECR). You `push` images to a registry and `pull` them elsewhere.
- **Docker Compose** — a tool to define and run multi-container applications (e.g., an app + a database) using a single YAML file. Good for local development, not for production-scale orchestration.

### Typical Docker workflow

```
Dockerfile  --(docker build)-->  Image  --(docker push)-->  Registry
                                    │
                                    └--(docker run)--> Running Container
```

**Takeaway:** Docker is the tool most people use to *create* and *run* individual containers — but it doesn't manage hundreds of them across multiple machines. That's where Kubernetes comes in.

---

## 4. Why We Need Orchestration

Running one container on one laptop is easy. Real-world production systems need to:

- Run **many containers** across **many servers** (a "cluster")
- **Automatically restart** containers that crash
- **Scale** up or down based on traffic
- **Load balance** traffic across container replicas
- Roll out updates **without downtime**
- Manage secrets, configuration, storage, and networking consistently

Doing this manually with plain Docker commands doesn't scale. This category of tool is called a **container orchestrator**. Kubernetes is the dominant one.

---

## 5. Kubernetes — The Orchestrator

**Kubernetes** (often shortened to **K8s** — "K", 8 letters, "s") is an open-source system for automating deployment, scaling, and management of containerized applications across a cluster of machines.

### The core mental model

You describe the **desired state** ("I want 3 replicas of this app running, exposed on port 80") in a YAML file, and Kubernetes continuously works to make the **actual state** match it. If a container dies, Kubernetes notices and starts a new one — you don't have to intervene.

### The two halves of a cluster

| Part | Role |
|---|---|
| **Control Plane** | The "brain" — makes decisions about scheduling, scaling, and healing the cluster |
| **Worker Nodes** | The "muscle" — machines (VMs or physical) that actually run your containers |

---

## 6. Kubernetes Core Objects

These are the building blocks you'll define in YAML files:

| Object | What it is |
|---|---|
| **Pod** | The smallest deployable unit in Kubernetes. Usually wraps a single container (sometimes more), giving it a shared network IP and storage. |
| **Deployment** | Manages a set of identical Pods (replicas), handles rolling updates and self-healing. This is how you normally run stateless apps. |
| **ReplicaSet** | Ensures a specified number of Pod replicas are running at all times (usually managed for you by a Deployment). |
| **Service** | A stable network endpoint (fixed IP/DNS name) that load-balances traffic across a group of Pods — because Pods come and go and their individual IPs change. |
| **Ingress** | Manages external HTTP/HTTPS access into the cluster, with routing rules (e.g., `/api` → service A, `/app` → service B). |
| **ConfigMap** | Stores non-sensitive configuration data (e.g., environment variables) separately from application code. |
| **Secret** | Like a ConfigMap, but for sensitive data (passwords, API keys, tokens), stored more securely. |
| **Namespace** | A way to logically divide a single cluster into virtual sub-clusters (e.g., `dev`, `staging`, `prod`). |
| **Volume / PersistentVolume** | Storage that outlives an individual Pod's lifecycle, for stateful apps like databases. |
| **Node** | A single worker machine (VM or physical server) in the cluster. |

### How they relate

```
Deployment
   └── manages → ReplicaSet
                    └── manages → Pods (1 or more containers each)

Service
   └── load-balances traffic → to a set of Pods (matched by labels)

Ingress
   └── routes external traffic → to Services
```

---

## 7. Kubernetes Architecture Diagram

```
                              ┌───────────────────────────────────────────┐
                              │              CONTROL PLANE                 │
                              │  (the "brain" of the cluster)              │
                              │                                             │
                              │  ┌───────────────┐   ┌──────────────────┐  │
                              │  │  API Server   │   │  Scheduler       │  │
                              │  │ (front door,  │   │ (decides which   │  │
                              │  │ all commands  │   │ node runs which  │  │
                              │  │ go through it)│   │ pod)             │  │
                              │  └───────────────┘   └──────────────────┘  │
                              │  ┌───────────────┐   ┌──────────────────┐  │
                              │  │ etcd          │   │ Controller       │  │
                              │  │ (cluster's    │   │ Manager          │  │
                              │  │ database of   │   │ (keeps actual    │  │
                              │  │ desired state)│   │ state = desired) │  │
                              │  └───────────────┘   └──────────────────┘  │
                              └────────────────────┬────────────────────────┘
                                                    │ manages
                       ┌────────────────────────────┼────────────────────────────┐
                       │                            │                            │
                ┌──────▼──────┐             ┌───────▼─────┐              ┌───────▼─────┐
                │  WORKER NODE │             │ WORKER NODE │              │ WORKER NODE │
                │              │             │             │              │             │
                │ ┌──────────┐ │             │ ┌─────────┐ │              │ ┌─────────┐ │
                │ │ kubelet  │ │             │ │ kubelet │ │              │ │ kubelet │ │
                │ │(node agent)│             │ │         │ │              │ │         │ │
                │ └──────────┘ │             │ └─────────┘ │              │ └─────────┘ │
                │ ┌──────────┐ │             │ ┌─────────┐ │              │ ┌─────────┐ │
                │ │  Pod(s)  │ │             │ │ Pod(s)  │ │              │ │ Pod(s)  │ │
                │ │[container│ │             │ │[container│ │              │ │[container│ │
                │ │ (s)]     │ │             │ │ (s)]     │ │              │ │ (s)]     │ │
                │ └──────────┘ │             │ └─────────┘ │              │ └─────────┘ │
                └──────────────┘             └─────────────┘              └─────────────┘

  Traffic flow:  User → Ingress → Service → Pod (on any node)
```

**In plain words:**
- You send commands (`kubectl apply -f app.yaml`) to the **API Server**.
- The **Scheduler** decides which node has room to run your Pod.
- **etcd** stores the cluster's source of truth (desired state).
- The **Controller Manager** constantly checks: "does reality match what was asked for?" and fixes drift (e.g., restarts a crashed Pod).
- On each **worker node**, the **kubelet** agent talks to the control plane and actually starts/stops containers via the container runtime (Docker, containerd, etc.).

---

## 8. Managed Kubernetes: AKS, EKS, GKE

Running your *own* Kubernetes control plane (installing etcd, API server, managing upgrades, patching, high availability) is a significant operational burden. **Managed Kubernetes services** let cloud providers run the control plane for you, so you just manage your workloads and worker nodes.

| Service | Cloud Provider | Full Name |
|---|---|---|
| **AKS** | Microsoft Azure | Azure Kubernetes Service |
| **EKS** | Amazon Web Services | Elastic Kubernetes Service |
| **GKE** | Google Cloud | Google Kubernetes Engine |

### What the cloud provider takes care of

- Hosting and patching the **control plane** (API server, etcd, scheduler)
- High availability of the control plane across data centers
- Integration with the cloud's native networking, identity (IAM/RBAC), storage, and load balancers
- Simplified cluster creation via CLI/console (`az aks create`, `eksctl create cluster`, etc.)

### What you still manage

- Worker **node pools** (VM sizes, scaling rules, OS patching depending on setup)
- Your Kubernetes objects (Deployments, Services, Ingress, etc.)
- Application-level configuration, secrets, and monitoring
- Cluster access control (who can do what, via RBAC)

### Everything you learned in Sections 5–7 still applies directly

Whether it's AKS, EKS, GKE, or a cluster running on your laptop (e.g., via Minikube or Docker Desktop), you interact with it the same way — using `kubectl` and the same Pod/Deployment/Service/Ingress concepts. The **managed service only changes who runs the control plane** — not the concepts.

---

## 9. AKS vs EKS — Quick Comparison

| Aspect | AKS (Azure) | EKS (AWS) |
|---|---|---|
| Control plane cost | Free (you pay only for worker nodes) | Charged per cluster-hour |
| CLI tooling | `az aks` (Azure CLI) | `eksctl`, AWS CLI, AWS Console |
| Identity integration | Azure Active Directory (Entra ID) | AWS IAM |
| Container registry | Azure Container Registry (ACR) | Amazon Elastic Container Registry (ECR) |
| Networking model | Azure CNI or kubenet | Amazon VPC CNI |
| Node OS options | Ubuntu, Azure Linux | Amazon Linux, Bottlerocket, Ubuntu |
| Autoscaling | Cluster Autoscaler / Node Autoprovisioning | Cluster Autoscaler / Karpenter |

Both are simply "Kubernetes-as-a-service" — the differences are mostly around cloud-native integrations (identity, networking, storage), not Kubernetes itself.

---

## 10. Glossary

| Term | Meaning |
|---|---|
| **Cluster** | A set of machines (nodes) working together, managed as one Kubernetes environment |
| **Node** | A single machine (VM or physical) in the cluster |
| **Pod** | Smallest deployable unit; one or more containers sharing network/storage |
| **kubectl** | The command-line tool used to interact with a Kubernetes cluster |
| **YAML manifest** | A text file describing the desired state of a Kubernetes object |
| **Helm** | A "package manager" for Kubernetes — bundles YAML manifests into reusable, configurable packages called "charts" |
| **CNI** | Container Network Interface — the plugin system Kubernetes uses for networking |
| **RBAC** | Role-Based Access Control — controls who can do what in the cluster |
| **Container Registry** | A place to store and distribute container images (Docker Hub, ACR, ECR) |
| **Ingress Controller** | The actual software (e.g., NGINX) that implements Ingress routing rules |

---

## 11. Suggested Learning Path

1. **Run a container locally** — install Docker, build a simple image from a Dockerfile, run it.
2. **Push/pull from a registry** — practice with Docker Hub or a free-tier ACR/ECR repo.
3. **Local Kubernetes** — try Minikube or Docker Desktop's built-in Kubernetes to write your first Deployment + Service YAML.
4. **Learn `kubectl` basics** — `kubectl get pods`, `kubectl apply -f`, `kubectl logs`, `kubectl describe`.
5. **Move to a managed cluster** — create a small AKS or EKS cluster, deploy the same YAML you used locally, and notice it behaves the same way.
6. **Explore cloud-native integrations** — Ingress with a cloud load balancer, secrets from Key Vault/Secrets Manager, autoscaling.
7. **Add Helm** once you're comfortable with raw YAML, to manage more complex, reusable deployments.

---

*This document is meant as a conceptual foundation. Once these ideas feel solid, cloud-specific tutorials (Microsoft Learn for AKS, AWS documentation for EKS) will make a lot more sense because you'll already understand what problem each piece is solving.*
