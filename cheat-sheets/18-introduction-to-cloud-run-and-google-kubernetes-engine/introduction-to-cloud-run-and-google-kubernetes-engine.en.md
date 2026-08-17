# Module 18 – Introduction to Cloud Run and Google Kubernetes Engine

> This is Module 2 of the "Developing Containerized Applications on Google Cloud" course, following Module 1 (Module 17: Introduction to Containers).

---

# Overview

Three ways to run a container image, in order of decreasing managed-ness:

```text
Cloud Run (fully managed, serverless) → Google Kubernetes Engine (managed, fine-grained control) → Container-Optimized OS (you manage the VM)
```

---

# Cloud Run

**What it is:** A fully managed compute platform that deploys and runs containers directly on Google's scalable infrastructure. Any language works, as long as it compiles into a container image with a **64-bit Linux binary**.

**Developer workflow (3 steps):** write code (must start a server listening for web requests) → build and package into a container image → deploy to Cloud Run (console, gcloud CLI, YAML, or Terraform). You get a unique HTTPS URL; Cloud Run starts containers on demand and dynamically adds/removes them to handle all incoming requests.

**Two workflows:**

| Workflow | How it works |
| --- | --- |
| Container-based | You build the container image yourself (full transparency/flexibility over what's packaged) |
| Source-based | You deploy source code (Go, Node.js, Python, Java, .NET Core, Ruby); Cloud Run uses **Buildpacks** to build and package it into a container image for you |

**HTTPS & networking:** Cloud Run provisions a valid TLS certificate and handles HTTPS termination — your app only needs to listen on **port 8080** (configurable) over plain HTTP; a process in a container has its own private virtual network stack. Cloud Run services run continuously and respond to web requests/events; **jobs** perform work and quit when done.

**Pricing:**

| Model | How it works |
| --- | --- |
| Request-based (default) | Pay only for resources used while handling requests, plus startup/shutdown — nothing while idle |
| Instance-based | Pay for the entire container lifecycle; CPU is always allocated even with no requests — often more economical for steady-state workloads |

Price scales with the vCPUs and memory allocated to the container.

---

# Cloud Run Use Cases

| Use case | Pattern |
| --- | --- |
| REST API / website | Cloud Run service, optionally connected to a database (e.g. PostgreSQL) to persist data |
| Complex public site (e.g. ecommerce) | Add Cloud CDN (performance) and Google Cloud Armor (content-based traffic filtering); backend connects to a relational DB, Redis (sessions), third-party APIs |
| Microservices | Services communicate via REST/gRPC (direct request/response) or **Pub/Sub** (asynchronous, guaranteed delivery) — Pub/Sub integrates via push subscriptions, forwarding messages as authenticated HTTP requests |
| Event processing | Cloud Run integrates with Cloud Storage, Cloud Build, Pub/Sub, Eventarc, and other event sources to build event-driven workflows |
| Scheduled tasks | **Cloud Scheduler** (a fully managed cron scheduler) securely triggers a Cloud Run service on a schedule — running a scheduled job *inside* a container is unreliable, since a container's lifetime is only guaranteed while it's handling requests; the service must finish within the configured request timeout |

---

# High Availability on Cloud Run

| Feature | Detail |
| --- | --- |
| **Revisions** | Every deployment creates a new, **immutable** revision (container image + service configuration — env vars, memory limits, etc.). Split traffic by percentage between revisions to reduce failure impact, roll back to a stable revision, or gradually shift 100% to the new one. |
| **Autoscaling** | Cloud Run adds container instances when all existing ones are busy, and shuts idle ones down as demand drops. Driven by: incoming request rate, **CPU utilization** (target ~60%), the **concurrency** setting (max parallel requests per instance), and min/max instance count. |
| **Regions & zones** | Cloud Run is regional — you pick a region (≥3 zones each; a zone is a single failure domain). For HA, Cloud Run spreads containers across multiple zones in the region. |
| **Global load balancing** | The global external Application Load Balancer exposes one global IP in front of multiple regional Cloud Run services, routing each client to the nearest region — improves availability and reduces worldwide latency. |
| **Portability** | Containers run anywhere by nature; Cloud Run is also API-compatible with **Knative** (open source), implementing the same container runtime contract — so apps can run in Kubernetes-based environments too (e.g. for data sovereignty or to avoid vendor lock-in). |

**Considerations:** autoscaling incurs cost (cap it with max instances), rapid scale-up can overwhelm downstream systems' throughput capacity, and VM-based workloads need a migration plan to move to containers on Cloud Run or GKE.

---

# Google Kubernetes Engine (GKE)

**What it is:** GKE is Google's fully managed **Kubernetes** service — Kubernetes is the open-source container orchestration system (originally designed by Google, now maintained by the **CNCF**) for automating deployment, scaling, and management. GKE manages cluster creation, load balancing, autoscaling, automatic node upgrades/repair, and logging/monitoring via Google Cloud's operations suite.

## Cluster Architecture

| Component | Role |
| --- | --- |
| **Control plane** | Manages everything running on the cluster's nodes: schedules workloads, manages their lifecycle/scaling/upgrades, and network/storage resources. Runs the Kubernetes API server (`kube-apiserver`) — the hub all cluster components talk to, reachable via HTTP/gRPC, `kubectl`, or the console. **Zonal cluster** = single control plane in one zone; **regional cluster** = multiple control plane replicas across multiple zones (more available). |
| **Nodes** | Compute Engine VMs running your containerized workloads. Each runs a container runtime and the Kubernetes node agent (`kubelet`), which talks to the control plane and starts/runs scheduled containers. |
| **Pod** | The smallest deployable unit in Kubernetes — one or more containers sharing storage/network, with a spec for how to run them. Rarely created directly (usually via Deployments or Jobs). Ephemeral: stays on its node until it finishes, is deleted, is evicted, or the node fails. |

## Key Kubernetes Resources

| Resource | Purpose |
| --- | --- |
| **Deployment** | Declaratively creates/manages pods via a **ReplicaSet** (desired replica count); the Deployment Controller reconciles actual state toward desired state at a controlled rate. Defined in YAML: replica count, a selector label, and a pod template with containers. |
| **Service** | A stable network abstraction (fixed IP for its lifetime + load balancing) for a set of pods selected by label — needed because pod IPs change as ephemeral pods are recreated. Types include `LoadBalancer` (externally accessible), `ClusterIP`, `NodePort`. Defined via a `kind: Service` YAML with a selector, type, and `port`/`targetPort`. |
| **Volume** | A directory accessible to all containers in a pod. **Ephemeral** types (ConfigMap, Secret) share the pod's lifetime; **durable** types (PersistentVolume) exist independently and preserve data after the pod terminates. |
| Others | DaemonSet, StatefulSet, Jobs, and custom resource types also exist. |

Kubernetes objects can be configured **imperatively or declaratively**; Kubernetes continuously works to maintain the declared desired state.

## Developing for GKE

- **Cloud Code** — IDE plugins (with Kubernetes/Cloud Run explorers) for creating, deploying, and integrating apps with Google Cloud.
- **`kubectl`** — CLI to manage cluster resources and workloads.
- **CI/CD pipeline:** develop (Cloud Code + a source repo) → build/test (**Cloud Build** rebuilds the image, stores it in **Artifact Registry**, stores build artifacts in Cloud Storage, runs tests) → deploy to a **staging** GKE cluster via **Google Cloud Deploy** → after approval, promote to **production** → manage on GKE → monitor via Google Cloud's operations suite.

---

# Container-Optimized OS

An OS image for Compute Engine VMs, maintained by Google and based on the open-source **Chromium OS** project, optimized for running Docker containers.

| Benefits | Limitations |
| --- | --- |
| Runs containers out of the box (Docker + `cloud-init` pre-installed, no on-host setup) | No package manager included (use CoreOS toolbox for debugging/admin tools) |
| Smaller attack surface | Doesn't support non-containerized applications |
| Locked-down by default (firewall + security settings) | Can't install third-party kernel modules/drivers |
| Automatic weekly updates (just needs a reboot) | Not supported outside Google Cloud |

**Use it when:** you need to run Docker containers with minimal setup, a secure small-footprint OS, or Kubernetes on Compute Engine instances. **Avoid it when:** your app isn't containerized, needs unavailable kernel modules/drivers/packages, or must be fully supported outside Google Cloud.

---

# Module Summary

This module covers three points on a spectrum for running container images: **Cloud Run** (fully managed, serverless, pay-per-use, best for apps serving web requests including microservices, event processing, and scheduled tasks), **Google Kubernetes Engine** (a managed Kubernetes service offering fine-grained control via clusters, control planes/nodes, pods, and declarative resources like Deployments/Services/Volumes), and **Container-Optimized OS** (a minimal, container-focused VM image for when you manage your own Compute Engine instances). Each trades operational control for managed convenience differently.

---

# Key Points

- Cloud Run apps only need to listen on port 8080 over plain HTTP — TLS termination is handled entirely by the Cloud Run proxy.
- The only real constraint for running a container on Cloud Run: it must compile to a 64-bit Linux binary.
- Request-based pricing has no idle cost; instance-based pricing keeps CPU always allocated and can be cheaper for steady-state workloads.
- A container's lifetime is only guaranteed while handling requests — schedule work externally with Cloud Scheduler, not inside the container.
- Cloud Run service revisions are immutable; traffic splitting enables rollback or gradual rollout.
- Autoscaling is driven by request rate, ~60% CPU utilization target, concurrency, and min/max instances — not request volume alone.
- A region has ≥3 zones, each a single failure domain; Cloud Run spreads containers across zones for HA.
- Cloud Run's Knative compatibility makes apps portable to Kubernetes-based environments.
- In GKE, the control plane schedules and manages everything via `kube-apiserver`; nodes (via `kubelet`) actually run the containers.
- A zonal cluster has one control plane in one zone; a regional cluster has multiple control plane replicas across zones (higher availability).
- A pod is the smallest deployable Kubernetes unit, is ephemeral, and is usually created via a Deployment or Job rather than directly.
- Pod IPs change as pods are recreated — a Service's fixed IP and load balancing exist to solve exactly that.
- Volumes are either ephemeral (ConfigMap, Secret — tied to the pod's life) or durable (PersistentVolume — independent, data survives).
- Container-Optimized OS has no package manager, doesn't run non-containerized apps, rejects third-party kernel modules, and only works on Google Cloud.
