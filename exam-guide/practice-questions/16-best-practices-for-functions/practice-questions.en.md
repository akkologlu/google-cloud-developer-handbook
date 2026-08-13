# Module 16 — Best Practices for Cloud Run Functions: Practice Questions

This set covers implementation best practices (idempotency, always returning an HTTP response, cleaning up background/async work and temp files, avoiding `process.exit()`/`sys.exit()`, handling uncaught exceptions, Error Reporting and default logging), performance and networking (what a cold start actually does, trimming unused dependencies, caching expensive objects in global scope due to instance recycling, lazy initialization, minimum instance counts, persistent connections, Google API clients, and Serverless VPC Access), retry on failure (availability, enabling/disabling it, using it safely for transient failures, and avoiding infinite retry loops), and configuration/scaling best practices (least privilege and function-to-function access, dedicated runtime service accounts, the memory/CPU relationship, timeout, concurrency, and revisions/traffic). This module is Module 5 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 4 (Module 15).

The questions are weighted toward the distinctions that actually trip people up: why idempotency is the precondition for safe retries, why an uncaught exception affects future invocations and not just the current one, why global-scope caching works because of instance recycling, why lazy initialization exists as a counterbalance to that same caching advice, why retry must never be enabled before bugs are fixed through testing, and why enabling concurrency shifts a real responsibility onto the function's own code.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A function occasionally fails partway through and needs to be safely retried without producing duplicate side effects (e.g., double-charging a customer). What property should the function have, and why does the module connect it directly to retry safety?

A. The function should be stateless, since statelessness alone guarantees safe retries regardless of what the function does.
B. The function should be idempotent — producing the same result when called multiple times — which is what makes it safe to retry an invocation that partially failed.
C. The function should use `process.exit()` to guarantee a clean termination before any retry is attempted.
D. The function should disable automatic retry entirely, since retries are inherently unsafe for any function regardless of design.

**2.** An HTTP function occasionally fails to send back a response, and separately, some functions leave asynchronous work still running after the invocation appears to finish. What does the module warn about each of these situations?

A. Neither is a real concern — Cloud Run functions automatically closes out any invocation after a fixed, short timeout with no cost impact, and background work is isolated per invocation.
B. An HTTP function without a response is billed only for a few milliseconds regardless of timeout, and leftover background work is safely discarded without affecting later invocations.
C. Only the missing HTTP response is a problem (billing until timeout); leftover background async work is explicitly supported as a way to warm up the next invocation.
D. An HTTP function that never returns a response may keep running (and being billed) until timeout, and leftover background activity can resume during a later invocation on the same environment, interfering with it — so all asynchronous operations should finish before the function terminates.

**3.** A function writes several files to the temporary directory during each invocation but never removes them. What does the module say this risks, and why?

A. The temp directory is part of an in-memory filesystem, so the files consume the function's available memory and can eventually force a cold start unless explicitly deleted.
B. Nothing — files written to the temporary directory are automatically wiped after every single invocation with no risk to memory.
C. The files are risk-free because the temporary directory is backed by persistent disk storage completely separate from the function's memory allocation.
D. This only risks exceeding a separate disk quota unrelated to the function's memory or performance; cold starts are unaffected.

**4.** A Node.js function calls `process.exit()` at the end of its handler to make sure it terminates cleanly. What does the module say about this practice?

A. This is the recommended way to terminate any function, since it guarantees the fastest possible shutdown and lowest billing.
B. This is required specifically for event-driven functions, though HTTP functions should never do this.
C. Manually exiting with `process.exit()` (or `sys.exit()` in Python) may cause unexpected behavior; the function should instead return implicitly/explicitly (event-driven) or return an HTTP response (HTTP functions).
D. This has no effect either way, since Cloud Run functions ignores explicit exit calls entirely.

**5.** A function occasionally throws an exception that isn't caught anywhere in the code. Besides potentially breaking the current invocation, what does the module say this causes?

A. Nothing beyond the current invocation — uncaught exceptions have no effect on any invocation other than the one that threw them.
B. It permanently disables the function until it is manually redeployed by an administrator.
C. It automatically enables retry for the function, overriding whatever retry setting was previously configured.
D. It forces a cold start on future invocations of the function, in addition to whatever effect it has on the current one — which is why runtime errors should always be handled in code.

