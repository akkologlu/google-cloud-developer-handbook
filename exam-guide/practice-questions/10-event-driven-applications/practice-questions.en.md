# Module 10 — Event-Driven Applications: Practice Questions

This set covers what an event is, why point-to-point communication between microservices becomes a problem, how an event intermediary decouples producers from consumers, and the three benefits of event-driven architecture: centralized auditing/access control, decoupling, and resilience through asynchronous processing (including push vs. polling). This module is Module 2 of the "Service Orchestration and Choreography on Google Cloud" course, building directly on Module 1 ([Introduction to Microservices](../09-introduction-to-microservices/practice-questions.en.md)).

The questions are weighted toward the distinctions that actually trip people up: what actually makes something an "event," why an unconsumed event isn't automatically a bug, how an event intermediary differs from an Enterprise Service Bus (ESB), and why asynchronous processing changes failure behavior.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A developer proposes editing an existing event record in the event log to "correct" a typo in a field, instead of emitting a new event. What does the module say about this, and why?

A. This is fine — events can always be edited in place as long as the change is documented.
B. Events are typically treated as immutable facts — a historical record of an occurrence that should not be modified or deleted. The correction should be a new event, not an edit to the old one.
C. Events can be edited, but only by the original producer service.
D. Events cannot be edited because the event intermediary enforces write-once storage at the network layer.

**2.** A service emits an event every time a product is added to a shopping cart, but no other service currently consumes that event. An engineer argues this must be a bug — "why would you emit an event nobody's listening to?" Is this actually a bug?

A. Yes — every event must have at least one active consumer at the moment it's produced, or the producer should not emit it.
B. No — an event can be generated even if it's never consumed; many producers don't know, or need to know, whether anything is consuming their events.
C. Yes, but only because events are expensive to store, so unconsumed events waste resources.
D. No, but only if the event intermediary is configured to silently drop events with zero consumers.

**3.** Three months after an event was originally produced, a newly added analytics service wants to process it for the first time, and a separate existing service wants to reprocess it after fixing a bug in its handler. Does the architecture described in the module support this?

A. No — an event can only be consumed once, by the first service that reads it.
B. No — events expire and are purged shortly after being produced, so reprocessing old events isn't possible.
C. Yes — an event can be persisted indefinitely and consumed as many times as necessary, including in parallel by multiple services.
D. Yes, but only the original consumer is allowed to re-read an event; new consumers cannot read events produced before they existed.

**4.** In a microservices system using direct, point-to-point calls, a team notices that Service A must know how to call Services B, C, and D directly, and that adding a new downstream service means modifying Service A's code. What problem does the module say this illustrates, and what architectural change addresses it?

A. This is expected and has no real downside; point-to-point calls are always preferred over any alternative.
B. This is the point-to-point "spider web" coupling problem; inserting an event intermediary between services lets producers stop needing to know which services consume their events.
C. This is solved by adding more microservices, which automatically reduces coupling.
D. This is solved by merging Services B, C, and D back into a single monolith.

**5.** A service acts as an event producer. According to the module, what does it need to know about the services that might consume its events?

A. It needs to know the network address and API contract of every consumer in advance.
B. It needs to know how many consumers exist so it can fan out requests correctly.
C. It isn't necessary for the producer to know anything about the services consuming its events — it only needs to know the event's format.
D. It needs to know which consumer will process the event first, to sequence delivery correctly.

**6.** A junior engineer says an event intermediary is "basically just an ESB with a different name" because both sit between services and handle message delivery. Is this an accurate comparison?

A. Yes — the module treats them as fully interchangeable terms for the same architectural role.
B. No — an ESB is a centralized routing/transformation layer that became a bottleneck because every integration change had to go through it and its owning team; an event intermediary specifically exists to decouple producers from consumers without recreating that kind of centralized change bottleneck.
C. No — an event intermediary requires messages to be synchronous, while an ESB only supports asynchronous messaging.
D. Yes, but only in Google Cloud implementations; on other clouds they are structurally different.

**7.** A compliance officer needs a timed, ordered record of every state change made to a distributed application, for an upcoming audit. Which property of event-driven architecture directly supports this requirement?

