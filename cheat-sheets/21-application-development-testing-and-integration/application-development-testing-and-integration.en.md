# Module 21 – Application Development, Testing, and Integration

> Third module of the same course as `deep-dive/19-fundamentals-of-cloud-run/` and `deep-dive/20-service-identity-and-authentication/` (the Cloud Run-focused course). Covers developing/testing locally, deploying and managing revisions, and integrating with other Google Cloud services.

---

# Overview

```text
Development & testing → Managing deployments & revisions → Integrating with Google Cloud services
```

---

# Is Your App a Good Fit for Cloud Run?

Must meet **all** of:

- Serves HTTP/HTTP2/WebSockets/gRPC requests, streams, or events — or executes to completion (a job).
- No local **persistent** file system required (ephemeral local FS or network FS is fine).
- Built to handle multiple instances running simultaneously.
- ≤8 CPU and ≤32 GiB memory per instance.
- Containerized, or written in Go/Java/Node.js/Python/.NET (or otherwise containerizable).

---

# Container Runtime Contract

| Requirement | Detail |
| --- | --- |
| Any language, any base image | Executables must compile for **Linux 64-bit** |
| Supported image formats | Docker Image Manifest V2 (Schema 1/2), OCI |
| As a service | Listen for requests on the correct port; respond within the configured timeout (**max 1 hour**, including startup time) or the request ends with a **504** |
| As a job | Must **not** listen on a port or start a web server; exit `0` on success, non-zero on failure |
| TLS | Don't implement it yourself — Cloud Run terminates TLS for HTTPS/gRPC and proxies as HTTP/1 or gRPC; handle HTTP/2 requests in cleartext format |

---

# Execution Environments

| | First generation | Second generation |
| --- | --- | --- |
| Default for | Services (changeable) | Jobs (**not** changeable) |
| Best for | Fast scale-out, short cold starts, infrequent traffic, <512 MiB memory | Network file system, steady traffic tolerant of slower cold starts, CPU-intensive workloads, unimplemented syscalls causing issues |
| System calls | Emulates most (not all) | Full Linux compatibility (all syscalls, namespaces, cgroups) |
| Extras | — | Faster CPU/network performance, network file system support |

---

# File System and Data Storage

- **In-memory filesystem**: writable, uses the container's allocated memory, **doesn't persist** past instance stop — good for caching, disposable per-request data.
- **Network file system**: use Filestore or self-managed options for standard filesystem semantics that persist beyond instance lifetime — requires the **second generation** execution environment. Cloud Storage can be mounted via **Cloud Storage FUSE**.
- **No filesystem needed**: use cloud data storage client libraries to connect to Firestore, Cloud SQL, Spanner, Cloud Storage, Memorystore, BigQuery.

---

# Local Development

**Cloud Code** — IDE plugins (VS Code, IntelliJ, Cloud Shell) for the full dev cycle: sample templates, a local **Cloud Run emulator** (configure CPU/memory, env vars, Cloud SQL connections), one-click **Deploy to Cloud Run** (build locally or via Cloud Build, push, deploy, live URL shown).

| Local testing tool | How |
| --- | --- |
| Cloud Code | IDE extension + Cloud Run emulator |
| gcloud CLI | Local dev environment; builds from a Dockerfile if present, else Google Cloud's buildpacks; auto-rebuilds on source changes; test at `http://localhost:8080/` |
| Docker | `docker run` with the image URL and listening port; test at `http://localhost:port/` |

---

# Building and Deploying Containers

| Tool | Role |
| --- | --- |
| Docker | Build locally with a Dockerfile (`docker build`), push to a repo (`docker push`) |
| Cloud Build | Build on Google Cloud with a Dockerfile or Google Cloud's buildpacks (`gcloud builds submit`, add `--pack` for buildpacks) |
| Cloud Run | `gcloud run deploy --source` builds from source (Dockerfile if present, else buildpacks), uploads the image, and deploys — or build locally with buildpacks via the `pack` command |

**Deployment requires Artifact Registry (or Docker Hub) access** — Google recommends Artifact Registry; other registries need an Artifact Registry remote repository. Push unsupported-registry images with `docker push` first.

**Artifact Registry**: universal package manager (Docker, NPM, Maven, PyPI repositories); recommended registry for Google Cloud; integrates with Cloud Build. Create a **Docker repository** to host container images — each image gets a unique URL (e.g. `us-central1-docker.pkg.dev/${PROJECT_ID}/my-repo/my-image`).

**Pull behavior**: Cloud Run pulls the image from Artifact Registry and **copies it into its own internal storage** for fast, reliable startup — large images load as fast as small ones, and accidentally deleting the image from Artifact Registry doesn't break the running service.

**Creating/updating a service**: requires Owner, Editor, or both Cloud Run Admin + Service Account User roles (or an equivalent custom role). First deploy creates the service **and** its first revision — one container image per service.

---

# Managing Revisions

**Update flow (5 steps):** modify code → build/package image → push to Artifact Registry → redeploy to the service → wait for Cloud Run to deploy.

