# Module 10 – Event-Driven Applications

> This is Module 2 of the "Service Orchestration and Choreography on Google Cloud" course (Module 1 was [Introduction to Microservices](../09-introduction-to-microservices/introduction-to-microservices.en.md)). No specific Google Cloud products are named in this lesson yet — it stays at the conceptual/architectural level. Later modules of this course will be added to this handbook as their own numbered modules once their transcripts become available.

---

# Overview

Module 9 ended on a problem: point-to-point communication between microservices tends to turn into an unreadable "spider web," and each service ends up coupled to every downstream service it calls directly.

Event-driven architecture is the pattern that fixes this — by putting an **event intermediary** between services instead of having them call each other directly.

```text
Point-to-point (module 9's problem)

Service A ──→ Service B
    │             │
    └────→ Service C ←────┘

Event-driven (this module's fix)

Service A ──┐              ┌──→ Service C
            ├─→ Event Intermediary ─┤
Service B ──┘              └──→ Service D
```

---

# What Is an Event?

An **event** is a record of something that has happened — an employee logging in, a product being added to a shopping cart.

That sounds obvious, but three attributes matter more than the plain definition:

| Attribute | What it means |
| --- | --- |
| Immutable | An event is a historical record of an occurrence. It should never be modified or deleted after the fact. |
| Independent of consumption | An event can be generated even if it's never consumed. The producer often doesn't know, or care, whether anything is listening. |
| Durable and replayable | An event can be persisted indefinitely and consumed as many times as necessary — a single event can be consumed by multiple services in parallel. |

> These three properties are what make an event fundamentally different from a request in a request/response call. A request expects someone to act on it right away and disappears once handled. An event is a permanent fact that may be read by zero, one, or many consumers, at any point in the future.

---

# From Point-to-Point to an Event Intermediary

As covered in Module 9, point-to-point communication between microservices requires every service to know how to talk to every downstream service — this introduces coupling and can turn into a "spider web" that's hard to reason about.

An event-driven architecture inserts an **event intermediary** between services:

- A service acting as an **event producer** sends events to the intermediary. It doesn't need to know anything about which services will consume those events.
- A service acting as an **event consumer** receives events from the intermediary. It doesn't need to know anything about which service produced them.

```text
Producer → knows only the event format → Event Intermediary → routes to → Consumer(s)
```

> **Exam trap:** Don't confuse an event intermediary with an Enterprise Service Bus (ESB) from Module 9's Service-Oriented Architecture discussion. An ESB is a centralized routing/transformation layer that became a bottleneck precisely because every integration change had to go through it and its owning team. An event intermediary's role is narrower and more decentralized in spirit — it decouples producers from consumers so that neither side needs to know about the other, without becoming the kind of central integration-change bottleneck the ESB did.

---

# Benefits of Event-Driven Applications

## 1. Centralized Auditing and Access Control

A centralized event service simplifies auditing and control for a distributed application:

- A log of immutable events can be used for auditing — it gives you a timed, ordered record of every change to the state of an application.
- Requiring authentication and authorization at the event service lets you control access to your event-based services and data from one place.

## 2. Decoupling

With an event intermediary, producers and consumers are decoupled:

- A service can create an event without sending direct requests to whatever consumes it.
- A service can consume an event without knowing anything about whoever produced it.
- Producers and consumers are only required to agree on the **format** of a specific event — nothing else.
- New event consumers can be added to the application **without modifying any existing services**.

This is what eliminates the point-to-point spider web: every event travels through the intermediary, which routes it to the correct consumer or consumers.

## 3. Resilience Through Asynchronous Processing

Microservices built around synchronous request/response calls have a structural weakness: the health of a service is affected by the health of every service it calls, directly or indirectly. A single failing service can bring down the entire chain.

Event-driven architecture generates events asynchronously, without waiting for a response:

- The system can be designed to survive the temporary loss of a service.
- Events sent to an unhealthy service can be **replayed or redelivered** once that service comes back up.

| | Synchronous request/response | Event-driven (asynchronous) |
| --- | --- | --- |
| Failure behavior | A downed service can cascade failures up the call chain | A downed consumer just falls behind; events wait to be replayed/redelivered |
| Coupling | Caller must know and reach the callee directly | Producer and consumer only share an event format |
| Consumption | Exactly one caller waits for exactly one response | Zero, one, or many consumers can process the same event, in parallel |

---

# Push-Based Messaging vs. Polling

Asynchronous, event-driven services typically use **push-based** delivery rather than polling:

- **Polling** — consumers continuously ask the source "is there new work?" This increases network I/O and adds unnecessary delay before work is picked up.
- **Push-based messaging** — consumers are automatically notified when there's an event to consume; events are routed to consumers efficiently, without the consumer having to ask.

```text
Polling:  Consumer → "anything new?" → Source → "no" → Consumer → "anything new?" → ... (repeat)

Push:     Event Intermediary → notifies → Consumer (only when there's something to deliver)
```

---

# Module Summary

An event is an immutable historical record that can be produced without being consumed, and persisted and replayed indefinitely. Event-driven architecture solves the point-to-point coupling problem from Module 9 by inserting an event intermediary between producers and consumers, so neither side needs to know about the other — they only need to agree on an event's format.

This decoupling delivers three concrete benefits: centralized auditing and access control through an immutable, authenticated event log; the ability to add new consumers without touching existing services; and resilience, because asynchronous processing means one service's failure doesn't automatically cascade, and undelivered events can be replayed once a consumer recovers. Push-based delivery is preferred over polling because it avoids the network overhead and delay of consumers repeatedly asking whether new work exists.

---

# Key Points

- An event is immutable, may never be consumed, and can be persisted and consumed multiple times in parallel.
- An event intermediary decouples producers from consumers — producers don't know who consumes their events, and consumers don't know who produced them.
- An event intermediary is not the same thing as an SOA-style Enterprise Service Bus (ESB) — it exists specifically to avoid becoming that kind of centralized bottleneck.
- Centralizing events simplifies auditing (immutable, ordered log) and access control (auth enforced at the event service).
- New consumers can be added without modifying existing services — this is what removes the point-to-point "spider web."
- Asynchronous, event-driven processing is more resilient than synchronous request/response chains: a failing service doesn't necessarily cascade, and events can be replayed or redelivered.
- Push-based messaging is preferred over polling because it avoids the network I/O and delay overhead of continuously asking for new work.
