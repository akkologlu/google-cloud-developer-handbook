# Module 11 – Choreography and Orchestration

> This is Module 3 of the "Service Orchestration and Choreography on Google Cloud" course (Module 1: [Introduction to Microservices](../09-introduction-to-microservices/introduction-to-microservices.en.md), Module 2: [Event-Driven Applications](../10-event-driven-applications/event-driven-applications.en.md)). This is the first module in the course to name concrete Google Cloud products: Pub/Sub, Eventarc, Workflows, Cloud Tasks, and Cloud Scheduler — together, Google Cloud's "Application Integration" toolbox.

---

# Overview

Coordinating communication between microservices is one of the hardest parts of a microservices architecture — some of the complexity that used to live inside a monolith shifts into how services talk to each other.

There are two basic coordination patterns: **service choreography** and **service orchestration**.

```text
Choreography (dance)                Orchestration (orchestra)

Service A ──event──┐                        ┌──→ Service A
Service B ──event──┼─→ (no conductor)        Orchestrator ─┼──→ Service B
Service C ──event──┘                        └──→ Service C
```

---

# Service Choreography vs. Service Orchestration

| | Service Choreography | Service Orchestration |
| --- | --- | --- |
| Analogy | A choreographed dance — each dancer knows their part and performs it independently | An orchestra — a conductor actively synchronizes every musician |
| Control | Distributed — each service reacts to events on its own | Centralized — one orchestrator controls all interactions |
| Coupling | Loosely coupled; services created, changed, scaled separately | Also loosely coupled; services don't need to know about each other |
| Source of truth | None — business logic is distributed across services | Central — the orchestrator/workflow defines the process |
| Main strength | Decentralized control across teams/orgs; easy to leverage GCP service events | High-level view of the process; easier troubleshooting and tracking |
| Main weakness | No central source of truth — harder to understand the overall flow | Single point of failure — if the orchestrator is down, nothing runs |

> Both patterns produce loosely coupled, independently deployable services. The real difference is *where the process logic lives*: scattered across services (choreography) or centralized in one place (orchestration).

---

# Pub/Sub

Pub/Sub is a fully managed, real-time messaging service for sending and receiving messages between independent services or applications.

```text
Publisher → Topic → [Subscriber 1's queue, Subscriber 2's queue, ...] → Subscribers
```

- A **publisher** sends a message to a **topic**. The message is stored in Pub/Sub and delivered to each subscriber's own queue for that topic.
- A **pull subscription** has the subscriber occasionally poll for new messages.
- A **push subscription** has Pub/Sub automatically send the message to a configured endpoint.
- A subscriber **acknowledges** a message to remove it from its queue. Because the message is only removed *after* it's handled, Pub/Sub guarantees **at-least-once delivery**.

**Example — image resizing:** a Cloud Storage bucket is configured to publish to an "image uploads" topic whenever a new file lands. Two subscribers react: a Resizing Service (resizes the image, writes to another bucket, updates Firestore) and an Upload Confirm app (updates Firestore to record the successful upload). Each Cloud Run service stays simple — Pub/Sub is what kicks off processing.

---

# Eventarc

Eventarc is Google Cloud's fully managed eventing system for building event-driven applications.

- Many GCP products can send events to Eventarc **directly**; others (without direct support) are captured via **Cloud Audit Logs** entries — you don't have to write log-parsing code.
- Third-party providers can create events via the Eventarc API. Pub/Sub topics can also be used as an event source.
- An **event trigger** is a rule-based filter that routes a specific event type from a specific source to a specific target.
- Eventarc uses **Pub/Sub as its transport layer** (for reliability and observability) and automatically manages the underlying topics/subscriptions — your app only needs to accept the HTTP requests Eventarc sends. You never touch Pub/Sub directly.
- Events are delivered in the standard CNCF **CloudEvents** format regardless of source — a common metadata format with SDKs in Python, JavaScript, Java, Go, C#, Ruby, and PHP, so your event-handling code doesn't change based on where an event came from.

