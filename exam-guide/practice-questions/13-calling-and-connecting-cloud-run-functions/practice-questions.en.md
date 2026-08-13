# Module 13 — Calling and Connecting Cloud Run Functions: Practice Questions

This set covers the two trigger categories (HTTP triggers and event triggers) and how each specific trigger type — HTTP, Pub/Sub, Cloud Storage, Firestore, and Firebase — behaves, the role of Eventarc as the universal event-delivery mechanism, how CloudEvent functions and Background functions receive event data differently, how Workflows orchestrates Cloud Run functions and other services (build process, workflow definitions, chaining results, calling external REST APIs and Cloud Run services), and how Serverless VPC Access connects functions to resources with only an internal IP address. This module is Module 2 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 1 (Module 12).

The questions are weighted toward the distinctions that actually trip people up: the difference between binding one trigger to a function versus fanning an event out across multiple functions, why functions called from a workflow must be HTTP functions, the document-level-only granularity of Firestore triggers, and why a Serverless VPC Access connector's region has to match the function's region.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A function needs to run whenever a specific kind of event occurs inside a Google Cloud project, with no external caller involved. Which trigger category fits, and what is it correlated with?

A. An event trigger, which corresponds to an event-driven function.
B. An event trigger, which corresponds to an HTTP function.
C. An HTTP trigger, which corresponds to an event-driven function.
D. Neither category applies; only Cloud Scheduler can start a function without a caller.

**2.** A team wants three different functions to all execute whenever the same Pub/Sub message arrives, but a developer insists this is impossible because "a function can only have one trigger." Evaluate this claim.

A. The claim is correct, and there is no way to have multiple functions react to a single event.
B. The claim is correct, but only for HTTP-triggered functions, not event-driven ones.
C. The claim is wrong, because a single function can be bound to multiple triggers simultaneously.
D. A single function can indeed be bound to only one trigger at a time, but this doesn't prevent fan-out — you deploy multiple functions that each share the same trigger source settings.

**3.** A developer wants to know which service actually delivers events to an event-driven Cloud Run function, regardless of whether the source is Pub/Sub, Cloud Storage, or something else entirely. What is the answer, and roughly how many sources does it support?

A. All event-driven Cloud Run functions use Eventarc for event delivery, which supports more than 90 Google Cloud sources.
B. Each event source (Pub/Sub, Cloud Storage, Firestore) has its own separate, unrelated delivery mechanism.
C. Cloud Tasks delivers all events, regardless of the originating source.
D. Only Pub/Sub delivers events; Cloud Storage and Firestore triggers do not go through any delivery mechanism.

**4.** An HTTP-triggered function needs to support clients performing full CRUD-style operations against it directly over HTTP. Which request methods does an HTTP trigger support?

A. Only GET and POST; PUT and DELETE are not supported by HTTP triggers.
B. Only POST, since HTTP triggers are designed exclusively for webhook-style calls.
C. GET, POST, PUT, DELETE, and OPTIONS.
D. HTTP triggers do not support any specific method restrictions or list — any custom method name is accepted.

**5.** A function is meant to run whenever a message is published to a particular Pub/Sub topic. What must be true about the function's implementation for a Pub/Sub trigger to work, and how is the trigger implemented under the hood?

A. The function can be implemented any way at all; Pub/Sub triggers do not require an event-driven implementation.
B. The function must be an HTTP function, since Pub/Sub triggers are a special case of HTTP triggers.
C. The function must poll the topic manually; Pub/Sub triggers only notify, they don't invoke.
D. The function must be implemented as an event-driven function, and in Cloud Run functions, Pub/Sub triggers are implemented as a type of Eventarc trigger.

**6.** Two functions are both triggered by the same Pub/Sub topic — one is a CloudEvent function, the other is a Background function. How does the Pub/Sub event data reach each of them?

A. Both receive the data in an identical format; the implementation style makes no difference to the payload.
B. The CloudEvent function receives the data in the CloudEvents format; the Background function receives it in the `PubsubMessage` format.
C. The CloudEvent function receives the data in the `PubsubMessage` format; the Background function receives it in the CloudEvents format.
D. Neither function can actually receive Pub/Sub event data; only HTTP functions can process Pub/Sub messages.

**7.** A Cloud Storage trigger is configured on a bucket, and the function that handles it is implemented as a Background function. In what format does the Cloud Storage event data arrive?

