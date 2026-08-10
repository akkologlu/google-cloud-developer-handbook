# Module 9 – Introduction to Microservices

> This is Module 1 of the "Service Orchestration and Choreography on Google Cloud" course. Later modules of that course (service orchestration and choreography patterns on Google Cloud) will be added to this handbook as their own numbered modules once their transcripts become available.

---

# Overview

Before you can talk about orchestrating or choreographing services, you need to know why services exist as separate units in the first place.

This lesson traces how application architecture evolved from a single codebase to independently deployable microservices, and lays out the concrete trade-offs of making that move.

```text
Monolith

↓ (decompose into reusable services)

Service-Oriented Architecture (SOA)

↓ (decentralize, drop the shared middleware)

Microservices
```

---

# Monolithic Applications

Early enterprise applications were built as a single, self-contained codebase: UI, business logic, and data access all in one application, backed by one large relational database.

```text
Monolith
├── User Interface
├── Business Logic
└── Data Access → Relational Database
```

- The codebase grows more complex with every major change.
- Because everything lives in one application, the code tends to become **tightly coupled**.
- Tightly coupled code is hard to maintain — fixing one bug risks introducing another.

---

# Service-Oriented Architecture (SOA)

SOA was an attempt to fix monoliths by breaking an application into reusable **services**, each handling one discrete business function, communicating over defined interfaces via messaging.

```text
Service A   Service B   Service C
    \           |           /
     \          |          /
        Enterprise Service Bus (ESB)
   (connectivity, security, routing, transformation)
```

**What SOA got right:**

- Services were smaller and more loosely coupled than a monolith.
- Smaller services → smaller, more focused teams.
- Applications were assembled by combining reusable services.

**Where SOA broke down:**

- All inter-service messaging went through a central **Enterprise Service Bus (ESB)** — a messaging middleware component that handled protocol transformation, routing, and data transformation.
- Complexity didn't disappear, it **shifted** from the services into the ESB integrations.
- The ESB was typically owned by one central team, so integration work for any application became a bottleneck.
- Changing one application's integration could destabilize other applications sharing the ESB.
- Even ESB software upgrades risked breaking existing integrations, so they required heavy testing.

---

# Microservices

Microservices are a **decentralized** alternative to SOA: separate, limited-scope services, each with its own database, exposing an API that other services call.

```text
Orders Service        Products Service       Reviews Service
   + own DB               + own DB               + own DB
      \                      |                       /
       \                     |                      /
            Calls happen directly via APIs (no shared ESB)
```

- Separation between microservices leads to **loose coupling**.
- Loosely coupled services are easier to maintain, update, and deploy independently.

## Starting with Microservices vs. Starting with a Monolith

| Situation | Recommended starting point |
| --- | --- |
| You don't yet understand the problem domain well enough to draw service boundaries | Start with a **monolith**, migrate to microservices later as you learn the domain |
| You need to release changes quickly and iterate in an agile fashion | Start with **microservices** |
| Your team will grow significantly over time | Start with **microservices** — natural service boundaries let new members focus on one smaller piece |

If you start with a monolith, design it to be **modular** so a future migration to microservices is easier.

> **Exam trap:** "Microservices are always the right first choice" is false. Designing service boundaries without domain expertise is one of the hardest parts of a new microservices project — a monolith lets you defer that decision until you understand the domain better.

---

# Benefits of Microservices

| Benefit | Why it matters |
| --- | --- |
| Simpler codebase | Each microservice is smaller and easier for a small team to fully understand |
| Easier unit testing | Clear service boundaries make isolated testing straightforward |
| Independent deployability | Teams update and deploy their own service on their own schedule; other services are only affected by breaking interface changes |
| More agile development | Services can be updated and deployed without touching the rest of the system |
| Polyglot technology choice | Each team can pick the language/framework that best fits their service — callers only depend on the API, not the implementation |
| Cross-platform interoperability | Services on different platforms can still call each other over HTTP APIs |
| Independent scaling | Each service scales based on its own traffic, optimizing infrastructure cost instead of over-provisioning for peak load everywhere |

---

# Challenges of Microservices

| Challenge | Why it's hard |
| --- | --- |
| Operational burden | Tens, hundreds, or thousands of deployable entities need automated builds, testing, and deployment to stay manageable |
| Consistency across services | Logging, reporting, security, and authorization must stay consistent even as the number of services grows |
| Communication complexity | Poorly designed inter-service communication becomes a "spider web" that's hard to reason about |
| Network latency | Calls between microservices cross the network — thousands of times slower than in-process calls in a monolith; latency compounds when a business operation needs many calls |
| Integration testing | Testing the full system realistically requires modeling the entire production deployment, not just each service in isolation |
| Debugging | Each microservice produces its own logs, so tracing a request across many services is harder than debugging a single process |

> A microservices architecture pays off only with a real commitment to automation and operational excellence — the benefits generally outweigh the challenges, but only if you invest in the tooling to manage the added complexity.

---

# Monolith vs. SOA vs. Microservices

| | Monolith | SOA | Microservices |
| --- | --- | --- | --- |
| Coupling | Tightly coupled | Loosely coupled, but ESB-centralized | Loosely coupled, decentralized |
| Communication | In-process calls | Messaging via central ESB | Direct API calls between services |
| Data | One shared database | Often shared/centrally governed | Each service owns its own database |
| Deployability | Single deployable unit | Services deployable, but ESB changes are a bottleneck | Fully independent deployment per service |
| Bottleneck | The codebase itself | The central ESB and its owning team | Operational tooling and cross-service consistency |

---

# Module Summary

Applications evolved from **monoliths** (simple to start, but tightly coupled and hard to maintain at scale) to **SOA** (reusable services, but complexity shifted into a centrally-owned ESB that became the new bottleneck) to **microservices** (decentralized, independently deployable services communicating directly over APIs).

Choosing microservices from day one isn't automatically correct — designing service boundaries without domain expertise is genuinely hard, so starting with a modular monolith and migrating later is often the safer path. Microservices deliver simpler codebases, easier testing, independent deployability, technology freedom, and independent scaling — at the cost of operational burden, cross-service consistency work, network latency, harder integration testing, and harder debugging.

---

# Key Points

- Monoliths are simple to start but become tightly coupled and hard to maintain as they grow.
- SOA introduced reusable services connected through an Enterprise Service Bus (ESB) — but the ESB itself became a centralized bottleneck.
- Microservices decentralize further: each service owns its data and exposes an API, with no shared middleware layer.
- Start with a monolith if you don't yet understand the problem domain; start with microservices if you need agile delivery or expect significant team growth.
- Microservices' main benefits: simpler codebases, easier testing, independent deployability, polyglot technology, independent scaling.
- Microservices' main challenges: operational burden, consistency across services, communication complexity, network latency, integration testing, and debugging across distributed logs.
