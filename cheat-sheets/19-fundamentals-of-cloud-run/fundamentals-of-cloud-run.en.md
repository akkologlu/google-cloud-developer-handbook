# Module 19 – Fundamentals of Cloud Run

> Source material refers back to the "Developing Containerized Applications on Google Cloud" course (modules 17–18) in the past tense, indicating this module belongs to a new, third course focused specifically on Cloud Run. This module goes deeper than the earlier Cloud Run overview: resource model, full container lifecycle, autoscaling internals, and access control.

---

# Overview

```text
Resource model → Container lifecycle → Autoscaling → Access control
```

**Services vs. jobs:** code runs continuously as a **service** (responds to web requests/events) or as a **job** (does work, then quits). Both run in the same environment with the same Google Cloud integrations.

**HTTPS:** Cloud Run provisions a TLS cert and an HTTPS endpoint on a unique `*.run.app` subdomain (custom domains supported); handles/decrypts/forwards requests to your app over HTTP. Supports WebSockets, HTTP/2, gRPC. Your only responsibility: listen on a TCP port and handle HTTP.

**Running containers:** any language works, as long as it compiles to a **64-bit Linux binary** packaged in a container image.

**Jobs in detail:** a job = one or more independent **tasks** run in parallel during a **job execution**; each task runs one container instance. All tasks must succeed for the execution to succeed (configure timeouts/retries). **Array jobs** run multiple identical instances in parallel (e.g. processing many files from Cloud Storage at once).

**Invocation methods:**

| Method | Use case |
| --- | --- |
| HTTPS | Custom REST API, private microservice, HTTP middleware/reverse proxy, packaged web app |
| gRPC | Internal microservice communication; high data loads (up to 7x faster than REST via protocol buffers); simple service definitions; streaming |
| WebSockets | No extra config needed |
| Pub/Sub (push) | Transform data on Cloud Storage upload, process exported ops-suite logs, custom event publishing |
| Cloud Scheduler | Cron-like scheduled service triggers — backups, recurring admin tasks, generating bills |
| Cloud Tasks | Securely schedule async task processing |
| Eventarc | Trigger from Cloud Storage/BigQuery/other Google Cloud events via Cloud Audit Logs |

---

# Resource Model

| Resource | Detail |
| --- | --- |
| **Service** | The main Cloud Run resource. Regional; instances can start in any zone in the region (spread across zones for redundancy). One project can run many services across regions. Each exposes a unique endpoint and autoscales. |
| **Revision** | Created on every deployment of a container image to a service. Consists of a specific container image + environment config (env vars, memory limits, concurrency). **Immutable** — never modified, only superseded. Requests route to the latest healthy revision automatically. |
| **Container instance** | Handles requests to a revision; autoscaled to the number needed. Can handle multiple concurrent requests (see concurrency setting). |
| **Job / Task / Execution** | A job lives in a region; a job execution starts all its tasks; each task runs one container instance; the execution succeeds only if all tasks succeed. |

**Regions & zones:** a region is a specific geographic location (≥3 zones, each a single failure domain); Cloud Run spreads containers across zones within a region for HA.

---

# Container Lifecycle

```text
Starting → Serving requests ⇄ Idle → Shutting down → Stopped
```

| State | What happens |
| --- | --- |
| **Starting** | Cloud Run materializes the container image and starts the app. Four steps: (1) materialize the image into the container's root filesystem, (2) run the entrypoint program, (3) continuously probe port 8080 (configurable; can configure HTTP/TCP/gRPC startup & liveness probes via YAML) until the app accepts TCP connections, (4) forward requests once it does — only open the port when truly ready. |
| **Serving requests** | Container is handling web requests. |
| **Idle** | Entered after 100 ms with no requests. Doesn't serve requests, incurs **no charge**, CPU throttled to nearly zero (app runs very slowly), can be shut down anytime (has a graceful-shutdown hook). Background tasks are unreliable while throttled — use Cloud Tasks instead. Network calls to third parties likely fail. Can set CPU **always allocated** to avoid throttling (useful for background/async work), but then you're billed for the entire instance lifecycle. |
| **Idle → Serving requests** | Instant — CPU is unthrottled and full access returned immediately, no lag. Cloud Run may keep some instances idle up to 15 minutes to handle spikes/minimize cold starts; **minimum instances** keeps a set number warm permanently. |
| **Shutting down** | If the app handles SIGTERM, it gets 10 seconds to clean up (close TCP/DB connections and file descriptors, flush buffers, write a debug log) before removal. If SIGTERM isn't handled, the container is stopped immediately. |
| **Stopped** | Final state. Under normal conditions Cloud Run never stops a container serving requests — only exceptions: the app crashes, or it exceeds its memory limit (default 512 MiB, configurable up to 32 GiB). In-flight requests fail; new ones may wait for a replacement container. |

**Where the image comes from:** on **deploy**, Cloud Run pulls and copies the image from Artifact Registry into its own internal storage (optimized so large images load as fast as small ones). On every container **start**, it pulls from internal storage, not Artifact Registry — this insulates services from Artifact Registry outages or accidental image deletion.

---

# Autoscaling