A. In the CloudEvents format, identical to a CloudEvent function.
B. As a raw file byte stream, with no structured format at all.
C. In the `StorageObjectData` format.
D. Cloud Storage triggers are not compatible with Background functions under any circumstances.

**8.** A team wants a function to fire whenever a specific field inside a Firestore document changes, without reacting to changes elsewhere in the document. Is this possible with a Firestore trigger?

A. No — Firestore triggers apply only at the document level; it is not possible to create a trigger scoped to a specific field or collection.
B. Yes — Firestore triggers can be scoped to an individual field using a field-path filter.
C. Yes — Firestore triggers can be scoped to an entire collection, which is finer-grained than the document level.
D. Firestore triggers do not support "update" as an event type, only "create" and "delete."

**9.** A function in Project A is meant to react to changes in a Firestore database that lives in Project B. Will this configuration work as described?

A. Yes, as long as both projects belong to the same organization.
B. Yes, cross-project Firestore triggers are explicitly supported with no extra configuration.
C. It depends on the event type; only "write" events can cross project boundaries.
D. No — Firestore must be in the same Google Cloud project as the function for the trigger to work.

**10.** A mobile backend team wants a function triggered by Firebase Authentication events, and needs to know if this is available regardless of which Cloud Run functions generation they use. What does the module say?

A. Firebase Authentication triggers work identically and without restriction on both generations.
B. Firebase Authentication triggers (like Google Analytics for Firebase triggers) are only available for Cloud Run functions (1st gen).
C. Firebase Authentication triggers are only available for Cloud Run functions (2nd gen), never 1st gen.
D. Firebase Authentication events cannot trigger Cloud Run functions at all; only Firestore and Realtime Database can.

**11.** A team wants a central platform to orchestrate a multi-step business process, holding state and retrying failed steps across a process that might run for months. What does the module recommend, and what specific capability makes long-running processes practical?

A. Cloud Scheduler, since it is designed for any process that runs longer than a few minutes.
B. Workflows, a fully managed serverless orchestration platform that can hold state, retry, poll, or wait for up to a year.
C. Pub/Sub, since messages can be retained indefinitely by configuring a long retention period.
D. Eventarc, since it can chain triggers together without any orchestration layer needed.

**12.** While building a workflow that will call two Cloud Run functions as steps, a developer wants to know what type those functions need to be. What is required, and why?

A. The functions must be Background functions, since only event-driven functions can be embedded in a workflow step.
B. There is no requirement; any function type works interchangeably as a workflow step.
C. The functions must be HTTP functions, since Workflows invokes each step through an HTTP request to the function's URL endpoint.
D. The functions must be CloudEvent functions specifically, since Workflows only understands the CloudEvents format.

**13.** A workflow definition includes a step that calls a first Cloud Run function, followed by a step that calls a second one. What format is the workflow definition written in, and how does data move between the two steps?

A. The definition is written in YAML or JSON, and the result generated by the first step can be provided as input to the second step.
B. The definition can only be written in a proprietary Workflows-specific binary format, and steps cannot share data.
C. The definition is written in XML, and each step must independently re-fetch any data it needs.
D. The definition is written in YAML only (never JSON), and results cannot be passed between steps under any circumstances.

**14.** Beyond invoking Cloud Run functions, a workflow needs to call an external REST API and also incorporate a Cloud Run service into the same process. Does the module describe this as achievable within a single workflow definition?

A. No — a workflow definition can only ever call Cloud Run functions, nothing else.
B. Only the external REST API can be included; Cloud Run services cannot be referenced from a workflow.
C. Only the Cloud Run service can be included; external REST APIs are out of scope for Workflows.
D. Yes — a workflow definition can include configuration to call an external REST API (e.g., passing a prior result as a query parameter) and to connect a Cloud Run service, whose result can become the workflow's overall result.

**15.** A function needs to reach a Compute Engine VM instance that only has an internal IP address, without exposing any of that traffic to the public internet. What Google Cloud capability enables this, and what kind of resource does it connect to?

A. Cloud NAT, which is designed specifically to expose internal IP addresses to Cloud Run functions.
B. Serverless VPC Access, which connects Cloud Run functions directly to a VPC network, enabling access over internal DNS and internal IP addresses to resources like Compute Engine VM instances and Memorystore.
C. Cloud Interconnect, which is used exclusively for connecting on-premises networks to Google Cloud.
D. No mechanism exists for a Cloud Run function to reach an internal-IP-only resource; only public IP resources are reachable.

