# Module 9 — Introduction to Microservices: Practice Questions

This set covers monolithic applications, Service-Oriented Architecture (SOA) and the Enterprise Service Bus (ESB), microservices, when to start with a monolith vs. microservices, and the benefits and challenges of a microservices architecture. This module is Module 1 of the "Service Orchestration and Choreography on Google Cloud" course.

The questions are weighted toward the distinctions that actually trip people up on the real exam: what actually broke down in SOA, when a monolith is the *right* starting point, and which "benefit" of microservices is really a trade-off in disguise.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A team describes their five-year-old application as "one codebase containing the UI, business logic, and data access, backed by a single relational database, where a change in one area frequently breaks something unrelated." What architectural style are they describing, and what is the specific cause of the fragility?

A. A monolith; the fragility comes from tight coupling between parts of a single, ever-growing codebase.
B. Service-Oriented Architecture; the fragility comes from ESB misconfiguration.
C. Microservices; the fragility comes from too many independently deployed services.
D. A monolith; the fragility comes from having too many databases.

**2.** A team adopted Service-Oriented Architecture (SOA) specifically to reduce complexity in their service code. A year later, they report that overall system complexity didn't go away — it just moved somewhere else, and that "somewhere else" is now the single biggest bottleneck to shipping any change. Where did the complexity go, and why did it become a bottleneck?

A. It moved into the relational database schema, which now requires a DBA for every deploy.
B. It disappeared — SOA fully eliminates integration complexity by design.
C. It moved into the Enterprise Service Bus (ESB) integrations, typically owned by one central team, so nearly every application or service change required ESB work and competed for that team's time.
D. It moved into the client applications, which now must implement their own message routing.

**3.** In an SOA deployment, a developer needs to change how their service's messages are routed through the shared Enterprise Service Bus (ESB). Another team objects, warning this could destabilize their own application. Why is this a realistic concern in SOA specifically (as opposed to microservices)?

A. It isn't realistic — ESB routing changes are always isolated to a single service.
B. Because the ESB is a shared, centralized integration point; changing an integration for one application can affect other applications that route through the same ESB.
C. Because SOA applications share a single codebase, so any change requires a full redeploy of everything.
D. Because ESBs do not support routing changes at all without a full platform upgrade.

**4.** Two engineers debate whether their new project should be built as microservices from day one. One argues "microservices are strictly more modern, so we should always start there." The team, however, doesn't yet have deep expertise in the business domain they're building for. What does the module's guidance actually recommend here?

A. Start with microservices regardless — expertise can be developed after the service boundaries are drawn.
B. Start with SOA as a middle ground between monolith and microservices.
C. The decision is irrelevant — the module states architecture style has no bearing on domain expertise.
D. Start with a monolith, and design it modularly, since designing service boundaries is one of the hardest parts of a new microservices project and is easier once you understand the domain.

**5.** A startup expects its engineering team to grow substantially over the next two years and wants new hires to be able to contribute without first learning an entire large codebase. Which architectural starting point does the module recommend for this specific concern, and why?

A. Microservices, because natural service boundaries let each team member focus on a smaller, bounded piece of the system instead of the entire application.
B. A monolith, because a single codebase is easier to onboard new engineers into as a whole.
C. SOA, because the ESB automatically documents the entire system for new hires.
D. Neither — team growth has no bearing on architecture choice according to the module.

**6.** In a microservices architecture, the Orders, Products, and Reviews domains each have their own database and are called through their own API. A teammate assumes this is functionally identical to SOA, just with a different name. What is actually different?

A. Nothing — microservices and SOA are the same pattern under different marketing terms.
B. Microservices route every call through a shared Enterprise Service Bus, exactly like SOA.
C. Microservices are a decentralized approach: there is no shared ESB middleware — services call each other's APIs directly, and each service owns its own database.
D. Microservices require all services to share one central database, unlike SOA.

**7.** A team switches from a monolith to microservices and is surprised that a single business operation, which used to complete in a few milliseconds, now takes noticeably longer even though no individual service got slower. What's the most likely architectural explanation?