| | Pub/Sub (direct) | Eventarc |
| --- | --- | --- |
| Event sources | You wire up publishers yourself | Many built-in GCP + third-party sources, no custom ingestion code |
| Topic/subscription management | You manage it | Automatically created and managed for you |
| Event format | Whatever the publisher chose | Standardized CloudEvents format, with SDKs |
| Interface | Write ingestion code | Simple rule-based trigger: source, filter, destination |

> **Exam trap:** Pub/Sub and Eventarc are not competitors. Eventarc is an abstraction layer built *on top of* Pub/Sub — use Eventarc when you want a standardized event format and built-in support for many event sources without hand-rolling ingestion and topic management.

---

# Workflows

Workflows is Google Cloud's fully managed orchestration platform — the concrete implementation of the service-orchestration pattern.

- You design and deploy workflows that orchestrate GCP services and API calls into **stateful, automated processes**.
- A workflow is a **central source of truth** for the application's flow.
- Every execution is logged and observable, making troubleshooting easier.
- A workflow can hold state, retry, poll, or wait for **up to a year** — enabling genuinely long-running business processes.
- Workflows handles retries and exceptions thrown by APIs, improving overall reliability.

**Example — order processing:** check Firestore inventory → lock available items → branch on an "out of stock" flag (used again later in the flow) → prepare a confirmation message (Cloud Run function) or request more inventory from suppliers (Cloud Run service) → update Firestore → email the customer → optionally post to Slack if anything was out of stock. Each execution is logged for tracing a single transaction.

Workflows is a strong fit for chaining HTTP-based microservices into durable, stateful flows, and for batch/set-of-items processing where robust error handling matters.

---

# Choosing Choreography vs. Orchestration

With **choreography**, the receiving service controls what happens next. When Service A emits an event, it has no idea whether — or how many — other services will act on it; that's not A's responsibility. A consumer (Service B) must understand the event's format and may know it came from Service A, without being tightly coupled to it. Using Eventarc, most GCP or custom services can act as producers.

The order-processing example *could* be built with choreography: each service emits a "ready for next step" event, and the in-stock/out-of-stock decision is made by whichever service owns that step. But an enterprise order system needs visibility, error handling, and retries — and those are hard to get right in a purely event-based design. How do you troubleshoot it? What happens if the process aborts between locking inventory and updating it? How do you guarantee a supplier request actually succeeded?

Orchestration answers those questions directly: each execution is tracked separately, the ordering logic lives in one place, and retries/error handling are built in.

| Question | Favors |
| --- | --- |
| Do you need to manage a complex process centrally, with strong visibility, retries, and error handling? | Orchestration |
| Do the services span different teams or organizations that each manage their own piece independently? | Choreography |
| Do you mainly want to react to events already emitted by GCP services? | Choreography |
| Do you need a single traceable execution record for troubleshooting? | Orchestration |

> **Exam trap:** Orchestration isn't automatically "better" just because it's easier to troubleshoot — a shared central orchestrator requires shared central control, which breaks down across independently managed teams/organizations. Choreography's decentralization is the actual advantage there, not a limitation to work around.

In practice, the two combine: e.g., Order, Fulfillment, and Marketing services can each be implemented internally with **Workflows** (orchestration), while **Eventarc** carries event triggers *between* those services and reacts to new files landing in a Cloud Storage bucket (choreography across the boundaries).

---

# Cloud Tasks vs. Pub/Sub

Cloud Tasks manages the execution, dispatch, and delivery of large numbers of distributed tasks — each task is dispatched to a specific HTTP service.

- Queues support a configurable max dispatch rate, max concurrent dispatches, max retry attempts, and delay between retries.
- A task can be scheduled for dispatch at a specific future time.
- Delivery is **at-least-once**, with duplicate tasks eliminated.
- Authenticated calls can automatically attach a token tied to the creating application's service account.

Cloud Tasks and Pub/Sub are conceptually similar (both do message passing / async integration) but solve different problems:

| | Cloud Tasks | Pub/Sub |
| --- | --- | --- |
| Invocation | **Explicit** — the creator retains full control over execution and destination | **Implicit** — publishing a message triggers whichever subscribers exist |
| Destination | A specific, known endpoint chosen by the creator | Unknown to the publisher — any current subscriber |
| Best for | Asynchronously running one specific service, optionally on your own schedule (e.g., offloading slow background work from the main request path) | Event-based architectures where receiving services react to events from other services |