**16.** A Serverless VPC Access connector has been created, attached to a VPC network, and given a dedicated subnet, but a function still cannot reach VPC resources. The connector was created in a different region from the function. What requirement was violated, and what else must still happen before the function can use the connector?

A. Regions do not need to match; the failure must be caused by a missing firewall rule instead.
B. The connector's region has no bearing on connectivity; the real missing step is enabling Cloud NAT.
C. The connector's region must match the region where the function is deployed, and in addition, each function that needs to use the connector must be individually configured to do so.
D. The subnet must be shared with other Google services, and dedicating it exclusively to the connector was itself the mistake.

---

## Answer Key & Explanations

**1. Correct answer: A.**
An event trigger reacts to events within the Google Cloud project and corresponds to an event-driven function — exactly the "no external caller" scenario described. HTTP triggers, by contrast, correspond to HTTP functions and require an HTTP(S) request.

**2. Correct answer: D.**
A single function can be bound to only one trigger at a time, but the same event can still cause multiple functions to execute — you achieve that fan-out by deploying multiple functions that share the same trigger source settings, not by attaching several triggers to one function.

**3. Correct answer: A.**
All event-driven Cloud Run functions use Eventarc for event delivery, and Eventarc supports more than 90 Google Cloud sources — including Cloud Audit Logs, external SaaS event sources, and custom sources published to Pub/Sub.

**4. Correct answer: C.**
HTTP triggers support the GET, POST, PUT, DELETE, and OPTIONS request methods, which is enough to support full CRUD-style interactions directly over HTTP.

**5. Correct answer: D.**
For a function to use a Pub/Sub trigger, it must be implemented as an event-driven function; in Cloud Run functions, Pub/Sub triggers are implemented as a type of Eventarc trigger under the hood.

**6. Correct answer: B.**
If a CloudEvent function is used, Pub/Sub event data is passed to it in the CloudEvents format; if a Background function is used, the same data is passed in the `PubsubMessage` format instead — the implementation style determines the payload shape.

**7. Correct answer: C.**
If a Background function is used, Cloud Storage event data is passed to it in the `StorageObjectData` format (a CloudEvent function would instead receive the CloudEvents format).

**8. Correct answer: A.**
Firestore triggers only apply at the document level — it is not possible to create a trigger scoped to a specific document field or to an entire collection; the finest granularity available is the document itself.

**9. Correct answer: D.**
Firestore must be in the same Google Cloud project as the function for a Firestore trigger to work — cross-project Firestore triggers are not supported, regardless of organization membership or event type.

**10. Correct answer: B.**
The module specifies that Google Analytics for Firebase and Firebase Authentication triggers are available for Cloud Run functions (1st gen) only, unlike Firebase Realtime Database and Firebase Remote Config triggers, which are not called out with that same restriction.

**11. Correct answer: B.**
Workflows is a fully managed, serverless orchestration platform, and the module specifically attributes its ability to hold state, retry, poll, or wait for up to a year as what makes long-running business processes practical.

**12. Correct answer: C.**
Functions used as workflow steps are written and deployed as HTTP functions with HTTP triggers, because Workflows invokes each step through an HTTP request sent to the function's URL endpoint.

**13. Correct answer: A.**
A workflow definition — the set of steps — can be written in either YAML or JSON format, and the result generated by one step (e.g., a first Cloud Run function) can be provided as input to the next step (e.g., a second Cloud Run function).

**14. Correct answer: D.**
A workflow definition can include configuration to connect to an external REST API endpoint (with a prior result passed in as a query parameter, for example) and configuration that connects a Cloud Run service, whose result can become the result of the overall workflow.

**15. Correct answer: B.**
Serverless VPC Access connects Cloud Run functions directly to a VPC network, enabling access to resources with only an internal IP address — such as Compute Engine VM instances and Memorystore — over internal DNS and internal IP addresses, so traffic is never exposed to the internet.

**16. Correct answer: C.**
The region configured for a Serverless VPC Access connector must match the region where the Cloud Run function is deployed. Beyond that, having a connector isn't enough on its own — each function that needs to reach the VPC network must be individually configured to use that connector.