A. Microservices always run on slower hardware than monoliths.
B. The operation now involves multiple calls across the network between services rather than in-process calls within a single application, and network calls are dramatically slower than in-process calls — this compounds when many calls are chained together.
C. This is a bug — properly configured microservices should always be faster than a monolith.
D. The database itself must have been misconfigured during the migration.

**8.** A team building microservices lets each team pick its own programming language and framework, and different services end up written in different languages running on different platforms. A skeptic worries this will break the system. Why does this generally work fine in a microservices architecture?

A. It doesn't work — all microservices must be written in the same language to interoperate.
B. It only works if every service is written in the same language as the ESB.
C. It works only because Kubernetes automatically translates between languages at the network layer.
D. Services interoperate through their API interface (typically HTTP), so the language, framework, or platform used inside a service is invisible to the services calling it.

**9.** During a traffic spike, one microservice (checkout) needs significantly more compute capacity than the others (search, recommendations), which are running at normal load. In a monolith, you'd have to scale the entire application to handle checkout's spike. What's the microservices-specific advantage here?

A. Each microservice can be scaled independently, so you can dedicate more resources to checkout specifically instead of over-provisioning the entire application.
B. Microservices always run at a fixed capacity regardless of traffic, so this scenario cannot occur.
C. This advantage is unique to SOA and does not apply to microservices.
D. Independent scaling requires abandoning APIs in favor of direct database access.

**10.** An organization moves from 3 monolithic applications to roughly 80 microservices. Six months later, their operations team says they're overwhelmed just keeping builds, tests, and deployments running smoothly across all of them. What does the module identify as the underlying cause and the necessary mitigation?

A. This is unrelated to the architecture change — it's a hiring problem.
B. Microservices should never exceed 10 services for this exact reason.
C. Having many more deployable entities creates a much greater operational burden; automated builds, testing, and deployment are vital to keeping this manageable.
D. The fix is to consolidate all 80 services back into a single ESB to reduce operational surface area.

**11.** A security engineer notices that as the number of microservices grew, logging formats, authorization checks, and reporting became inconsistent across teams, making audits much harder than they were with the old monolith. What underlying challenge of microservices does this illustrate?

A. Microservices make security strictly worse with no way to mitigate it.
B. With many independently developed services, maintaining consistent logging, reporting, security, and authorization across all of them becomes an explicit challenge that requires deliberate effort.
C. This only happens if services are written in different programming languages.
D. Security concerns are irrelevant in a microservices architecture since each service is isolated.

**12.** A new engineer draws a diagram of how requests flow between an organization's 60 microservices and describes it as "an unreadable spider web" — it's nearly impossible to tell which services depend on which. What does the module say is the root cause, and is this inevitable?

A. It's inevitable — any system with more than 10 services always becomes unreadable.
B. It only happens when services are written in different languages.
C. It's a sign the team should merge all services back into one monolith immediately.
D. It results from poorly designed inter-service communication; the module frames this as a real risk to manage through deliberate design, not an unavoidable outcome of using microservices.

**13.** A QA lead complains that unit tests for each individual microservice are fast and simple, but verifying that the *whole system* behaves correctly requires standing up something close to the entire production environment. Why does the module say this is expected in a microservices architecture?

A. The distributed nature of microservices generally means that testing the full system requires modeling the entire production deployment, unlike a monolith where components run in the same process.
B. It isn't expected — integration testing should be exactly as simple as unit testing in microservices.
C. This only happens when services share a single database.
D. Integration testing is unnecessary once unit tests pass for each service individually.

**14.** After an incident, an engineer needs to trace a single failed business operation that touched twelve different microservices, each writing its own separate logs. They report this is far harder than debugging the equivalent flow in the team's old monolith. What does the module identify as the reason?

A. Microservices don't produce logs at all, so there's nothing to trace.
B. This is unrelated to architecture — it's purely a tooling budget problem.
C. Because each microservice creates its own logs, tracing a call that spans many microservices is inherently more challenging than tracing calls within a single monolithic process.
D. Debugging is easier in microservices because each service is smaller.

**15.** A leadership team is weighing whether to adopt microservices. They ask: "do the benefits actually outweigh the challenges?" Based on the module's framing, what is the most accurate answer?