A. Push-based messaging, since it delivers events faster than polling.
B. A log of immutable events, which provides a timed, ordered record of every change to the application's state and can be used for auditing.
C. The fact that events can be deleted once consumed, keeping the audit log small.
D. The use of synchronous request/response calls, which guarantee ordering better than events do.

**8.** A security team wants a single place to enforce authentication and authorization for a distributed, event-based application, rather than reimplementing access checks in every microservice. What does the module say makes this possible?

A. Nothing — access control must be implemented independently in every consumer service.
B. A centralized event service can require authentication and authorization at the point where events flow through it, letting you control access to event-based services and data from one place.
C. Only the event producer can enforce access control; consumers have no way to restrict who reads events.
D. Access control is unnecessary in event-driven systems because events are immutable.

**9.** A team wants to add a brand-new fraud-detection service that consumes "order placed" events, without changing any code in the existing order-processing services. Does the module's description of event-driven architecture support this, and why?

A. No — any new consumer requires updating the producer to add the new service to its list of recipients.
B. Yes — because producers and consumers are decoupled and only need to agree on an event's format, a new event consumer can be added without modifying any existing services.
C. No — adding a new consumer always requires re-deploying the event intermediary and every producer.
D. Yes, but only if the new consumer is deployed on the same platform as the producer.

**10.** In a system built entirely on synchronous request/response calls, Service X calls Service Y, which calls Service Z. Service Z becomes unhealthy. What does the module say happens, and how would an event-driven design change this outcome?

A. Nothing changes either way — synchronous and event-driven architectures fail identically.
B. In the synchronous chain, the health of a service is affected by the health of every service it calls, so Z's failure can cascade back through Y to X; in an event-driven design, events destined for the unhealthy service can simply be replayed or redelivered once it recovers, rather than failing the whole chain immediately.
C. The synchronous chain is actually more resilient because failures are detected immediately.
D. Event-driven architecture prevents Service Z from ever becoming unhealthy in the first place.

**11.** After an outage, a consumer service comes back online and needs to catch up on the events it missed while it was down. What capability of event-driven architecture directly enables this recovery?

A. Events sent to an unhealthy service can be replayed or redelivered once that service comes back up, because events are persisted rather than discarded after a single delivery attempt.
B. This isn't possible — any events produced while a consumer was down are permanently lost.
C. The producer must manually resend each event by direct API call once it notices the consumer is back.
D. Recovery is only possible if the consumer was using synchronous request/response calls instead of events.

**12.** A team implements their event consumer by having it call the event source every 500 milliseconds and ask "is there anything new for me?" What is this pattern called, what's the downside, and what does the module recommend instead?

A. This is push-based messaging, and it's the recommended approach because it's simple to implement.
B. This is polling; it typically increases network I/O and adds unnecessary delay before new work is processed. Push-based messaging, where consumers are automatically notified when there's something to consume, is preferred.
C. This is called replay, and its downside is that it can only be used for consumer recovery after downtime, not for normal operation.
D. This is the event intermediary pattern, and there is no real downside compared to alternatives.

**13.** With push-based messaging, how does a consumer typically find out that a new event is available to process?

A. The consumer repeatedly queries the event intermediary on a fixed schedule until it finds new data.
B. The consumer is automatically notified by the intermediary when there's an event to consume, rather than having to ask.
C. The consumer has no way to know; a human operator must manually trigger processing.
D. The producer calls the consumer directly and synchronously, bypassing the event intermediary entirely.

**14.** A team argues that decoupling producers and consumers via an event intermediary must mean producers and consumers don't need to agree on anything at all. Is that accurate?

A. Yes — full decoupling means zero shared contract between producer and consumer.
B. No — a producer or consumer is only required to know the format of a specific event; that shared understanding of event format is still necessary even though neither side needs to know about the other's identity or implementation.
C. No — producers and consumers must share the same programming language and deployment platform.
D. Yes, but only for events related to auditing; functional events still require a direct API contract.

**15.** Summarizing the module's overall argument, why does event-driven architecture fit naturally with a microservices architecture (as opposed to, say, a monolith)?