- An **internal Application Load Balancer** distributes requests to a revision's container instance pool; adds instances when all are busy, removes them (shuts down) as demand drops.
- Instance count is driven by: request rate, **CPU utilization** (~60% target), **max concurrency**, and **min/max instance settings**. Default quota cap: **1,000 instances** (requestable increase).
- **Scale to zero:** with no incoming requests, even the last instance shuts down — no charge while idle. A new instance starts on demand for the next request.
- **Cold start / request queuing:** the first requests after scale-to-zero queue while the first instance starts.
- **Minimum instances:** keeps a set number of instances warm/idle to avoid cold-start latency — those idle instances **do incur cost**. `gcloud run services update my-service --min-instances 3`
- **Maximum instances:** caps total instances for cost control and downstream compatibility (e.g. a DB with a limited concurrent-connection capacity). Default: **100**. Setting it too low limits Cloud Run's ability to serve all traffic. `gcloud run services update my-service --max-instances 3`
- **Maximum concurrency:** max simultaneous requests per instance. Default **80**, max **1,000**. Lower to 1 if each request uses most of the container's CPU/memory, the app can't handle concurrent requests, or it relies on unshareable global state. Higher concurrency raises per-container memory needs and can stress downstream services during spikes. `gcloud run services update my-service --concurrency 1`

---

# Access Control

**Google Cloud is a collection of APIs** — the console, gcloud CLI, Terraform, and client libraries all call them. Deploying a container image (`gcloud run deploy`) is itself an API call to `run.googleapis.com`.

**IAM** authorizes every API call, verifying caller identity and checking permissions — the same mechanism whether you're deploying a revision or your app is publishing to Pub/Sub.

| IAM concept | Definition |
| --- | --- |
| **Policy** | Always attached to a resource; a list of policy bindings |
| **Policy binding** | Binds a member (identity) to a single role; a member can have multiple bindings (multiple roles) |
| **Role** | A set of permissions (e.g. Pub/Sub Publisher → `pubsub.topics.publish`) |

**Default access:** only project Owner/Editor identities can create/update/delete/invoke Cloud Run services and jobs; Owner and **Cloud Run Admin** (`roles/run.admin`) can modify IAM policies on the project or on individual services/jobs.

**Making a service public:** grant the **Cloud Run Invoker** role (`roles/run.invoker`) to the `allUsers` member — via `gcloud run services add-iam-policy-binding my-service --member="allUsers" --role="roles/run.invoker"`, the `--allow-unauthenticated` deploy flag, console, YAML, or Terraform.

**Controlling access:**

| Scope | Command |
| --- | --- |
| Individual service/job | `gcloud run services add-iam-policy-binding` / `gcloud run jobs remove-iam-policy-binding` |
| All services/jobs in a project | `gcloud projects add-iam-policy-binding` (project-level IAM) |

**Network ingress settings** (independent of IAM, can be layered together):

| Setting | Allows |
| --- | --- |
| **All** (default, least restrictive) | Every request, including directly from the internet |
| **Internal** (most restrictive) | Only internal HTTP(S) load balancer, resources allowed by a VPC Service Controls perimeter containing the service, VPC networks in the same project/perimeter, and same-project/perimeter Cloud Tasks/Eventarc/Pub/Sub/Workflows — internet requests can't reach the service at all |
| **Internal and Cloud Load Balancing** | Everything Internal allows, plus the external HTTP(S) load balancer (still not directly from the internet) |

**Serverless VPC Access:** connects a service/job directly to a VPC network (to reach VMs, Memorystore, etc. by internal IP) using internal DNS/IP so traffic never touches the internet. Setup: enable the API → create a connector (region must match the service/job's region; uses a dedicated, unused `/28` subnet or CIDR) → attach it to a VPC network and region → configure the service/job to use it (`--vpc-connector`); restrict further with firewall rules. Internal services should route all egress through the connector.

---

# Module Summary

Building on the earlier Cloud Run overview, this module explains what actually happens underneath: the resource model (service → revision → container instance; job → task → execution), the full container lifecycle from starting to stopped, the internals of autoscaling (load balancer, scale to zero, cold starts, min/max instances, concurrency), and access control at both the identity (IAM) and network (ingress, VPC) layers.

---

# Key Points

- A service is a stable, permanent resource; a revision is an immutable snapshot created on every deploy.
- A job's execution succeeds only if every one of its parallel tasks succeeds.
- Container images are pulled from internal storage on every container start — not from Artifact Registry, which is only touched at deploy time.
- Idle containers are free but CPU-throttled near zero; background work there is unreliable — use Cloud Tasks.
- SIGTERM gives 10 seconds to clean up; without a handler, the container stops instantly.
- Cloud Run never stops a container mid-request under normal conditions — only a crash or exceeding the memory limit (default 512 MiB) does.
- Scale to zero saves money but risks cold starts; minimum instances avoids cold starts but bills for idle time.
- Default max instances is 100 (your config); the platform quota cap is 1,000 (requestable) — different things.
- Default concurrency is 80, max 1,000 — drop to 1 for CPU/memory-heavy requests or unshareable global state.
- Every Cloud Run action is an API call; IAM authorizes it via policies (bindings of member + role) attached to resources.
- Making a service public means granting `roles/run.invoker` to `allUsers`.
- IAM and network ingress settings are independent layers that can be combined.
- Serverless VPC Access routes Cloud Run traffic to internal VPC resources via internal DNS/IP, never touching the public internet.
