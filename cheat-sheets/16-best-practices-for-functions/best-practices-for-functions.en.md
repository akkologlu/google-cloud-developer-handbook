# Module 16 – Best Practices for Cloud Run Functions

> This is Module 5 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 4 (Module 15: Integrating with Cloud Databases).

---

# Overview

Four practical areas for turning a working function into a production-ready one:

```text
Implementation practices → Performance & networking → Retry on failure → Configuration & scaling
```

---

# Implementation Best Practices

| Practice | Why |
| --- | --- |
| Write idempotent functions | Same result on repeated calls — makes it safe to retry a partially failed invocation |
| HTTP functions must always return an HTTP response | Otherwise the function may run until timeout, and you're billed the whole time |
| No background activity after invocation ends | CPU isn't accessible post-termination; a later invocation on the same environment can cause that background work to resume and interfere |
| Ensure async operations finish before terminating | Prevents leaked work from bleeding into subsequent invocations |
| Explicitly delete files written to the temp directory | `/tmp` is an in-memory filesystem — leftover files consume memory and can eventually force a cold start |
| Develop and test locally first | Deploy → wait → check logs is slow; local testing is much faster |
| Use a platform compatible with Cloud Run functions' open-source abstraction layers for data locality needs | Lets you comply with geographic/network boundary restrictions the standard environment can't meet |
| Don't use `process.exit()` (Node.js) / `sys.exit()` (Python) | Causes unexpected behavior — return implicitly/explicitly from event-driven functions, return HTTP responses from HTTP functions |
| Don't throw uncaught exceptions | They force cold starts on **future** invocations, not just the current one |
| Always handle runtime errors/exceptions in code | Unhandled exceptions can terminate the function and cause future cold starts |

**Error Reporting & logging:**

- Runtime exceptions are sent to **Error Reporting** — aggregate, view, and get notified in the Google Cloud console.
- Simple runtime logging is included by default; anything written to `stdout`/`stderr` appears automatically in the console.
- HTTP functions should report the error and respond with an appropriate HTTP status code; event-driven functions should report and return an error message.

---

# Performance and Networking

**Cold start:** creates and initializes a function's execution environment; dependencies imported by the function are loaded during this time, adding to invocation latency.

| Technique | Effect |
| --- | --- |
| Don't load unused dependencies | Reduces cold start latency and deploy time |
| Cache expensive objects (API clients, network connections) in global scope | Execution environments are often recycled — global-scope values persist across invocations on the same instance, avoiding recomputation |
| Lazily initialize global variables not used on every code path | Initializing globals always adds cold-start latency; lazy init avoids paying that cost when the path isn't taken |
| Set a minimum number of instances | Keeps instances warm and ready, reducing cold starts entirely |
| Create persistent HTTP connections and cache them in global scope | Reduces CPU spent per invocation establishing new connections, and reduces the risk of exhausting your connection quota |
| Create Google API service client objects in global scope | Avoids unnecessary connections and DNS queries per invocation |
| Use Serverless VPC Access connectors for VPC-internal traffic | Traffic to internal resources uses internal DNS/IP and is never exposed to the internet |

---

# Retry on Failure

| Property | Detail |
| --- | --- |
| Availability | **Event-driven functions only** — not available for HTTP functions |
| Default | **Disabled** |
| Enable | `--retry` flag with `gcloud functions deploy`, or "Retry on failure" in the console |
| Disable | Redeploy without `--retry`, or clear the console option |
| Retry window | Up to **7 days by default**, until success or the max retry period elapses |

**Common failure causes:** an unhandled exception from a bug, an unreachable/timed-out service endpoint, an intentionally thrown unhandled exception, or (Node.js) a rejected promise / non-null value passed to a callback — the function stops and the event is discarded.

**Using retry safely:**

- Best suited for **intermittent/transient** failures (connection failures, timeouts) that are likely to resolve on their own.
- Fix bugs that cause failures **through testing before** enabling retry — a buggy function retries continuously without ever succeeding.
- Handle exceptions that should **not** trigger a retry.
- Include an **end condition** (e.g., discarding events older than a timestamp) to prevent infinite retry loops when failures are persistent.