A. It doesn't — event-driven architecture is specifically discouraged in microservices and is only useful inside a monolith.
B. Because microservices already communicate over the network in separate deployable units, and the point-to-point coupling problem this creates is exactly what an event intermediary is designed to remove — enabling decoupled, resilient, independently evolving services.
C. Because events eliminate the need for microservices to have their own databases.
D. Because event-driven architecture requires all services to be written in the same programming language, which microservices already assume.

---

## Answer Key & Explanations

**1. Correct answer: B.**
The module is explicit that events are typically treated as immutable facts — historical records that should not be modified or deleted. A "correction" should be expressed as a new event, not an edit to the existing record; editing history in place breaks the audit-log property that makes events useful in the first place.

**2. Correct answer: B.**
The module directly states that an event can be generated even if it's never consumed, and that many applications producing events don't know whether those events are ever consumed. Treating "no current consumer" as inherently a bug misunderstands that producers and consumers are intentionally decoupled — a consumer might simply not exist yet.

**3. Correct answer: C.**
Events can be persisted indefinitely and consumed as many times as necessary, including by multiple services in parallel — this is exactly what allows a newly added consumer to process historical events and an existing consumer to reprocess after a bug fix.

**4. Correct answer: B.**
This is the "spider web" of point-to-point communication described in the module: each service must know how to reach every downstream service, which introduces coupling. Inserting an event intermediary removes the need for a producer to know anything about its consumers, breaking that direct dependency.

**5. Correct answer: C.**
The module states plainly that it isn't necessary for a producer to know anything about the services consuming its events — the only shared requirement between producer and consumer is agreement on the event's format.

**6. Correct answer: B.**
An event intermediary and an ESB are not the same thing, even though both sit "between" services. The module's broader narrative (carried over from the SOA discussion) frames the ESB as a centralized routing/transformation layer that became a bottleneck because every integration change had to flow through it and its owning team. An event intermediary exists specifically to decouple producers from consumers without recreating that kind of centralized bottleneck.

**7. Correct answer: B.**
The module ties auditing directly to the immutability property: a log of immutable events provides a timed, ordered record of every change to an application's state, which is exactly what an audit needs. Deleting events after consumption (C) would destroy this property, and synchronous calls (D) have nothing to do with providing an auditable history.

**8. Correct answer: B.**
The module states that a centralized event service can help control access to services and data by requiring authentication and authorization at the event service itself — giving you one place to enforce access control across a distributed, event-based application instead of reimplementing it everywhere.

**9. Correct answer: B.**
Because producers and consumers are decoupled and only need to share an event's format, a new consumer (the fraud-detection service) can be added purely by having it start reading the relevant event stream — no changes to the existing order-processing producers are required. This is explicitly called out as an extra benefit of the decoupling.

**10. Correct answer: B.**
In a synchronous chain, a service's health depends on the health of everything it calls, directly or indirectly, so a failure in Z can cascade back through Y to X. In an event-driven design, events are produced asynchronously without waiting for a response, and events destined for an unhealthy service can be replayed or redelivered once it recovers — this is exactly the resilience benefit the module describes.

**11. Correct answer: A.**
Because events are persisted (not just delivered once and discarded), events sent to a service while it was unhealthy can be replayed or redelivered once it comes back online — this is the specific mechanism the module names for surviving a temporary service outage.

**12. Correct answer: B.**
Continuously asking "is there anything new?" is the polling model, and the module states this typically increases network I/O and introduces unnecessary delay in processing. Push-based messaging — where the consumer is notified automatically — is the preferred alternative described in the module.

**13. Correct answer: B.**
Push-based messaging means consumers are automatically notified when there are events to consume, and events are efficiently routed to consumers — the consumer does not need to repeatedly ask, which is exactly what distinguishes push from polling.

**14. Correct answer: B.**
Decoupling removes the need for producers and consumers to know about each other's identity, implementation, or location, but it does not remove all shared context: the module is explicit that a producer or consumer is only required to know the format of a specific event. That shared format is the one thing that still has to be agreed upon.

**15. Correct answer: B.**
The module frames event-driven architecture as a direct response to a problem introduced by microservices themselves — point-to-point communication between many independently deployed services becomes an unmanageable spider web. An event intermediary is the fix for exactly that microservices-specific coupling problem, which is why the two patterns are presented together rather than as unrelated ideas.
