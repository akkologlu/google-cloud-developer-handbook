# Module 12 – Introduction to Cloud Run Functions

> This is Module 1 of the "Developing Applications with Cloud Run Functions on Google Cloud" course. This is a new course, separate from the "Service Orchestration and Choreography on Google Cloud" course covered in modules 09–11.

---

# Overview

Cloud Run functions is Google Cloud's fully managed, serverless functions-as-a-service (FaaS) platform: you write single-purpose function code, deploy it, and Cloud Run functions handles provisioning, scaling, and running it — no servers to manage.

```text
Develop function code → Deploy (console / gcloud / Cloud Build / Cloud Code) → Set up a trigger (HTTP or event)
```

There are two versions:

| | Cloud Run Functions (2nd gen) | Cloud Run Functions (1st gen) |
| --- | --- | --- |
| Former name | Cloud Functions (2nd generation) | Cloud Functions (1st generation) |
| Deployed as | A Cloud Run service under the hood | The original functions runtime |
| Triggered via | Eventarc and Pub/Sub | A more limited set of event triggers |
| Configurability | Full — this is the current, recommended path | Limited |

Cloud Run itself is a managed compute platform for running containers on Google's infrastructure; Cloud Run functions builds on it — a function is simple code that serves a single piece of functionality, triggered by an event from cloud infrastructure or other services (e.g., a file upload to Cloud Storage, an incoming Pub/Sub message).

---

# Features and Benefits

**Features:**

- Local development and testing before deploying.
- Seamless authentication to other Google Cloud services via service accounts.
- HTTP and event-driven execution.
- Built-in integration with Cloud SQL, Bigtable, Spanner, and Firestore.
- Portability — a function runs in any standard runtime environment for its language.

**Benefits:**

- Augments and extends existing cloud services with programming logic.
- Serverless — Google fully manages the software and infrastructure; no servers to patch or frameworks to update.
- Autoscaling — resources are automatically provisioned in response to events, from a few invocations a day to millions, with no extra work.
- Observable — integrated with Cloud Logging and Cloud Monitoring for diagnosis.
- Pay-as-you-go pricing based on number of invocations, compute time, and outbound network data transfer.

**Underlying capabilities:**

| Capability | Limit / behavior |
| --- | --- |
| Memory / CPU per instance | Up to 32 GB RAM, up to 4 vCPUs |
| Concurrency | Up to 1000 concurrent requests per instance (reduces cold starts, improves scaling latency) |
| Timeout | Up to 60 minutes for HTTP functions, up to 10 minutes for event-driven functions |
| Revisions | A new revision is created on every deploy — supports traffic splitting and rollback |
| Portability | Can be moved to Cloud Run or Kubernetes, since it's built on Cloud Run's container platform |

> **Exam trap:** Cloud Run functions is not a separate product with its own execution model — it *is* Cloud Run underneath (2nd gen), just with a function-oriented deployment and triggering model layered on top. That's exactly why it inherits Cloud Run's revision/rollback/traffic-splitting behavior and can be migrated to plain Cloud Run or Kubernetes.

---

# HTTP Functions vs. Event-Driven Functions

Cloud Run functions come in two types:

| | HTTP functions | Event-driven functions |
| --- | --- | --- |
| Trigger | An HTTP(S) request | An event from your cloud environment (e.g., new Pub/Sub message, file deleted from Cloud Storage) |
| Assigned a URL? | Yes — the function receives requests at that URL | No |
| Default authentication | Required (can opt into unauthenticated) | N/A — governed by the event trigger source |
| Typical use case | Webhooks, APIs handling requests from other services | Reacting automatically to infrastructure changes |
| Implementation | Write an HTTP handler registered with the Functions Framework for your language; process the request, send a response; any background tasks (threads, promises) must finish before the response is sent | Implemented as a **CloudEvent function** or a **Background function** |

Event-driven functions use **event triggers**, supported for Pub/Sub, Cloud Storage, Firestore, Firebase, and Eventarc sources — and, via Eventarc, any Google Cloud service that supports Pub/Sub as an event bus.

| | CloudEvent functions | Background functions |
| --- | --- | --- |
| Basis | The CloudEvents industry-standard specification | Older-style, receives event data based on event type |
| Registered with | Functions Framework (an open-source library wrapping user functions in a persistent HTTP application) | — |
| Supported by | Cloud Run functions, all language runtimes; Cloud Run functions 1st gen with .NET, Ruby, PHP | Cloud Run functions 1st gen only, with Node.js, Python, Go, Java |

> **Exam trap:** Background functions are not an alternative style you'd freely choose today — they're the *older* event-driven implementation tied specifically to Cloud Run functions 1st gen on a handful of runtimes. CloudEvent functions (built on the Functions Framework and the CloudEvents standard) are what current, full-featured Cloud Run functions (2nd gen) uses across all supported languages.

---

# Use Cases

| Use case | How it works |
| --- | --- |
| Data processing | React to Cloud Storage events on files/objects: validate, transform, process images/video for downstream use |
| Webhooks and APIs | HTTP functions handle webhook/API calls from external systems |
| Mobile backend | Respond to events triggered by Firebase and Firestore |
| IoT | Process and store device data streamed into Pub/Sub |