> **Exam trap:** Don't treat Cloud Tasks and Pub/Sub as interchangeable just because both are asynchronous messaging. The deciding question is *does the sender need to control exactly which service handles this and when* (Cloud Tasks) *or does the sender just need to announce that something happened, indifferent to who's listening* (Pub/Sub)?

---

# Cloud Scheduler

Cloud Scheduler is a fully managed, enterprise-grade cron job scheduler, managed from a single dashboard.

- Jobs use the standard **Unix cron format**: 5 space-separated fields — minute, hour, day-of-month, month, day-of-week (0 = Sunday … 6 = Saturday).
- A field can be a single number, a hyphen range (inclusive), `*` for the full range, `*/N` for a step, or a comma-separated list.
- Jobs can target a Pub/Sub topic, an App Engine app, or a public HTTP endpoint.
- Like Cloud Tasks, Cloud Scheduler can attach a service-account token to authenticated HTTP requests.
- Execution is guaranteed, and failed executions are retried.

```text
Cron fields:  minute   hour   day-of-month   month   day-of-week
Example:        15      0          *           *          *      → 00:15 every day
Example:         *      */2        *           *          *      → every 2 hours
```

> **Exam trap:** Cloud Scheduler's default time zone is UTC, and the module explicitly recommends staying on UTC — time zones observing daylight saving time can cause a job to be **skipped** (clocks moving forward) or **run twice** (clocks moving back).

---

# Choosing the Right Application Integration Service

| Need | Service |
| --- | --- |
| Publish/subscribe messaging you build and manage yourself | Pub/Sub |
| React to events from many GCP services/third parties without writing ingestion code, in a standard format | Eventarc |
| A centrally controlled, stateful, long-running process with retries and full observability | Workflows |
| Asynchronously execute one specific known service, possibly deferred to a future time | Cloud Tasks |
| Run a job on a recurring schedule (cron) | Cloud Scheduler |

---

# Module Summary

Microservices push coordination complexity out of individual services and into how they communicate. Two patterns handle that: **service choreography**, where services react to each other's events independently with no central source of truth, and **service orchestration**, where a central orchestrator controls the whole process with a single point of failure in exchange for visibility and easier troubleshooting.

Google Cloud provides an "Application Integration" toolbox to implement both: **Pub/Sub** for publish/subscribe messaging (at-least-once delivery, pull or push subscriptions); **Eventarc**, an abstraction over Pub/Sub that wires up many event sources automatically and standardizes on the CloudEvents format; **Workflows**, the managed implementation of orchestration, providing a stateful, observable, long-running central process; **Cloud Tasks**, for explicitly invoking one known service asynchronously (as opposed to Pub/Sub's implicit, subscriber-agnostic invocation); and **Cloud Scheduler**, a managed cron scheduler (default and recommended time zone: UTC, to avoid daylight-saving skip/duplicate issues). In practice, real applications combine orchestration within a service (Workflows) with choreography between services (Eventarc).

---

# Key Points

- Choreography = distributed control, no central source of truth, strong for decentralized teams/orgs. Orchestration = centralized control, single point of failure, strong for complex centrally managed processes.
- Pub/Sub gives at-least-once delivery; a message is removed from a subscriber's queue only after acknowledgment.
- Eventarc is built on top of Pub/Sub as its transport layer — it's not a competing product, it's an abstraction that adds many event sources and the standardized CloudEvents format.
- Workflows is a central source of truth that can hold state, retry, poll, or wait up to a year, making long-running processes practical.
- Cloud Tasks uses explicit invocation (you choose exactly which service and when); Pub/Sub uses implicit invocation (you publish, whoever's subscribed reacts).
- Cloud Scheduler jobs use standard Unix cron syntax; default to UTC to avoid daylight-saving-time skip/duplicate execution bugs.
- Real systems typically combine both patterns — orchestration inside a service boundary (Workflows), choreography across service boundaries (Eventarc).
