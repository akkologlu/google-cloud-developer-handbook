# Module 13 – Calling and Connecting Cloud Run Functions

> This is Module 2 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 1 (Module 12: Introduction to Cloud Run Functions).

---

# Overview

This module covers three related mechanisms for calling and connecting Cloud Run functions to the rest of your architecture:

```text
Triggers (how a function gets invoked) → Workflows (how functions/services get chained together) → Serverless VPC Access (how a function reaches into a VPC network)
```

---

# Function Triggers

You set up a Cloud Run function to execute in response to a scenario by specifying a **trigger** at deployment time.

| Trigger category | Reacts to | Corresponds to |
| --- | --- | --- |
| HTTP trigger | HTTP(S) requests | HTTP functions |
| Event trigger | Events within your Google Cloud project | Event-driven functions |

- Multiple functions can be deployed with the same trigger source settings, so the same event causes all of them to execute.
- A single function, however, **cannot be bound to more than one trigger** at a time.
- Event-driven functions use **Eventarc event triggers**, created with filters (service name, method name, event type, and other criteria) — configurable via the Google Cloud console or the gcloud CLI.

> **Exam trap:** "One event can fan out to many functions" and "one function can have only one trigger" are not contradictory — fan-out happens by deploying *multiple functions* that share the same trigger source, not by attaching multiple triggers to a single function.

**All event-driven Cloud Run functions use Eventarc for event delivery.** Eventarc supports 90+ Google Cloud sources, including Cloud Audit Logs, external SaaS event sources, and custom sources published to Pub/Sub — which is also how Cloud Run functions can integrate with any Google service that uses Pub/Sub as an event bus (e.g., Cloud Logging, Cloud Scheduler, Gmail Push Notifications).

---

# Trigger Types

| Trigger type | Fires when | Requirements |
| --- | --- | --- |
| HTTP trigger | An HTTP(S) request hits the function's assigned URL; supports GET, POST, PUT, DELETE, OPTIONS | None beyond deploying as an HTTP function |
| Pub/Sub trigger | A message is published to a specified Pub/Sub topic | Function must be event-driven; implemented as an Eventarc trigger |
| Cloud Storage trigger | A chosen event type occurs on an object in a specified bucket | Function must be event-driven; implemented as an Eventarc trigger |
| Firestore trigger | A chosen event type (create, update, delete, write) occurs on a document at a specified path | Firestore must be in the same Google Cloud project as the function; applies only at the document level, never a field or collection |
| Firebase triggers | Events from Google Analytics for Firebase (1st gen only), Firebase Realtime Database, Firebase Authentication (1st gen only), Firebase Remote Config | Firebase service must be in the same Google Cloud project as the function |

**Event payload format depends on the function's implementation style:**

| Implementation style | Pub/Sub event data arrives as | Cloud Storage event data arrives as |
| --- | --- | --- |
| CloudEvent function | CloudEvents format | CloudEvents format |
| Background function | `PubsubMessage` format | `StorageObjectData` format |

> **Exam trap:** A Firestore trigger fires per-document, not per-field or per-collection. "Create a trigger for one specific field" is not something Firestore triggers can express — the finest granularity is the document.

---

# Workflows

**Workflows** is a fully managed, serverless orchestration platform that executes services in an order you define — the concrete implementation of the **service orchestration** pattern, acting as the central orchestrator.

| Property | Detail |
| --- | --- |
| What it connects | HTTP services built with Cloud Run functions, external APIs, and other Cloud services like Cloud Run |
| State | Can hold state, retry, poll, or wait for up to a year — enables long-running business processes |
| Observability | Every execution is logged and observable, giving a central source of truth for the application flow |
| Definition format | YAML or JSON — a series of steps |

**Build process:**

1. Enable the required APIs (Cloud Run functions, Cloud Run, Workflows, and any other services used); create any needed service accounts.
2. Write and deploy the functions as **HTTP functions** with HTTP triggers, so each gets a URL endpoint that Workflows can call.
3. Test the functions individually — with curl or another HTTP client, and locally before deployment as a best practice.
4. Create the workflow that connects the functions, using the Workflows syntax (YAML/JSON).
5. Deploy and execute the workflow.