**6.** Where do a function's runtime exceptions go, what happens to logs written to `stdout`/`stderr`, and how should an HTTP function versus an event-driven function each respond when an error occurs?

A. Exceptions and logs are both discarded unless a third-party logging library is explicitly integrated; neither function type has a defined error-response pattern.
B. Runtime exceptions are sent to Error Reporting for aggregation, viewing, and notification; `stdout`/`stderr` logs appear automatically in the console with no extra configuration; HTTP functions should report the error and respond with an appropriate HTTP status code, while event-driven functions should report and return an error message.
C. Runtime exceptions go only to Cloud Trace, not Error Reporting; both function types should simply retry silently without reporting anything.
D. `stdout`/`stderr` logs require an explicit logging agent to be manually installed before anything is visible in the console.

**7.** What does a cold start actually do, and what specific practice does the module recommend to reduce the latency it adds?

A. A cold start creates and initializes the function's execution environment, during which imported dependencies are loaded, adding to invocation latency; avoiding loading dependencies the function doesn't use reduces this latency and deploy time.
B. A cold start only affects the function's billing tier, not its latency; reducing it requires increasing the memory allocation, not touching dependencies.
C. A cold start re-runs the function's entire deployment pipeline on every single invocation; it cannot be reduced by any code-level change.
D. A cold start is a Cloud Run functions feature that intentionally throttles traffic; loading more dependencies upfront is the recommended way to reduce its impact.

**8.** A function creates a new database client object inside its handler on every single invocation, which is expensive. What does the module recommend, and why does this work?

A. Switch to creating the client inside a try/catch block instead — exception handling alone eliminates the recreation cost.
B. Nothing can be done — every invocation always runs in a brand-new, unrelated environment, so no value can ever be reused across invocations.
C. Declare the client as a global-scope variable — because a function instance's execution environment is often recycled across invocations, a global-scope value can be reused on subsequent invocations to the same instance without being recomputed.
D. Move the client creation into a separate Cloud Run function and call it synchronously on every invocation instead.

**9.** A function has several expensive global-scope variables, but only some of the function's code paths actually use each one. What does the module recommend, and what problem does this solve?

A. Move every global variable into a Secret Manager secret instead, since Secret Manager resolves values faster than global scope.
B. Consider initializing those global variables lazily, on demand — since initializing global variables always adds cold-start latency, and code paths that never use a particular variable shouldn't pay the cost of initializing it.
C. Delete unused global variables entirely and recompute their values inside the handler on every invocation, regardless of cold or warm start.
D. Nothing needs to change — global variable initialization has no measurable effect on cold-start latency.

**10.** A team wants to reduce cold starts and improve their application's overall latency, independent of any code-level optimization. What configuration does the module recommend?

A. Disable retry entirely, since retry is described as the leading cause of cold starts.
B. Set the function's timeout to the lowest possible value, which the module describes as directly preventing cold starts.
C. Increase the maximum instance count as high as possible, since more maximum instances always means fewer cold starts.
D. Set a minimum number of function instances to be kept ready to serve requests, which reduces cold starts and improves overall performance.

**11.** A function makes frequent outbound HTTP calls to an external URL, calls a Google API on every invocation, and also needs to reach a resource inside a VPC network. What three networking practices does the module recommend?

A. Create and cache persistent HTTP connections in global scope for the external URL, create the Google API service client object in global scope to avoid unnecessary connections and DNS queries, and use a Serverless VPC Access connector with internal DNS/IP addresses for the VPC-internal resource.
B. Open and close a fresh connection for the external URL on every invocation to avoid any risk of stale connections, and route all Google API calls and VPC traffic through the public internet for simplicity.
C. Cache the external HTTP connection only inside the function handler's local scope so each invocation gets a clean copy, and avoid using Serverless VPC Access since it always increases latency.
D. Use only environment variables to configure all three connections, since global scope cannot hold network connection objects of any kind.

**12.** A team wants an HTTP function to automatically retry itself when it fails. Is this possible, and separately, how is automatic retry actually enabled or disabled, and for how long does it retry by default?