---

# Language Runtimes and Source Structure

Each language runtime has specific conventions for where Cloud Run functions looks for your function's source code:

| Language | Default entry file | Dependency/config file |
| --- | --- | --- |
| Node.js | `index.js` at the root (override via `main` in `package.json`) | `package.json` — includes the Functions Framework for Node.js as a dependency |
| Python | `main.py` at the root | `requirements.txt` — includes the Functions Framework for Python |
| Go | A Go package at the project root | `go.mod` — includes the Functions Framework for Go |

The **function entry point** is the function (or class, depending on language) executed when the function is invoked; it must be defined in your main file or root package, and you specify it explicitly at deploy time.

**Region selection:** primary considerations are latency and availability. Generally pick the region closest to your users, but also weigh the location of the other Google Cloud services your app depends on — services spread across multiple locations affect both latency and pricing. Cloud Run functions and Cloud Run functions 1st gen have different regional availability.

---

# Building and Deploying

**IAM requirements:** to deploy, a user needs the **Cloud Functions Developer** IAM role (or an equivalent), plus the **Service Account User** IAM role on the Cloud Run functions runtime service account.

**Deployment methods:** Google Cloud console, Cloud Build, Cloud Code (lets you create, deploy, and invoke functions directly from your IDE), or the gcloud CLI.

Key `gcloud functions deploy` flags:

| Flag | Purpose |
| --- | --- |
| `--gen2` | Deploy to Cloud Run functions (2nd gen) |
| `--region` | Region to deploy into |
| `--runtime` | Language runtime |
| `--source` | Location of the source code |
| `--entry-point` | The function/class name that is the entry point |
| Trigger flags | Type and configuration of the trigger |

**Source location options (`--source`):**

| Source | Details |
| --- | --- |
| Local machine | A local filesystem path to the source root; optionally use `--stage-bucket` to upload to Cloud Storage as part of deployment; exclude files with `.gcloudignore` |
| Cloud Storage | A path to a bucket containing the source packaged as a zip, with source files at the zip root; the deploying account (1st gen) or the Cloud Run functions service agent (2nd gen) needs read access to the bucket |
| Source repository | A reference to a revision (`revisions/<name>`) and, optionally, a subdirectory (`paths/<source_directory_path>`); the service agent needs the **Source Repository Reader** (`roles/source.reader`) role; also enables deploying from a linked GitHub or Bitbucket repo |

You can also write and deploy directly from the Google Cloud console's inline editor (file list on the left, editor on the right).

---

# The Build Pipeline

```text
Your source code → Cloud Storage → Cloud Build → container image → Artifact Registry → Cloud Run functions runs it
```

- Deployed source is stored in a Cloud Storage bucket.
- **Cloud Build** automatically builds the source into a container image and pushes it to **Artifact Registry** — this is fully automatic and requires no direct input from you.
- Cloud Run functions accesses that image whenever it needs to run your function.
- The Cloud Build API must be enabled on your project; build resources run in your own project, and all build logs are available through Cloud Logging.
- **Artifact Registry** stores and manages the resulting container images (and other software artifacts) in private repositories, integrating with Cloud Build to store the packages it produces.

---

# Module Summary

Cloud Run functions is Google Cloud's fully managed FaaS platform, built on top of Cloud Run: you write single-purpose functions, deploy them (console, gcloud, Cloud Build, or Cloud Code), and attach a trigger — either HTTP (with a URL, authenticated by default) or event-driven (CloudEvent functions on the current 2nd-gen platform across all languages, or older Background functions on 1st gen for a handful of runtimes). Functions scale automatically from near-zero to millions of invocations, integrate with Cloud Logging/Monitoring, and are billed on invocations, compute time, and egress. Deployment always follows the same pipeline regardless of entry point (console, CLI, or source repo): your source lands in Cloud Storage, Cloud Build turns it into a container image, that image is pushed to Artifact Registry, and Cloud Run functions runs it from there — which is also why a function can, if needed, be migrated straight to plain Cloud Run or Kubernetes.

---

# Key Points

- Cloud Run functions (2nd gen, formerly Cloud Functions 2nd gen) deploys functions as Cloud Run services, triggered via Eventarc/Pub/Sub; Cloud Run functions 1st gen is the older, more limited version.
- HTTP functions get a URL and require auth by default; event-driven functions react to triggers from Pub/Sub, Cloud Storage, Firestore, Firebase, or Eventarc (90+ sources).
- CloudEvent functions (Functions Framework, CloudEvents standard, all languages on 2nd gen) are the current event-driven implementation; Background functions are the older 1st-gen-only style.
- Source-code location conventions are runtime-specific: `index.js` (Node.js), `main.py` (Python), package root (Go) — each with its own dependency manifest that also pulls in the Functions Framework.
- Deploying requires the Cloud Functions Developer role plus Service Account User on the runtime service account; source can come from a local path, a Cloud Storage zip, or a source repository (including linked GitHub/Bitbucket).
- Every deployment follows the same build pipeline: source → Cloud Storage → Cloud Build → container image → Artifact Registry → run.