**Service configuration (8 pieces)** — changing *any* of these creates a new revision, even with no image change:

1. Container image URL
2. Container entrypoint and arguments
3. Secrets and environment variables
4. Request timeout
5. Concurrency
6. CPU/memory limits
7. Scaling boundaries (min/max instances)
8. Google Cloud configuration (service account, connectors)

Subsequent revisions inherit these settings automatically unless explicitly changed.

**Revisions are immutable** — each deploy creates a new, immutable copy of the service resource (container image + configuration); only new revisions can be added.

**Traffic behavior on a new revision:**
1. Cloud Run scales the new revision up to match the current revision's capacity, waiting for its instances to finish starting — the old revision keeps serving in the meantime.
2. Once ready, traffic routes to the new revision; both revisions autoscale independently; the old one idles and eventually scales to zero.
3. For a gradual rollout, deploy with `--no-traffic` (0% initially), then increase the percentage incrementally.

| Feature | Purpose |
| --- | --- |
| **Pinning** | Set a revision's traffic percentage to 100 to decouple deployment from traffic migration — useful for rollback or pre-migration testing |
| **Tagging** | Give a revision a traffic-free test URL (`https://TAG---service-xyz.a.run.app`) to vet it before sending traffic |
| **Splitting** | Route a configurable percentage of requests to each revision (console/gcloud/YAML/Terraform); traffic changes aren't instantaneous — in-flight requests finish, possibly on either revision during the transition |
| **Session affinity** | Best-effort routing of the same client's requests to the same revision's container instance (off by default) |

---

# Integrating with Google Cloud Services

**Client libraries**: authenticate transparently via the service's service account. The **built-in** account has the broad **Project Editor** role — restrict access with a **per-service identity** (minimally-permissioned service account), e.g. Firestore User for read-only Firestore access.

**Memorystore (Redis)**: connect via **Serverless VPC Access** — determine the Redis instance's authorized VPC network, create a connector in the same region as the service, attach the connector to that network, then deploy with `--vpc-connector` and `REDISHOST`/`REDISPORT` env vars.

**Cloud Run Integrations**: a simplified console/CLI flow that automates connecting a service to Memorystore (or mapping a custom domain) — auto-creates the Redis cache, a new revision, and the networking/env var config.

**Pub/Sub trigger**: a push subscription delivers messages to the service endpoint as HTTP requests (endpoint can stay private, protected by IAM). The service must acknowledge within **600 seconds** or Pub/Sub redelivers. Setup: create a topic → add code to extract the message and return 200/204 (success, acked) or 400/500 (error, retried) → create a service account with **Cloud Run Invoker** → create a push subscription tied to that account, pointing at the service URL.

**Cloud SQL**:

| Path | Mechanism |
| --- | --- |
| Public IP (default) | Service account needs Cloud SQL Client or Cloud SQL Admin; deploy/update with `--add-cloudsql-instances=<connection-name>`; Cloud Run connects via the **Cloud SQL Auth proxy** (network sockets or language-specific connectors) with encryption |
| Private IP | Route all egress through a **Serverless VPC Access** connector |

Best practices: store credentials in **Secret Manager**, pass as env vars or a mounted volume; use a **connection pool** to auto-reconnect and cap connections (Cloud Run services are limited to **100 connections per service** to a Cloud SQL database).

---

# Module Summary

This module bridges "writing code" and "running a reliable production service": whether an app fits Cloud Run, the container runtime contract it must satisfy, execution environments and local testing options, how images move from Artifact Registry into Cloud Run's internal storage, how revisions are versioned and traffic controlled between them (pinning, tagging, splitting), and how a service safely connects to Pub/Sub, Memorystore, and Cloud SQL.

---

# Key Points

- Cloud Run fitness requires meeting all five criteria at once.
- Jobs must not listen on a port (they exit with a code); services must respond within the request timeout.
- Never implement TLS yourself — Cloud Run terminates it transparently.
- First generation is the (changeable) service default; second generation is the (unchangeable) job default and the only option supporting network file systems.
- The in-memory filesystem is ephemeral; use Filestore/network FS or client libraries for persistence.
- Cloud Run serves images from its own internal storage, not Artifact Registry directly, on every start.
- Any configuration change — not just a new image — creates a new immutable revision.
- Traffic to a new revision is deliberate: it scales up first, then migrates (optionally gradually via `--no-traffic`), while in-flight requests are never dropped.
- Pinning, tagging, and splitting are three distinct traffic-control tools for rollback, pre-release testing, and gradual rollout respectively.
- The built-in service account (Editor role) is overly broad — use a minimally-permissioned per-service identity instead.
- Memorystore and private-IP Cloud SQL both require Serverless VPC Access; public-IP Cloud SQL uses the Cloud SQL Auth proxy.
- Pub/Sub push endpoints don't need to be public — protect them with IAM and a Cloud Run Invoker-granted service account.
- Cap Cloud SQL connections with a connection pool (100/service limit) and keep credentials in Secret Manager.