---

# IAM and Configuration Best Practices

| Area | Practice |
| --- | --- |
| Least privilege | Limit function access to the minimum users/service accounts and minimum permissions needed |
| Function-to-function access | Restrict each function to calling only the specific subset of functions it legitimately needs (e.g., a login function may call a user-profiles function, but probably not a search function) |
| Runtime service account | Default service account is used unless specified — for production, assign a **dedicated user-managed service account** with a minimal permission set |
| Memory/CPU | Provision differing memory amounts; allocated memory determines allocated CPU; set via `--memory` or the console |
| Timeout | Set slightly higher than actual execution time to prevent premature timeouts; set via `--timeout` or the console |

**Concurrency:**

- By default, an instance handles **one request at a time**.
- Concurrency can be enabled (via the function's underlying Cloud Run service) with a per-function concurrency value — the max number of concurrent requests a single instance can handle.
- Reduces cold starts by letting an already-warmed instance absorb extra requests.
- Function code must be **safe for concurrent execution** — Cloud Run functions provides **no isolation** between concurrent requests on the same instance.

---

# Scaling, Revisions, and Traffic Splitting

- **Scaling** creates new instances based on request volume; controlled per-function via **minimum and maximum instance counts** set at deployment — each function scales independently.
- **Minimum instances** reduce cold starts and latency; **maximum instances** limit load on throughput-constrained downstream resources (e.g., databases).
- Instance limits can be **temporarily exceeded**: briefly during traffic spikes, and after a deployment (since limits are per-revision) so existing requests keep being served by old instances while new requests go to new ones.
- Every deployment creates a new, **immutable revision** of the function and its underlying Cloud Run service — to change a function, redeploy it.
- By default, **all traffic routes to the latest revision**; a custom traffic configuration can split traffic across revisions or roll back to a prior one.

---

# Module Summary

Turning a working Cloud Run function into a production-ready one spans four areas: writing idempotent, properly-terminating code that always responds, cleans up temp files and async work, and never throws uncaught exceptions or calls `process.exit()`/`sys.exit()`; optimizing performance by minimizing cold starts (fewer dependencies, global-scope caching of expensive objects, lazy init of rarely-used globals, minimum instances) and reusing network connections (persistent HTTP connections, Google API clients, Serverless VPC Access); enabling retry safely for event-driven functions only — fixing bugs before enabling it, and adding an end condition to avoid infinite retry loops on persistent failures; and configuring IAM (least privilege, restricted function-to-function access, dedicated runtime service accounts), memory/CPU/timeout, concurrency (with awareness that Cloud Run functions provides no isolation), and scaling/revisions/traffic splitting for controlled rollouts.

---

# Key Points

- Idempotent functions make retries safe; HTTP functions must always return a response or risk running (and billing) until timeout.
- No background work should survive invocation termination; ensure async operations finish first, and explicitly delete temp files to avoid memory exhaustion and cold starts.
- Don't call `process.exit()`/`sys.exit()`; don't throw uncaught exceptions — they force cold starts on future invocations too. Report errors to Error Reporting; `stdout`/`stderr` logging works out of the box.
- Cold starts load dependencies and initialize the execution environment; reduce them by trimming unused dependencies, caching expensive objects in global scope, lazily initializing rarely-used globals, and setting a minimum instance count.
- Reuse persistent HTTP connections and Google API clients via global scope; use Serverless VPC Access for internal-only traffic.
- Automatic retry exists only for event-driven functions, is off by default, and retries for up to 7 days — use it for transient failures, fix bugs before enabling it, and add an end condition to avoid infinite loops.
- Apply least privilege to function access and function-to-function calls; use a dedicated runtime service account in production instead of the default.
- Memory allocation determines CPU allocation; set timeout slightly above actual execution time.
- Concurrency lets one instance serve multiple requests to reduce cold starts, but requires code safe for concurrent execution since there's no isolation between requests.
- Minimum/maximum instance counts control scaling (and can be temporarily exceeded); every deployment creates an immutable revision, and traffic defaults to the latest one unless split or rolled back via custom configuration.