**Inside a workflow definition:**

- Steps invoke Cloud Run functions via an HTTP request (e.g., a `GET` step, then a `POST` step), with the function's URL supplied as an argument.
- The result of one step can be passed as input to the next step — chaining function outputs.
- A workflow can also call an **external REST API** (passing prior results as query parameters) and connect to a **Cloud Run service** — the Cloud Run service's result can become the overall workflow's result.

> **Exam trap:** Functions called from a workflow must be **HTTP functions** — Workflows invokes them over HTTP(S), so an event-driven-only function with no HTTP trigger cannot be wired into a workflow step the same way.

---

# Serverless VPC Access

A **Virtual Private Cloud (VPC) network** is a virtual version of a physical network inside Google's production network — a global resource made of regional subnets connected by a global wide-area network.

**Serverless VPC Access** lets you connect Cloud Run functions directly to your VPC network, so the function can reach resources that only have an **internal IP address**:

- Compute Engine VM instances
- Memorystore
- Other resources with an internal IP

Traffic flows over **internal DNS and internal IP addresses** — it is never exposed to the public internet.

**Setup steps:**

| Step | Detail |
| --- | --- |
| 1. Enable the API | Enable the Serverless VPC Access API |
| 2. Create a connector | A Serverless VPC Access connector handles traffic between the serverless environment and the VPC network |
| 3. Attach the connector | Attach it to a specific VPC network and region — the connector's region **must match** the region where the function is deployed |
| 4. Dedicate a subnet | A subnet or CIDR range must be configured exclusively for the connector's use |
| 5. Configure the function | Each function that needs the connector must be individually configured to use it (console or gcloud) |

You can further restrict a connector's access with **firewall rules**, and connectors can also reach resources in a **Shared VPC** network.

> **Exam trap:** A Serverless VPC Access connector's region must match the function's deployment region — mismatched regions are a common cause of "the function can't reach my VPC resource" failures, not a firewall or IAM problem.

---

# Module Summary

Cloud Run functions are invoked through triggers — HTTP triggers for direct HTTP(S) calls, and event triggers (all delivered via Eventarc) for Pub/Sub, Cloud Storage, Firestore, and Firebase events; a function binds to exactly one trigger, though many functions can share the same trigger source. To chain functions and other services into a stateful, observable process, Workflows orchestrates HTTP functions, external REST APIs, and Cloud Run services through a YAML/JSON step definition, passing results from one step into the next. To reach resources that only have internal IP addresses — VM instances, Memorystore, and more — a function uses Serverless VPC Access: a connector, dedicated to a subnet/CIDR range, attached to a VPC network and region matching the function, and explicitly enabled per function, keeping all traffic off the public internet.

---

# Key Points

- Triggers come in two categories — HTTP (HTTP functions) and event (event-driven functions) — and a function can be bound to only one trigger, though multiple functions may share the same trigger source.
- All event-driven functions use Eventarc for delivery, which supports 90+ sources, including custom sources published to Pub/Sub.
- Pub/Sub, Cloud Storage, Firestore, and Firebase triggers each require an event-driven function; Firestore triggers apply strictly at the document level.
- CloudEvent functions receive Pub/Sub/Cloud Storage data in the CloudEvents format; Background functions receive `PubsubMessage`/`StorageObjectData` formats instead.
- Workflows is a fully managed orchestration platform that chains HTTP functions, external REST APIs, and Cloud Run services via a YAML/JSON step definition, holding state and retrying for up to a year.
- Functions invoked from a workflow must be deployed as HTTP functions so Workflows can call them over HTTP(S).
- Serverless VPC Access connects functions to VM instances, Memorystore, and other internal-IP resources over internal DNS/IP, with no exposure to the public internet.
- A Serverless VPC Access connector needs its own dedicated subnet/CIDR range, must be attached to a region matching the function's region, and must be explicitly configured per function; firewall rules and Shared VPC are also supported.