A. Yes, automatic retry is available for both HTTP and event-driven functions; it's enabled by default and retries indefinitely until manually stopped.
B. No automatic retry mechanism exists at all in Cloud Run functions, for any function type, under any configuration.
C. Automatic retry is not available for HTTP functions — only for event-driven functions — and even there it's disabled by default; it's enabled with the `--retry` flag on `gcloud functions deploy` (or the console's "Retry on failure" option) and retries for up to 7 days by default.
D. Automatic retry is available for HTTP functions only, is enabled by default, and retries for exactly 24 hours with no way to disable it.

**13.** A team enables retry on an event-driven function that has a bug causing it to fail on every single invocation, without first testing and fixing the bug. What does the module say will happen, and what should have been done first?

A. Retry automatically detects and skips over any function with a persistent bug, so nothing problematic happens.
B. The function is automatically disabled by the platform after its first failed retry, preventing any further cost.
C. Retry is specifically designed to fix bugs in the function's code automatically, so enabling it first is actually the recommended troubleshooting step.
D. Because the function retries continuously until it succeeds, a persistent bug causes it to fail repeatedly without ever succeeding — bugs causing failures should be found and fixed through testing before enabling retry, and an end condition (e.g., discarding events older than a timestamp) should be added to prevent infinite retry loops on persistent failures.

**14.** A team is designing IAM for a multi-function service where a login function needs to call a user-profiles function but should not be able to call a search function, and they haven't yet assigned a dedicated identity to any of the functions. What does the module recommend across these two concerns?

A. Grant every function broad access to every other function for simplicity, and rely exclusively on the default service account even in production, since IAM roles alone are sufficient regardless of identity.
B. Follow least privilege — limit each function's access to the minimum users/service accounts and permissions needed, restrict each function to calling only the specific subset of functions it legitimately needs (login → user-profiles, but not search), and for production, assign each function a dedicated user-managed runtime service account rather than relying on the default one.
C. Grant the Cloud Functions Admin role broadly to all calling functions, since Admin is the only role capable of restricting function-to-function access.
D. Skip IAM configuration entirely for internal function-to-function calls, since the module states such calls are always implicitly trusted within a project.

**15.** A team is choosing a memory setting for a CPU-intensive function and separately wants to avoid premature timeouts. What does the module say about how memory relates to CPU, and how should the timeout be set?

A. The amount of allocated memory corresponds to the amount of allocated CPU, so an under-provisioned memory setting can indirectly starve the function of CPU; timeout should be set slightly higher than the function's actual execution time.
B. Memory and CPU are configured completely independently with no relationship between them, and timeout should always be set to the platform's absolute maximum regardless of execution time.
C. CPU allocation determines memory allocation (the reverse relationship), and timeout should be set to match execution time exactly, down to the millisecond.
D. Memory and CPU are unrelated, and the module recommends disabling timeouts entirely for CPU-intensive functions.

**16.** A team enables concurrency on a Cloud Run function so a single instance can handle multiple requests at once, hoping to reduce cold starts. What default behavior does this change, and what new responsibility does it place on the function's code?

A. By default, a single instance already handles unlimited concurrent requests, so enabling concurrency has no effect on behavior or code requirements.
B. Concurrency changes only billing granularity, not actual request handling; no change to the function's code is ever required.
C. By default, a function instance handles only one request at a time; enabling concurrency (configured via the function's underlying Cloud Run service) lets an already-warmed instance absorb additional concurrent requests, but the function's code must be safe for concurrent execution since Cloud Run functions provides no isolation between requests on the same instance.
D. Concurrency is enabled automatically once a minimum instance count is set, and it requires no code changes because each concurrent request runs in a fully isolated sub-environment.

---

## Answer Key & Explanations

**1. Correct answer: B.**
Writing functions to be idempotent means they produce the same result when called multiple times, which is exactly what makes it safe to retry an invocation that partially failed for some reason.

**2. Correct answer: D.**
An HTTP function must always return an HTTP response, otherwise it may continue executing until timeout and incur charges for that entire time. Separately, no background activity should run after invocation termination — because the CPU is not accessible, and a subsequent invocation on the same environment can cause that background activity to resume and interfere with it, so all asynchronous operations should finish before the function terminates.

**3. Correct answer: A.**
The temporary directory functions write to is part of an in-memory file system; these files consume memory available to the function and sometimes persist between invocations, so explicitly deleting any created files avoids eventually running out of memory and triggering a cold start.

**4. Correct answer: C.**
Manually exiting from functions using `process.exit()` (Node.js) or `sys.exit()` (Python) may cause unexpected behavior; instead, return implicitly or explicitly from event-driven functions, and return HTTP responses from HTTP functions.

**5. Correct answer: D.**
Uncaught exceptions can terminate the function and force cold starts in future function invocations — the effect isn't limited to the invocation that threw the exception, which is why runtime errors and exceptions must always be handled in function code.

**6. Correct answer: B.**
Runtime exceptions emitted from a function are sent to Error Reporting, where they can be aggregated, viewed, and used to trigger notifications. Cloud Run functions includes simple runtime logging by default, so logs written to `stdout` or `stderr` appear automatically in the console. HTTP functions should report the error and respond with an appropriate HTTP status code based on the error type, while event-driven functions should report and return an error message when an exception occurs.

**7. Correct answer: A.**
A cold start creates and initializes a function's execution environment; during this process, any dependencies the function imports are loaded, adding to invocation latency — avoiding loading dependencies the function doesn't use reduces this latency and the time needed to deploy the function.

**8. Correct answer: C.**
A function instance's execution environment from a previous invocation is often recycled; declaring a variable in global scope lets its value be reused on subsequent invocations to that same instance without being recomputed — an approach well suited to caching objects like API client objects that are expensive to recreate on every invocation.

**9. Correct answer: B.**
Initialization of global variables always increases a function's latency during cold-start invocations; if some global variables aren't used on every code path, they should be initialized lazily, on demand, so paths that don't need them don't pay that initialization cost.

**10. Correct answer: D.**
Since function instances scale based on incoming request volume, setting a minimum number of instances to be kept ready to serve requests reduces cold starts and improves the application's overall performance.

**11. Correct answer: A.**
When accessing URLs from functions, create persistent HTTP connections and cache them in global scope to reduce per-invocation connection-setup CPU time and connection-quota risk. To avoid unnecessary connections and DNS queries when communicating with Google APIs, create the Google service client object in global scope. To reach VPC-internal resources without exposing traffic to the internet, use Serverless VPC Access connectors with internal DNS and internal IP addresses.

**12. Correct answer: C.**
Automatic retry is available for event-driven functions only, and is disabled by default. It's enabled with the `--retry` flag on the `gcloud functions deploy` CLI command, or by selecting "Retry on failure" in the Google Cloud console when deploying; with retry enabled, an event is retried repeatedly for up to seven days by default until the function executes successfully or the maximum retry period elapses.

**13. Correct answer: D.**
Because a function is retried continuously until it executes successfully, any bugs causing function failure should be discovered and fixed in the code through testing before enabling retries — and because failures can be persistent, an end condition (such as discarding events older than a specified timestamp) should be included before the function's processing code executes, to prevent infinite retry loops.

**14. Correct answer: B.**
Following the principle of least privilege, access to functions should be limited to the minimum number of users and service accounts, with the minimum permissions needed. Each function should be restricted to sending requests only to the specific subset of other functions it legitimately needs — the module's own example is that a login function should be able to access a user-profiles function but probably not a search function. For production, each function should be given a dedicated identity via a user-managed service account rather than relying on the default one.

**15. Correct answer: A.**
The amount of allocated memory chosen for a function corresponds to an amount of allocated CPU, so memory and CPU are linked rather than independent. Separately, to prevent a function from timing out prematurely, its timeout duration should be specified slightly higher than the function's actual execution time.

**16. Correct answer: C.**
By default, function instances handle only one request at a time. This behavior can be changed for Cloud Run functions so a single instance can handle multiple concurrent requests, configured via a concurrency value on the function's underlying Cloud Run service — this reduces cold starts by letting an already-warmed instance process additional requests. With concurrency enabled, the function's code must be safe to execute concurrently, since Cloud Run functions does not provide isolation between concurrent requests handled by the same instance.
