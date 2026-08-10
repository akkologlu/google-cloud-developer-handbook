# Module 11 — Choreography and Orchestration: Practice Questions

This set covers service choreography vs. service orchestration, Pub/Sub, Eventarc and the CloudEvents format, Workflows, choosing between choreography and orchestration in practice, Cloud Tasks vs. Pub/Sub, and Cloud Scheduler. This module is Module 3 of the "Service Orchestration and Choreography on Google Cloud" course, and is the first to name concrete Google Cloud products, building on Module 1 ([Introduction to Microservices](../09-introduction-to-microservices/practice-questions.en.md)) and Module 2 ([Event-Driven Applications](../10-event-driven-applications/practice-questions.en.md)).

The questions are weighted toward the distinctions that actually trip people up: why orchestration isn't a strictly "safer" default, why Eventarc isn't a Pub/Sub competitor, why Cloud Tasks and Pub/Sub aren't interchangeable, and the UTC/daylight-saving trap in Cloud Scheduler.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A team implements an order-processing flow where each microservice listens for events and independently decides what to do next — there is no single service that owns the overall process definition. Which coordination pattern is this, and what is its defining characteristic?

A. Service orchestration; the defining characteristic is a central orchestrator controlling every interaction.
B. Service choreography; the defining characteristic is that each service works independently, with no central source of truth for the overall process.
C. Service choreography; the defining characteristic is that all services share a single database as the source of truth.
D. Service orchestration; the defining characteristic is that services are tightly coupled to each other's internal implementation.

**2.** A central Workflows-based process coordinates every step of a business process, and the team notes that if this central process goes down, none of the coordinated work can proceed. What concept does this illustrate, and does it disqualify orchestration as a valid pattern?

A. This illustrates a single point of failure, which is an inherent trade-off of orchestration, not a disqualifying flaw — orchestration is still a strong pattern for centrally managed complex processes.
B. This illustrates decentralized control, which means the team accidentally implemented choreography instead.
C. This is a bug specific to Workflows; other orchestration tools don't have this property.
D. This disqualifies orchestration entirely — the module recommends never using a central orchestrator for production systems.

**3.** A Pub/Sub subscriber processes a message but crashes before it can acknowledge it. What happens to that message, and what delivery guarantee does this illustrate?

A. The message is permanently lost, since Pub/Sub does not track acknowledgment state.
B. The message remains in the subscriber's queue (since it was never acknowledged) and will be redelivered — this is what "at-least-once delivery" means.
C. The message is delivered to a different topic automatically to avoid reprocessing.
D. Pub/Sub guarantees exactly-once delivery, so the message is discarded rather than redelivered.

**4.** A subscriber wants Pub/Sub to automatically send new messages to a configured HTTP endpoint as soon as they arrive, rather than checking for them itself. Which subscription type does this describe?

A. A pull subscription.
B. A push subscription.
C. A polling subscription.
D. A choreographed subscription.

**5.** A GCP service doesn't have built-in direct support for sending events to Eventarc. How does Eventarc typically still capture events from that service, and what does this spare the developer from doing?

A. Eventarc cannot capture events from that service at all; direct support is mandatory.
B. Eventarc can use Cloud Audit Logs entries from that service to generate events, sparing the developer from writing code to parse audit logs or poll for events.
C. The developer must manually publish a custom Pub/Sub message every time the service does something notable.
D. Eventarc requires the service to be rewritten using the Eventarc SDK before any events can be captured.

**6.** A developer claims that using Eventarc means their application must directly manage Pub/Sub topics and subscriptions, just like using Pub/Sub directly would. Is this accurate?

A. Yes — Eventarc is just a thin naming convention over raw Pub/Sub with no real difference in what the application must manage.
B. No — Eventarc uses Pub/Sub as its transport layer for reliability and observability, but automatically creates and manages the underlying topics and subscriptions; the application only needs to accept the HTTP requests Eventarc sends.
C. No — Eventarc replaces Pub/Sub entirely and never uses it internally.
D. Yes, but only for third-party event sources; GCP-native sources bypass Pub/Sub entirely.