A. No — the challenges of microservices always outweigh the benefits, so monoliths are strictly superior.
B. Generally yes, but only with a real commitment to automation and operational excellence; without investing in that tooling, the operational burden of many independently deployable services can overwhelm the benefits.
C. Yes, unconditionally — microservices have no meaningful downsides once adopted.
D. The trade-off is irrelevant — the choice should be based solely on team size, never on operational readiness.

---

## Answer Key & Explanations

**1. Correct answer: A.**
A single codebase combining UI, business logic, and data access over one database, where unrelated changes cause breakage, is the textbook description of a monolith — and the module attributes that fragility specifically to tight coupling within the single application, not to database count or ESB behavior.

**2. Correct answer: C.**
SOA reduced complexity inside individual services, but the module is explicit that this complexity was *shifted*, not eliminated — it moved into ESB integrations, which were typically centrally owned, making ESB work a bottleneck that every application/service team had to compete for.

**3. Correct answer: B.**
Because the ESB is a shared, centralized messaging layer, a routing change for one application's integration can genuinely destabilize other applications that route through the same bus — this centralized-bottleneck risk is precisely what microservices' decentralized, per-service APIs are designed to avoid.

**4. Correct answer: D.**
The module is direct on this point: if you lack expertise in the problem domain, designing service boundaries is one of the hardest parts of a new microservices project, so starting with a modular monolith and migrating later — once you understand the domain — is the recommended path. "Microservices are always the modern default" (A) is the trap.

**5. Correct answer: A.**
The module ties team growth specifically to microservices: natural service boundaries let new team members focus on a smaller part of the system rather than needing to learn a large monolithic codebase end to end.

**6. Correct answer: C.**
Microservices are explicitly framed as a decentralized alternative to SOA — there is no shared ESB middleware component; services call each other directly through their own APIs, and each service owns its own database. Treating microservices as "SOA with a new name" (A) misses this structural difference.

**7. Correct answer: B.**
Calls between microservices cross the network, which the module describes as thousands of times slower than in-process calls inside a monolith; when a single business operation requires many chained microservice calls, that latency compounds and becomes noticeable — even if no individual service is slow.

**8. Correct answer: D.**
Microservices connect through an API interface, so the language, framework, and technology used inside a service are invisible to whatever calls it — this is exactly what lets different teams choose different tech stacks without breaking interoperability.

**9. Correct answer: A.**
Independent scaling is a core microservices benefit: because each service is separately deployable, you can dedicate more resources specifically to the service under load (checkout) instead of scaling — and paying for — the entire application as you would with a monolith.

**10. Correct answer: C.**
More deployable entities directly translate into greater operational burden; the module is explicit that automated builds, testing, and deployment are vital to keeping a large number of microservices healthy and manageable — this isn't a hiring problem or a hard cap on service count, it's an automation requirement.

**11. Correct answer: B.**
The module lists consistent logging, reporting, security, and authorization across many services as an explicit challenge of microservices architectures — it doesn't happen automatically as the service count grows, and it isn't tied to any single programming language.

**12. Correct answer: D.**
The "spider web" of inter-service communication is described as a consequence of *not designing systems well* — it's a real risk that must be actively managed through deliberate design, not an unavoidable fate of any system past a certain service count, and not something limited to polyglot setups.

**13. Correct answer: A.**
Because microservices are distributed, fully verifying system behavior generally requires modeling the entire production deployment — this is fundamentally different from a monolith, where all components run together in the same process and can be tested more directly as a whole.

**14. Correct answer: C.**
Each microservice producing its own separate logs is exactly why tracing a request that spans many services is harder than debugging a single monolithic process, where everything is more naturally visible in one place — this is a direct consequence of the distributed architecture, not a tooling-budget issue.

**15. Correct answer: B.**
The module's own conclusion is that the benefits of microservices generally outweigh the challenges, but conditionally — only with a genuine commitment to automation and operational excellence. Presenting either extreme (A: monoliths are strictly superior, or C: microservices have no downsides) misrepresents that conditional framing.
