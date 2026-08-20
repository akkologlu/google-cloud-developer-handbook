# Module 19 — Fundamentals of Cloud Run: Practice Questions

This set covers Cloud Run fundamentals beyond the earlier overview: the service-vs-job distinction, HTTPS support and the responsibility split between Cloud Run and your code, the 64-bit Linux binary requirement, jobs/tasks/executions/array jobs, invocation methods (HTTPS, gRPC, WebSockets, Pub/Sub, Cloud Scheduler, Cloud Tasks, Eventarc), the resource model (service, revision, container instance), the full container lifecycle (starting, serving requests, idle, shutting down, stopped) including where container images actually come from, autoscaling internals (the internal load balancer, scale to zero, cold starts/request queuing, minimum and maximum instances, maximum concurrency), and access control (Google Cloud's API model, IAM policies/bindings/roles, making a service public, and network ingress settings plus Serverless VPC Access).

The questions are weighted toward the distinctions that actually trip people up: why a container image is pulled from Cloud Run's internal storage on every start rather than from Artifact Registry, why an idle container is throttled but still costs nothing, why Cloud Run essentially never stops a container serving requests except for two specific exceptions, why minimum instances still costs money despite being idle, why the default max-instances configuration differs from the platform's instance quota, and why IAM authorization and network ingress settings are independent layers you can combine.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** What's the difference between running code as a Cloud Run service versus as a Cloud Run job?

A. A service is used to run code that responds to web requests or events and runs continuously, while a job is used to run code that performs work and quits when that work is done — both run in the same environment and can use the same Google Cloud integrations.
B. A service always requires a Kubernetes cluster underneath it, while a job runs directly on a bare Compute Engine VM with no container involved.
C. A job responds to web requests indefinitely, while a service performs a single unit of work and then terminates — the reverse of their actual purposes.
D. Services and jobs are two names for the exact same underlying resource, and choosing between them has no effect on how the code executes.

**2.** What infrastructure does a Cloud Run service provide for handling HTTPS, and what is your own application code actually responsible for?

A. Your application code must implement its own TLS termination and certificate management; Cloud Run only forwards raw encrypted bytes without decrypting anything.
B. Cloud Run only supports plain HTTP with no HTTPS capability at all, so encryption must be handled entirely by an external, separately configured proxy.
C. Cloud Run provides the infrastructure needed to run a reliable HTTPS endpoint — provisioning a valid TLS certificate and endpoint on a unique `*.run.app` subdomain (or a custom domain), and handling, decrypting, and forwarding incoming requests, while also supporting WebSockets, HTTP/2, and gRPC; your responsibility is simply to make sure your code listens on a TCP port and handles HTTP requests.
D. HTTPS support requires manually uploading your own certificate files to Cloud Storage before every deployment, with no automatic provisioning available.

**3.** What is the one real requirement on what programming language an application deployed to Cloud Run can be written in?

A. Applications must be written in a language with first-class Google Cloud client library support, or they cannot be containerized for Cloud Run at all.
B. As long as it can be compiled to a 64-bit Linux binary and packaged in a container image, an application can be developed in any programming language and run on Cloud Run.
C. Applications must avoid any system packages or library dependencies whatsoever, regardless of the language used.
D. Applications must be limited to 32-bit binaries, since Cloud Run's underlying infrastructure doesn't support 64-bit execution.

**4.** How does a Cloud Run job relate to tasks and executions, and what is an Array job?

A. A job execution succeeds the moment its first task finishes, regardless of the state of any other tasks running in parallel.
B. A task can run multiple container instances simultaneously, while a job execution is limited to running exactly one task at a time.
C. Jobs, tasks, and executions are unrelated Cloud Run concepts with no structural relationship to one another, and Array jobs are a deprecated feature no longer available.
D. A job consists of one or more independent tasks executed in parallel during a given job execution, with each task running one container instance; all tasks in an execution must complete successfully for the execution to succeed (timeouts and retries can be configured to handle failures), and a job that runs multiple identical container instances is known as an Array job — useful for processing many files from Cloud Storage in parallel, for example.

**5.** A team needs to pick invocation methods for three needs: high-performance internal microservice-to-microservice communication with high data loads, asynchronous message delivery with guaranteed delivery between services, and securely triggering a service on a recurring schedule similar to a cron job. Which methods fit each need?

A. HTTPS for all three cases, since gRPC, Pub/Sub, and Cloud Scheduler are described as redundant with plain HTTPS requests and offer no distinct advantage.
B. WebSockets for all three cases, since the module describes WebSockets as the only invocation method Cloud Run actually supports without additional configuration.
C. gRPC for the first need (since it uses protocol buffers, up to seven times faster than REST calls, well suited to internal microservices and high data loads), Pub/Sub push subscriptions for the second (asynchronous messages with guaranteed delivery, forwarded as HTTP requests to the service endpoint), and Cloud Scheduler for the third (a fully managed cron job scheduler that securely triggers a service on a schedule).
D. Eventarc for all three cases, since Eventarc is described as a superset that replaces the need for gRPC, Pub/Sub, and Cloud Scheduler entirely.

**6.** What is the main resource in Cloud Run, and how does it relate to Google Cloud regions and zones?

A. The main resource in Cloud Run is a service. Each service is located in a specific Google Cloud region, and while services are a regional resource, their container instances can start in any zone within that region — for redundancy, services with high traffic and many instances are spread across multiple zones so the service keeps serving requests even if one zone has issues.
B. The main resource is the container instance itself, which is described as having no relationship to any particular region or zone.
C. The main resource is a job, and services are described as merely an optional wrapper around a job for HTTP-triggered use cases.
D. The main resource is a global object that exists identically and simultaneously across every Google Cloud region with no single home region.

**7.** What does a Cloud Run revision consist of, and why does the module say revisions are immutable?

A. A revision consists only of a container image, with no environment configuration of any kind bundled with it.
B. A revision can be edited in place after creation, as long as only environment variables (not the container image) are changed.
C. A revision is created only the first time a service is ever deployed, and no new revision is ever created by subsequent deployments to that same service.
D. A revision consists of a specific container image along with environment settings such as environment variables, memory limits, or the concurrency value; once created, a revision cannot be modified — deploying a different container image or changing configuration to the same service instead creates an entirely new revision, and requests are automatically routed to the latest healthy revision.

**8.** What handles requests to a specific Cloud Run service revision, and how does this relate to a job's execution structure?

A. Requests to a revision are handled directly by the Google Cloud console with no container instance involved at any point.
B. A container instance handles requests to a service revision (and can receive many requests at once, subject to the concurrency setting); analogously, for jobs, a job execution starts all of a job's tasks, and each task runs one container instance.
C. A single, permanently running container instance handles every revision across every service in a project, shared globally.
D. Job executions and container instances are entirely unrelated concepts, and a job's tasks are described as never running inside a container of any kind.

**9.** What are the relevant states in a Cloud Run container's lifecycle, and what four steps does the "starting" phase involve?

A. The lifecycle has no defined starting phase at all — Cloud Run is described as instantly ready to serve requests the moment a container image is deployed, with zero startup latency under any circumstances.
B. The only two lifecycle states are "Running" and "Not Running," with no distinction made between starting, idle, or shutting-down behavior.
C. The states are: Starting, Serving requests, Idle, Shutting down, and Stopped. Starting begins when Cloud Run pulls a container image and ends when the container starts serving requests, and involves four steps: materializing the container's root file system from the image, running the entrypoint program (your application), continuously probing port 8080 to check if the app is ready (configurable, with support for HTTP/TCP/gRPC startup and liveness probes), and forwarding incoming requests once the app accepts TCP connections.
D. The four steps of starting are, in order: shutting down, stopping, restarting, and re-materializing — a cycle that repeats before any request can be served.

**10.** When Cloud Run pulls a container image, what are the two distinct events this can happen for, and what are the two different sources it pulls from in each case?

A. Cloud Run always pulls directly from Artifact Registry for both deploying a new image and starting a new container, with no separate internal storage layer involved at any point.
B. Container images in Cloud Run are described as being compiled into the platform itself ahead of time, so no pulling from any registry or storage ever occurs after the very first deployment of any Cloud Run service in a project.
C. Cloud Run only pulls a container image once per project, permanently caching it for every future service regardless of which image is actually deployed.
D. When you deploy a container image for the first time, Cloud Run pulls and copies it from Artifact Registry into its own internal storage (optimized so large images load just as fast as small ones); when Cloud Run later starts a new container, it pulls the image from that internal storage instead — this also insulates the service from Artifact Registry failures or an accidentally deleted image.

**11.** What are the defining properties of an idle Cloud Run container, and what limitations does this state impose?

A. An idle container does not serve requests, does not incur charges, and has its CPU throttled to nearly zero (making the app run very slowly); it can be shut down at any time (though a lifecycle hook allows graceful shutdown), background tasks can't be reliably performed while throttled (use Cloud Tasks instead), and network requests to third parties are likely to fail.
B. An idle container continues serving requests exactly as fast as when it's active, incurs full billing charges at all times, and can never be shut down once started under any circumstances.
C. An idle container is described as identical in every respect to a stopped container, with no meaningful distinction between the two states.
D. An idle container immediately loses all its in-memory state the instant it becomes idle, even before any shutdown signal is sent.

**12.** When an idle container starts handling a request again, how quickly does it become fully responsive, and how does the minimum instances setting relate to this?

A. There is a mandatory, fixed 30-second delay every single time an idle container transitions back to serving requests, regardless of any configuration.
B. When a container transitions from idle to serving requests, Cloud Run unthrottles its CPU and returns full access immediately — the application and users won't notice any lag; to handle traffic spikes and minimize cold starts, Cloud Run may keep some instances idle for up to 15 minutes, and the minimum instances setting ensures Cloud Run always keeps a certain number of instances ready to serve requests.
C. Once a container becomes idle, it can never transition back to serving requests — a brand new container must always be started from scratch for every subsequent request.
D. The minimum instances setting is described as having no effect on idle-to-serving transitions whatsoever, only affecting billing and nothing related to latency.

**13.** If a container is idle and Cloud Run decides to shut it down, what happens by default, and what can an application do to shut down gracefully instead?

A. There is no way for an application to influence shutdown behavior at all — Cloud Run always forcibly terminates idle containers with no signal sent beforehand.
B. Cloud Run always waits exactly 60 seconds before shutting down any idle container, whether or not the application handles any signal.
C. Graceful shutdown is triggered by a SIGKILL signal that the application can catch and delay indefinitely, with no time limit imposed by the platform.
D. By default, a container just disappears when shut down; but if the application handles the SIGTERM signal, it gets 10 seconds to clean up before removal — for example, closing open TCP connections, file descriptors, and database connections, flushing buffers with batched data, and writing a log entry to help with later debugging.

**14.** Under what specific circumstances does Cloud Run stop a container that's actively serving requests, and what happens to in-flight requests when that occurs?

A. Cloud Run stops a serving container on a fixed schedule every hour regardless of application behavior, and in-flight requests are always completed successfully beforehand with no impact.
B. Cloud Run stops a serving container the moment CPU utilization exceeds 60%, with all in-flight requests automatically redirected to a pre-warmed backup container with zero downtime.
C. Under normal circumstances, Cloud Run never stops a container while it's serving requests; the two exceptions are if the application exits (e.g., due to an application code error) or if the container exceeds its memory limit (512 MiB by default, configurable up to 32 GiB) — if a container stops while handling requests, all in-flight requests are terminated and fail with an error, and new requests might have to wait while a replacement container starts.
D. Cloud Run stops a serving container whenever a new revision is deployed, immediately and unconditionally terminating every in-flight request tied to any older revision.

**15.** What component distributes incoming requests across a service revision's container instances, and what factors, beyond raw incoming request rate, affect how many instances Cloud Run maintains?

A. Requests are distributed entirely at random with no load balancing component involved, and the only factor affecting instance count is the total number of registered users of the project.
B. An internal Application Load Balancer distributes requests across the instance pool, adding instances when all are busy and shutting some down as demand drops; beyond request rate, instance count is also affected by CPU utilization of existing instances while processing requests (targeting around 60% utilization), the maximum concurrency setting, and the minimum/maximum instance count settings.
C. A single, manually configured DNS round-robin record distributes all requests, and instance count is fixed permanently at deployment time with no runtime adjustment possible.
D. Distribution is handled exclusively by a third-party load balancer that must be manually installed inside every container image, and Cloud Run itself plays no role in instance count decisions.

**16.** What does "scale to zero" mean for a Cloud Run service, and why is it described as economically attractive?

A. If there are no incoming requests to a service, even the last remaining container instance is shut down — this is scale to zero, and it's economically attractive because you don't pay for container instances that are idling; a new instance starts on demand when a new request arrives.
B. Scale to zero means the service's HTTPS endpoint is permanently deleted after a period of inactivity, requiring a completely new deployment to restore it.
C. Scale to zero means the service's maximum instance count is permanently capped at zero, preventing it from ever serving any request again until manually reconfigured.
D. Scale to zero is described as a legacy feature that has been fully replaced by mandatory minimum instances on every Cloud Run service today.

**17.** What happens to the first few requests that arrive right after a service has scaled to zero, and what is this phenomenon commonly called?

A. Those requests are silently and permanently dropped with no queuing behavior of any kind, requiring the client to manually retry from scratch.
B. Those requests queue while the first container instance starts — this is commonly known as a "cold start."
C. Those requests are automatically rejected with an HTTP 500 error, and Cloud Run recommends against ever allowing a service to scale to zero for this reason.
D. Those requests bypass the container entirely and are served directly by the internal load balancer without ever reaching application code.

**18.** What does setting a minimum number of instances actually change about billing, given that those instances may be sitting idle?

A. Minimum instances are described as being billed identically to instances actively serving requests, at the exact same full CPU rate regardless of idle status.
B. Setting minimum instances retroactively refunds all prior charges incurred while the service was scaling from zero, as a billing credit.
C. Minimum instances are entirely free of charge under all circumstances, since Cloud Run is described as never billing for any instance kept idle for any reason.
D. Idle instances kept running because of the minimum instances feature still incur billing costs, even though they aren't serving requests — the feature trades that ongoing cost for reduced latency by avoiding cold starts.

**19.** What is the default maximum number of container instances a Cloud Run service is configured to scale out to, and how does that differ from the platform's overall instance quota?

A. By default, Cloud Run services are configured to scale out to a maximum of 100 instances — this is your service's configuration setting, distinct from the platform's default quota cap of 1,000 instances per service (for which you can request a quota increase if needed).
B. The default maximum instance configuration and the platform's instance quota are described as being exactly the same fixed number with no distinction between them.
C. There is no default maximum instance configuration at all — every Cloud Run service must have this value explicitly set before it can be deployed, or the deployment fails.
D. The platform's instance quota is lower than the default maximum instance configuration, making the quota the binding constraint in virtually every real deployment.

**20.** What is the default and maximum value for the maximum concurrent requests per instance setting, and under what circumstances should a team consider lowering it to 1?

A. The default is 1 request per instance with no way to increase it, making high-throughput services architecturally impossible on Cloud Run.
B. The default is unlimited, and the setting exists only to lower a value that has no meaningful ceiling in practice.
C. The default is 80 concurrent requests per instance (configurable up to a maximum of 1,000); a team should consider setting it to 1 if each request uses most of the container's available CPU or memory, the application code isn't designed to handle multiple requests at the same time, or the code relies on global state that can't be shared across multiple requests.
D. The default is 1,000 and cannot be lowered below 500 under any configuration, regardless of the application's concurrency safety.

**21.** How does the module describe Google Cloud as a platform, and what is deploying a container image to Cloud Run actually an example of?

A. Google Cloud is described as a collection of APIs that let you create and manage virtual resources — accessed via the web console, the gcloud CLI, Terraform, or client libraries — and deploying a container image (e.g. running `gcloud run deploy`) is itself an example of an API call, in this case made to the Cloud Run Management API at `run.googleapis.com`.
B. Google Cloud is described as a fixed set of pre-built virtual machine images with no programmable interface of any kind, and deploying a container image is described as a purely manual, offline hardware provisioning process.
C. Google Cloud is described as accessible exclusively through Terraform, with the gcloud CLI, console, and client libraries described as deprecated legacy tools no longer functional.
D. Deploying a container image is described as bypassing all Google Cloud APIs entirely, communicating directly with physical server hardware without any API layer involved.

**22.** What is an IAM policy binding, and how do member, role, and permission relate to one another in IAM's model?

A. A permission directly grants access without any role or binding involved, making roles and policy bindings purely decorative concepts with no functional effect.
B. A policy binding binds a member (identity) to a single role, and a member can have multiple policy bindings (and therefore multiple roles) within an IAM policy; a role contains a set of permissions that lets the member identity perform specific actions on a resource — for example, the Pub/Sub Publisher role includes the `pubsub.topics.publish` permission.
C. A policy binding is a one-time action that, once used, cannot be referenced again for future authorization checks of any kind.
D. Roles and permissions are identical concepts under IAM, used interchangeably with no structural difference, and a member can never hold more than one role at a time.

**23.** What role must be granted, and to which member, to make a Cloud Run service publicly accessible without authentication — and what are the two ways to control access to individual services versus an entire project?

A. Making a service public requires disabling IAM entirely for the project, and access to individual services versus the whole project is described as controlled identically with no distinction available.
B. Making a service public requires granting the Owner role to a randomly generated service account, and access can only ever be controlled at the level of an entire Google Cloud organization, never per-project or per-service.
C. Making a service public means removing the service from Cloud Run entirely and re-deploying it on a separate, unauthenticated platform, since IAM cannot be configured to allow public access under any circumstances.
D. You grant the Cloud Run Invoker role (`roles/run.invoker`) to the `allUsers` member (e.g. via `gcloud run services add-iam-policy-binding my-service --member="allUsers" --role="roles/run.invoker"`, or the `--allow-unauthenticated` deploy flag); to control access to an individual service or job you add/remove principals with `gcloud run services/jobs add-iam-policy-binding`/`remove-iam-policy-binding`, while to control access across all services and jobs in a project you use project-level IAM with `gcloud projects add-iam-policy-binding`.

**24.** What are Cloud Run's three network ingress settings, and how does Serverless VPC Access differ from configuring ingress?

A. Ingress settings and Serverless VPC Access are described as the exact same feature under two different names, configured with a single identical command.
B. There is only one ingress setting available on Cloud Run ("All"), and Serverless VPC Access is described as a deprecated, non-functional feature.
C. The three ingress settings are: "All" (least restrictive — allows all requests, including directly from the internet), "Internal" (most restrictive — only internal HTTP(S) load balancer, allowed VPC Service Controls perimeter resources, same-project/perimeter VPC networks, and same-project/perimeter services like Cloud Tasks/Eventarc/Pub/Sub/Workflows), and "Internal and Cloud Load Balancing" (Internal's allowances plus the external HTTP(S) load balancer, but still not directly from the internet). These control inbound access to the service itself, while Serverless VPC Access instead connects a Cloud Run service or job outbound to a VPC network (using internal DNS/internal IP addresses via a connector) to reach resources like VM instances or Memorystore without exposing that traffic to the internet — the two mechanisms are independent and can be combined.
D. Ingress settings only apply to Cloud Run jobs, never to services, and Serverless VPC Access is described as replacing the need for ingress settings entirely.

---

## Answer Key & Explanations

**1. Correct answer: A.**
On Cloud Run, code can run continuously as a service or as a job. Services are used to run code that responds to web requests or events, while jobs are used to run code that performs work and quits when the work is done. Both run in the same environment and can use the same integrations with other Google Cloud services.

**2. Correct answer: C.**
A Cloud Run service provides the infrastructure required to run a reliable HTTPS endpoint: it provisions a valid TLS certificate and an HTTPS endpoint on a unique subdomain of `*.run.app` (with custom domains configurable), and it handles incoming requests, decrypts them, and forwards them to the application, while also supporting WebSockets, HTTP/2, and gRPC. The application's own responsibility is simply to listen on a TCP port and handle HTTP requests.

**3. Correct answer: B.**
Running containers is a major advantage of Cloud Run — applications can be developed in any programming language and run on Cloud Run, as long as they can be compiled to a 64-bit Linux binary and packaged in a container image.

**4. Correct answer: D.**
A job consists of one or multiple independent tasks that are executed in parallel in a given job execution, with each task running one container instance. All tasks in a job execution must complete successfully for the execution to be successful, and timeouts plus a specified number of retries can be configured to handle task failures. Jobs that run multiple identical container instances are known as Array jobs — for example, to process multiple image files from Cloud Storage at the same time.

**5. Correct answer: C.**
gRPC is a good fit for high-performance, high-data-load communication between internal microservices, since it uses protocol buffers that are up to seven times faster than REST calls. Pub/Sub push subscriptions let you send and receive asynchronous messages between services with guaranteed delivery, forwarding them as HTTP requests to a Cloud Run service's endpoint. Cloud Scheduler is a fully managed cron job scheduler that can securely trigger a Cloud Run service on a schedule, similar to using cron jobs.

**6. Correct answer: A.**
The service is the main resource of Cloud Run, and each service is located in a specific Google Cloud region where Cloud Run is available. Services are a regional resource, but their container instances can start in any zone within that region; for redundancy, services with high traffic and many container instances are spread across multiple zones, so the service continues serving requests even if Cloud Run experiences issues in one zone.

**7. Correct answer: D.**
Each deployment of an application container image to Cloud Run creates a service revision, which consists of a specific container image along with environment settings such as environment variables, memory limits, or the concurrency value. Revisions are immutable — once created, a revision cannot be modified; deploying a different container image (or changing configuration) to the same service instead creates a new revision, and requests are automatically routed as soon as possible to the latest healthy service revision.

**8. Correct answer: B.**
Each service revision that receives requests is automatically scaled with the number of container instances needed to handle all these requests, and a container instance can receive many requests at the same time (subject to the concurrency setting). Analogously, when a job is executed, a job execution is created in which all job tasks are started, and each task runs one container instance.

**9. Correct answer: C.**
The relevant states of a container on Cloud Run are Starting, Serving requests, Idle, Shutting down, and Stopped. The Starting phase begins when Cloud Run pulls a container image and ends when the container starts to serve requests, and starting a container requires four steps: Cloud Run creates the container's root file system by materializing the container image, runs the entrypoint program (the application), continuously probes port 8080 to check whether the application is ready (configurable, with support for HTTP/TCP/gRPC startup health checks and liveness probes), and forwards incoming web requests to the container once the application starts accepting TCP connections.

**10. Correct answer: D.**
There are two distinct events when Cloud Run pulls a container image, from two different sources: when you deploy a container image for the first time, Cloud Run pulls and copies it from Artifact Registry, then stores it in its own internal storage (optimized so large container images load just as fast as tiny ones); every time Cloud Run starts a new container after that, it pulls the image from that internal storage instead. Because Cloud Run copies the image this way, it also insulates the service from failures in Artifact Registry or from a deployed container image being accidentally removed from Artifact Registry.

**11. Correct answer: A.**
An idle container does not serve requests and does not incur charges, but has its CPU throttled to nearly zero, meaning the application runs very slowly. It can be shut down at any time, though a lifecycle hook is available for graceful shutdown. Because the CPU is throttled, background tasks can't reliably be performed on the container (Cloud Tasks can be used to schedule tasks instead), and network requests to third parties are likely to fail while the container is idle.

**12. Correct answer: B.**
When a container handles a request after being idle, it goes from the idle state to serving requests; during this transition, Cloud Run unthrottles the container's CPU and returns full access to the container immediately, with no lag noticeable to the application or its users. To handle traffic spikes and minimize cold starts, Cloud Run may keep some instances idle for a maximum of 15 minutes, and the minimum instances setting ensures Cloud Run always keeps a certain number of container instances ready to serve requests.

**13. Correct answer: D.**
By default, a container just disappears when it's shut down. However, an application can be built to handle the SIGTERM signal, which warns it that shutdown is imminent and gives it 10 seconds to clean up before the container is removed — for example, closing database connections or flushing buffers with data, closing open TCP connections/file descriptors/database connections more generally, flushing any batched data, and writing a log entry to help with later debugging.

**14. Correct answer: C.**
Cloud Run never stops a container while it serves requests under normal circumstances. However, a container can stop suddenly if the application exits (for instance, due to an error in the application code) or if the container exceeds its memory limit (512 MiB by default, configurable up to 32 GiB). If a container stops while it's handling requests, all in-flight requests are terminated and fail with an error, and while Cloud Run starts a replacement container, new requests might have to wait.

**15. Correct answer: B.**
Requests to a service revision are distributed by an internal Application Load Balancer across the group of container instances — if all instances are busy, Cloud Run adds additional ones, and as demand decreases, it stops sending traffic to some instances and shuts them down. In addition to the rate of incoming requests, the number of container instances is affected by the CPU utilization of existing instances while processing requests (targeting around 60% utilization), the maximum concurrency setting, and the minimum and maximum number of container instances setting.

**16. Correct answer: A.**
If there are no incoming requests to a service, even the last remaining container instance will be shut down — this is commonly referred to as scale to zero. This feature is attractive for economic reasons because you don't pay for container instances that are idling; a new container instance starts on demand when a new request is sent to the service.

**17. Correct answer: B.**
The first few requests that come in after a service has scaled to zero will queue while the first container instance starts — this is known as a "cold start."

**18. Correct answer: D.**
With minimum instances set, Cloud Run keeps at least that many instances running even if they're not serving requests (idle). Idle instances kept running using the minimum instances feature still incur billing costs — the module notes minimum instances exists specifically to reduce the latency of the service when there would otherwise be no instances available.

**19. Correct answer: A.**
By default, Cloud Run services are configured to scale out to a maximum of 100 instances, which you can set or update for your service. Separately, the number of container instances in a Cloud Run service is limited to 1,000 instances by default as a platform quota — if more are needed, a request for a quota increase can be submitted.

**20. Correct answer: C.**
By default, each Cloud Run container instance can receive up to 80 requests at the same time, and this can be increased to a maximum of 1,000. Setting the maximum concurrency to 1 should be considered if each request uses most of the available container CPU or memory, if the application code isn't designed to handle multiple requests at the same time, or if the application code relies on global states that can't be shared by multiple requests.

**21. Correct answer: A.**
Google Cloud can be viewed as a collection of APIs that let you create and manage virtual resources, accessed through the web console, the gcloud CLI, Terraform, or client libraries from application code. Deploying a container image to Cloud Run is itself an example of an API call: running `gcloud run deploy` via the gcloud CLI makes an API call to the Cloud Run Management API at `run.googleapis.com`.

**22. Correct answer: B.**
An IAM policy binding binds a member (an identity) to a single role, and a member can have multiple policy bindings within an IAM policy, giving that member more than one role. A role contains a set of permissions that allows the member identity to perform specific actions on Google Cloud resources — for example, the Pub/Sub Publisher role includes the `pubsub.topics.publish` permission, which provides access to publish messages to a topic.

**23. Correct answer: D.**
To make a Cloud Run service publicly accessible, you assign the IAM Cloud Run Invoker role to the `allUsers` member type on the service (via `gcloud run services add-iam-policy-binding` with `--member="allUsers" --role="roles/run.invoker"`, or the `--allow-unauthenticated` flag on `gcloud run deploy`). To control access to an individual service or job, you add or remove principals with the desired role using `gcloud run services/jobs add-iam-policy-binding`/`remove-iam-policy-binding`; to control access to all services and jobs in a project at once, you use project-level IAM with `gcloud projects add-iam-policy-binding`.

**24. Correct answer: C.**
Cloud Run provides three ingress settings at the service level: "All" (the least restrictive default, allowing all requests including those sent directly from the internet), "Internal" (the most restrictive, allowing only requests from the internal HTTP(S) load balancer, resources allowed by a VPC Service Controls perimeter containing the service, VPC networks in the same project/perimeter, and same-project/perimeter Google Cloud services like Cloud Tasks, Eventarc, Pub/Sub, and Workflows), and "Internal and Cloud Load Balancing" (everything Internal allows, plus the external HTTP(S) load balancer, but still not directly from the internet). These control inbound access to the service. Serverless VPC Access instead connects a Cloud Run service or job outbound to a VPC network — using internal DNS and internal IP addresses through a connector — to reach resources such as VM instances or Memorystore instances without exposing that traffic to the internet; ingress settings and Serverless VPC Access are independent mechanisms that can be used together.