**7.** Two teams build event-driven services using different publishers, and each publisher has historically used its own custom event format. What does Eventarc's use of the CloudEvents format solve here, and why does it matter for the code the teams write?

A. It solves nothing — CloudEvents is purely a documentation convention with no effect on application code.
B. It provides a common metadata format for describing event data, so both teams can use SDKs (available in Python, JavaScript, Java, Go, C#, Ruby, PHP, and more) and the same event-handling logic regardless of the source or type of the event, instead of writing custom parsing per publisher's format.
C. It forces both publishers to use the exact same programming language.
D. It eliminates the need for event triggers, since CloudEvents routes messages automatically without any configured rules.

**8.** A workflow needs to check inventory, branch based on stock availability, call different services depending on that branch, update a database, and finally notify a customer by email — potentially waiting on a slow external step along the way. Which Google Cloud product is designed for exactly this, and what capability makes long delays inside the process practical?

A. Cloud Tasks, because it guarantees at-least-once delivery.
B. Pub/Sub, because publishers don't need to know who their subscribers are.
C. Workflows, because a workflow can hold state, retry, poll, or wait for up to a year, which makes long-running, stateful business processes practical.
D. Cloud Scheduler, because it can trigger jobs on a recurring cron schedule.

**9.** A team implements the same order-processing logic twice: once using choreography (each service emits "ready for next step" events, including in-stock/out-of-stock decisions made locally by whichever service owns that step) and once using Workflows-based orchestration. What specific difficulty does the module say the choreographed version runs into that the orchestrated version avoids?

A. The choreographed version cannot use Firestore, while the orchestrated version can.
B. The choreographed version is inherently slower at runtime because events always add network latency, while orchestrated calls are always in-process.
C. The choreographed version makes it hard to get visibility, error handling, and retries right — for example, troubleshooting the system or handling a process that aborts between locking inventory and updating it — while Workflows tracks each execution separately with the logic defined in one place, with built-in retries and error handling.
D. The choreographed version requires all services to be owned by a single team, while the orchestrated version does not.

**10.** An organization's Order, Fulfillment, and Marketing services are each owned and independently managed by different teams in different parts of the company. A architect proposes putting all coordination between these three services under a single shared Workflows orchestrator owned by one team. What problem does the module warn this is likely to run into?

A. None — Workflows always scales cleanly across any number of independently managed teams with no coordination cost.
B. Shared management of a central orchestration process can be difficult when the coordinated services are built and managed by different teams or organizations — this is exactly the scenario where choreography's decentralized control tends to fit better.
C. Workflows technically cannot call services owned by a different team due to IAM restrictions that cannot be worked around.
D. This is not a real trade-off the module discusses; orchestration and choreography are presented as functionally identical in this scenario.

**11.** In a design where the Order, Fulfillment, and Marketing services are each internally implemented using Workflows, while Eventarc carries event triggers between those services and reacts to new files landing in a Cloud Storage bucket, which pattern is being used, and where?

A. Pure orchestration everywhere — Eventarc is just another orchestrator running alongside Workflows.
B. Pure choreography everywhere — Workflows is only used for logging, not for controlling any process.
C. Orchestration inside each service's own process (via Workflows) combined with choreography between the services (via Eventarc) — a common way the two patterns are combined in practice.
D. Neither pattern applies once more than two services are involved.

**12.** A backend needs to offload a specific slow operation (e.g., generating a large report) to a known worker service, wants to control exactly when that dispatch happens, and needs automatic retries with a configurable delay if the worker fails. Which service fits this best, and what is the key distinguishing property versus the most similar alternative?

A. Pub/Sub, because it guarantees at-least-once delivery to every interested subscriber.
B. Cloud Tasks, because it uses explicit invocation — the creator retains full control over the execution and destination of the task — unlike Pub/Sub's implicit invocation, where the publisher has no control over which subscribers receive the message.
C. Eventarc, because it can trigger Cloud Audit Log-based events automatically.
D. Cloud Scheduler, because cron jobs always guarantee retries with configurable delay.

**13.** A publisher sends a message to a Pub/Sub topic without knowing, or caring, which services (if any) are currently subscribed. Which invocation model does this describe, and how does it differ from Cloud Tasks?

A. Explicit invocation — the same model Cloud Tasks uses.
B. Implicit invocation — publishing implicitly causes any subscribers to execute, with no control over which subscribing services receive the message; Cloud Tasks instead uses explicit invocation, where the creator places a task in a queue tied to a specific, known endpoint.
C. Scheduled invocation — identical in behavior to a Cloud Scheduler cron job.
D. Choreographed invocation — a concept unrelated to Pub/Sub's actual delivery model.

**14.** A team configures a Cloud Scheduler job with the cron string `15 0 * * *` in the `America/New_York` time zone instead of leaving it at the default. Twice a year, the job either fails to run or runs twice on the same day. What causes this, and what does the module recommend to avoid it?

A. This is a Cloud Scheduler bug with no recommended fix.
B. The `America/New_York` time zone observes daylight saving time, which can cause a job to be skipped (when clocks move forward) or run twice (when clocks move back); the module recommends using the UTC time zone, which is also the default, to avoid this.
C. The cron string itself is invalid, which is unrelated to time zones.
D. This only happens with push subscriptions, not with Cloud Scheduler jobs.

**15.** In the Unix cron format used by Cloud Scheduler, a job's hour field is set to `*/2`. What does this mean, and which field would you change to restrict the job to only run on Mondays?

A. `*/2` means "every 2 hours"; restricting to Mondays would require changing the day-of-week field (the fifth field, where Monday is represented as 1 in the 0=Sunday...6=Saturday scheme).
B. `*/2` means "only at 2 AM"; restricting to Mondays would require changing the month field.
C. `*/2` means "twice per month"; restricting to Mondays would require changing the minute field.
D. `*/2` is invalid syntax; Cloud Scheduler does not support step values.

**16.** A solutions architect summarizes Pub/Sub, Eventarc, Workflows, Cloud Tasks, and Cloud Scheduler as Google Cloud's "Application Integration" toolbox and says a full-featured microservices application might reasonably use several of them together rather than picking just one. Is this an accurate takeaway from the module?

A. No — the module presents these five services as mutually exclusive alternatives; using more than one in the same application is discouraged.
B. Yes — the module explicitly frames these five services as a toolbox that a full-featured microservices application may benefit from using in combination (e.g., Workflows for orchestration within a service, Eventarc for choreography between services, Cloud Tasks for offloaded async work, Cloud Scheduler for recurring jobs, Pub/Sub underlying some of this).
C. No — only Pub/Sub and Eventarc are meant to be used together; Workflows, Cloud Tasks, and Cloud Scheduler are legacy services not meant for new applications.
D. Yes, but only if the application avoids Pub/Sub entirely, since Eventarc is meant to fully replace it.

---

## Answer Key & Explanations

**1. Correct answer: B.**
Distributed decision-making with no single service owning the overall process is exactly service choreography, as contrasted with orchestration's central controller. Sharing a database (C) is not what defines choreography, and tight coupling (D) contradicts the module's point that both patterns produce loosely coupled services.

**2. Correct answer: A.**
The module explicitly names this trade-off: unlike the fully distributed services in choreography, orchestration has a single point of failure — if the orchestrator is not operable, orchestrated processes cannot run. This is presented as an inherent trade-off, not a reason to abandon orchestration, which remains a strong pattern for centrally managed complex processes.

**3. Correct answer: B.**
A message is only removed from a subscriber's queue once acknowledged; since it wasn't acknowledged before the crash, it remains and will be redelivered. This "deletion follows successful handling" behavior is exactly what guarantees at-least-once delivery — a message may be delivered more than once, but not zero times.

**4. Correct answer: B.**
A push subscription is defined by Pub/Sub automatically sending the message to a configured endpoint, as opposed to a pull subscription, where the subscriber occasionally polls for new messages itself.

**5. Correct answer: B.**
For services without direct event-source support, Eventarc can seamlessly use Cloud Audit Logs entries to generate events — the developer doesn't need to write code to parse audit logs or poll for events, which is exactly the manual work this capability removes.

**6. Correct answer: B.**
Eventarc does use Pub/Sub as its transport layer, but it automatically creates and manages the topics and subscriptions involved — the application itself only needs to accept the HTTP requests that Eventarc sends, never touching Pub/Sub directly. This is the key difference from using Pub/Sub on its own.

**7. Correct answer: B.**
CloudEvents provides a common metadata format for describing event data regardless of source, and its SDKs (available for many popular languages) let developers reuse the same event-handling logic across different event types and sources instead of writing custom parsing per publisher — solving exactly the "everyone had their own format" problem described in the scenario.

**8. Correct answer: C.**
Workflows is the module's answer for stateful, automated, potentially long-running business processes; the ability for a workflow to hold state, retry, poll, or wait for up to a year is what specifically makes long delays and multi-step branching processes practical, unlike Cloud Tasks, Pub/Sub, or Cloud Scheduler, which solve different problems.

**9. Correct answer: C.**
The module is explicit that a choreographed order flow makes visibility, error handling, and retries hard to get right — including troubleshooting an event-based system and handling a process aborting mid-flow — while Workflows-based orchestration tracks each execution separately with the process logic defined in a single location, with retries and error handling built in.

**10. Correct answer: B.**
The module explicitly warns that shared management of a central orchestration process can be difficult when the underlying services are built and managed by different teams or organizations — this decentralized-ownership scenario is precisely where choreography's decentralized control tends to be the better fit, not a flaw to engineer around with orchestration.

**11. Correct answer: C.**
This is the module's own combined example: Order, Fulfillment, and Marketing are each implemented using Workflows (orchestration within each service), while Eventarc carries event triggers between the services and reacts to file uploads (choreography across the boundaries between services) — the two patterns operating at different scopes simultaneously.

**12. Correct answer: B.**
Cloud Tasks is defined by explicit invocation: the task creator retains full control over execution and destination, placing the task in a queue tied to a specific endpoint and optionally deferring dispatch — this is the opposite of Pub/Sub's implicit invocation, where the publisher has no control over which subscribers act on a message. Retry attempts and delay are also directly configurable on a Cloud Tasks queue.

**13. Correct answer: B.**
This is implicit invocation by definition: the publisher's action implicitly causes any current subscribers to execute, without the publisher controlling or even necessarily knowing who those subscribers are. Cloud Tasks instead uses explicit invocation, giving the creator direct control over exactly which endpoint receives the task.

**14. Correct answer: B.**
Time zones that observe daylight saving time can cause a scheduled job to be skipped when clocks move forward or to run twice when clocks move back, because the specified local time either doesn't exist or occurs twice on the transition day. The module recommends using UTC (also the default) specifically to avoid this class of bug.

**15. Correct answer: A.**
A cron field written as `*/N` means "every N units, following a range" — for the hour field, `*/2` means every two hours. Restricting execution to a single day of the week is done through the fifth (day-of-week) field, not the month or minute fields.

**16. Correct answer: B.**
The module explicitly closes by naming Pub/Sub, Eventarc, Workflows, Cloud Tasks, and Cloud Scheduler together as an "Application Integration" toolbox and states that a full-featured microservices application may benefit from any or all of them — it does not present them as mutually exclusive, and Eventarc is explicitly built on top of Pub/Sub rather than replacing it.
