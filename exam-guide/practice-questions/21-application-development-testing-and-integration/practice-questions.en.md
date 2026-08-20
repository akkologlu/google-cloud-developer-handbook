# Module 21 — Application Development, Testing, and Integration: Practice Questions

This set covers development and testing (the criteria an app must meet to fit Cloud Run, the container runtime contract for services versus jobs, execution environments, file system and data storage options, Cloud Code, and local testing tools), managing service deployments and revisions (building and deploying containers, Artifact Registry's role and internal image copying, creating/updating services, the eight service configuration components, revision immutability, how traffic migrates to a new revision, and pinning/tagging/splitting traffic), and integrating with Google Cloud services (client libraries and per-service identity, connecting to Memorystore and Cloud Run Integrations, triggering from Pub/Sub, and connecting to Cloud SQL).

The questions are weighted toward the distinctions that actually trip people up: why a job's container must never listen on a port while a service's must, why changing a configuration setting alone creates a new revision even without a new image, why Cloud Run copies a container image into its own storage rather than pulling directly from Artifact Registry every time, why pinning, tagging, and splitting traffic are three different tools rather than one, why the built-in service account is a broader risk than most people assume, and why a Pub/Sub-triggered Cloud Run endpoint doesn't need to be public.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** What criteria must an application meet, in full, to be considered a good fit for Cloud Run?

A. It must serve requests, streams, or events delivered over HTTP, HTTP/2, WebSockets, or gRPC (or execute to completion); not require a local persistent file system; be built to handle multiple simultaneous instances; use no more than 8 CPU and 32 GiB of memory per instance; and be containerized, written in Go/Java/Node.js/Python/.NET, or otherwise containerizable.
B. It only needs to be written in Python, regardless of any other characteristic such as memory usage, file system requirements, or request handling.
C. It must require a local persistent file system and use more than 8 CPU per instance, since Cloud Run is described as designed specifically for high-resource, stateful workloads.
D. It must be designed to run as a single instance only, since the module describes Cloud Run as incompatible with applications built to handle multiple simultaneous instances.

**2.** How does the container runtime contract differ for a Cloud Run service versus a Cloud Run job in terms of what the container must do?

A. Both services and jobs must listen on a port and return an HTTP response, with no distinction made between the two container types anywhere in the runtime contract.
B. A job's container must listen on a port and return an HTTP response, while a service's container must exit with code 0 regardless of whether the request succeeded.
C. A service's container must listen for requests on the correct port and respond within the configured timeout (up to 1 hour, including startup time) or the request ends with a 504 error; a job's container must not listen on a port or start a web server, and must instead exit with code 0 on success or a non-zero code on failure.
D. Neither services nor jobs are required to do anything specific under the runtime contract — the contract only governs which base image can be used.

**3.** What does the module say about implementing transport layer security (TLS) directly inside a Cloud Run container?

A. The module recommends implementing your own TLS termination inside the container for maximum control over encryption settings.
B. The container should not implement transport layer security directly, since TLS for HTTPS and gRPC is terminated transparently by Cloud Run, with requests then proxied to the container as HTTP/1 or gRPC (and HTTP/2 requests handled in cleartext format).
C. TLS is only relevant for jobs, never for services, so services can safely ignore any TLS-related guidance entirely.
D. TLS must be implemented directly in the container for gRPC requests specifically, even though HTTPS requests are handled transparently by Cloud Run.

**4.** What are Cloud Run's two execution environments, and which one can actually be changed?

A. Cloud Run has only a single execution environment, with no distinction between "first" and "second" generation anywhere in the platform.
B. Both execution environments are described as being changeable only for jobs, never for services, which is the reverse of the module's actual guidance.
C. The choice of execution environment is described as being made automatically by Cloud Run based on the programming language of the application, with no manual configuration possible for either services or jobs.
D. First generation and second generation; first generation is the default for services (and can be changed), while second generation is the default for jobs and **cannot** be changed for jobs — you can only choose between the two for services, based on your service's needs.

**5.** What capabilities does the second generation execution environment provide that make it suitable for CPU-intensive workloads or workloads requiring a network file system?

A. The second generation execution environment provides none of these capabilities — it exists solely to reduce cold start times for infrequent-traffic services.
B. The second generation execution environment removes support for all system calls entirely, relying purely on emulation instead of the first generation's full compatibility.
C. Faster CPU performance, faster network performance (especially with packet loss), full Linux compatibility including all system calls/namespaces/cgroups, and network file system support.
D. The second generation execution environment is described as identical in every respect to the first generation, with no distinguishing capabilities of any kind.

**6.** What is the behavior and best use of Cloud Run's in-memory file system, and what's required to use a network file system like Filestore instead?

A. The in-memory file system is writable and consumes the container instance's allocated memory, but data written to it does not persist when the instance is stopped — suitable as a cache or for disposable per-request data; to persist data beyond an instance's lifetime with standard file system semantics via Filestore or a self-managed network file system, you must specify the second generation execution environment when deploying.
B. The in-memory file system is read-only and automatically persists all written data indefinitely, even after the container instance is stopped, with no additional configuration required.
C. Network file systems like Filestore are available by default in the first generation execution environment with no configuration changes required at all.
D. Data written to the in-memory file system is automatically and permanently synced to Cloud Storage without any client library or explicit code being involved.

**7.** What does Cloud Code provide for developers working with Kubernetes and Cloud Run applications?

A. Cloud Code is described as a paid, closed-source product available only as a standalone desktop application entirely separate from any IDE.
B. Cloud Code only supports Kubernetes applications and explicitly excludes any support for Cloud Run applications of any kind.
C. Cloud Code requires a fully deployed production application before it can provide any debugging or sample-based support whatsoever.
D. Cloud Code is a set of plugins for popular IDEs (VS Code, IntelliJ, Cloud Shell) that provides support for the full development cycle of Kubernetes and Cloud Run applications — from creating and customizing a new application from sample templates to running the finished application, with configuration snippets, a tailored debugging experience, and log streaming/viewing support.

**8.** What are the three ways described for testing a container locally before deploying it to Cloud Run?

A. The only supported local testing method is deploying directly to a live, publicly accessible Cloud Run service and observing production traffic.
B. Cloud Code (using its Cloud Run emulator within the IDE), the gcloud CLI (which contains a local development environment that can build from source, run the container locally, and rebuild automatically on source changes), and Docker (using `docker run` with the image URL and listening port, testing at `http://localhost:port/`).
C. Local testing is described as entirely unsupported for Cloud Run applications — every single test must be performed against an already-deployed revision.
D. Local testing can only be performed using a physical, on-premises Kubernetes cluster, with no support described for gcloud CLI or Docker-based local testing.

**9.** What three tools does the module describe for building a container image, and how does Cloud Run's own source-based deployment fit in?

A. Only Docker is described as capable of building a container image; Cloud Build and Cloud Run's source-based deployment are described as nonexistent features.
B. `gcloud run deploy --source` is described as only capable of deploying pre-built container images, never building one from source code, contradicting its actual described purpose.
C. Docker (build locally with a Dockerfile and push with `docker push`), Cloud Build (build on Google Cloud with a Dockerfile or Google Cloud's buildpacks via `gcloud builds submit`, adding the `pack` flag for buildpacks), and Cloud Run itself, where `gcloud run deploy --source` builds the application source with a Dockerfile (if present) or Google Cloud's buildpacks, uploads the resulting image, and deploys it.
D. Cloud Build is described as requiring a physical on-premises server, and Cloud Run's source-based deployment is described as requiring a Dockerfile to be present at all times with no buildpacks fallback.

**10.** Before a container can be deployed to Cloud Run, where must the container image be stored, and what do you do if you normally host images somewhere unsupported?

A. Container images can be deployed directly from any local developer machine's disk without ever being uploaded to any repository at all.
B. Cloud Run is described as requiring container images to be embedded directly into the YAML deployment configuration file itself, rather than referenced by a repository URL.
C. Cloud Run only supports images from a single, fixed public registry controlled entirely by Google, with no support for private repositories or remote repository configuration.
D. The container image must be stored in a repository Cloud Run can access, such as Artifact Registry (Google's recommendation) or Docker Hub, or another public/private registry connected via an Artifact Registry remote repository; if you host images somewhere unsupported, you need to push them to Artifact Registry first (e.g. with `docker push`).

**11.** What kinds of software artifacts can Artifact Registry repositories store, and what happens after Cloud Run pulls a container image from a Docker repository there?

A. Artifact Registry repositories can store container images (Docker repository), Node.js packages (NPM repository), Java packages (Maven repository), and Python packages (PyPI), among others; after Cloud Run pulls the image, it copies and stores it locally in its own internal storage, ensuring large images load just as fast as small ones and insulating the service from the image later being accidentally deleted from Artifact Registry.
B. Artifact Registry can only store container images and explicitly cannot store any other software package type such as NPM, Maven, or PyPI packages.
C. After Cloud Run pulls an image, it discards its own copy immediately and re-pulls directly from Artifact Registry on every single container start, with no local caching of any kind.
D. Artifact Registry repositories are described as being usable only for storage, with no integration to Cloud Build for storing packages and images produced by builds.

**12.** What roles are required to deploy a container image to Cloud Run, and what happens the very first time you deploy a given container image to a new service?

A. No IAM roles or permissions of any kind are required to deploy to Cloud Run — anyone with network access to the API endpoint can deploy without any authorization check.
B. Deploying requires one of the Owner role, the Editor role, or both the Cloud Run Admin and Service Account User roles (or an equivalent custom role); the first deployment of a container image creates both the service and its first revision, and there is only one container image per service.
C. The first deployment of a container image only creates a revision, never a service, requiring the service itself to be created through a completely separate, unrelated process.
D. Deploying a container image requires the Viewer role exclusively, and no other role (including Owner or Editor) is described as sufficient for this purpose.

**13.** What are the general steps to update an application already running on Cloud Run?

A. Delete the existing Cloud Run service entirely and manually recreate every configuration setting from scratch, since revisions are described as not supporting redeployment of any kind.
B. Directly edit the already-deployed, immutable revision in place, since the module describes revisions as editable after creation as long as only the container image is changed.
C. Modify only the YAML file without ever rebuilding or repackaging the container image, since the module states code changes never require a new image to take effect.
D. Modify the application source code, build and package it into a container image, push the image to Artifact Registry, redeploy the container image to the Cloud Run service, and wait for Cloud Run to deploy the changes.

**14.** What are the eight components of a Cloud Run service's configuration, and what happens if you change even one of them without changing the container image?

A. There are only two configuration components — container image URL and request timeout — and changing either one is described as having no effect on revisions at all.
B. The eight components are described as being permanently fixed at the time the service is first created, with no ability to update any of them on subsequent deployments.
C. Container image URL, container entrypoint and arguments, secrets and environment variables, request timeout, concurrency, CPU/memory limits, scaling boundaries, and Google Cloud configuration (service account, connectors); changing any one of these settings results in the creation of a new revision, even if the container image itself doesn't change.
D. Changing a configuration setting without changing the container image is described as having no effect whatsoever; only container image changes are said to create new revisions.

**15.** What does it mean that a Cloud Run service revision is immutable, and what is a revision actually a copy of?

A. Immutable means you can make limited changes to a revision after creation, as long as you don't touch the container image, which contradicts the term's actual meaning in the module.
B. A revision is an immutable copy of the container image and the service configuration; "immutable" means you can't make changes to a revision after it's created — you can only add new revisions to make further updates.
C. A revision is described as a mutable, continuously-updated copy of only the container image, with service configuration stored completely separately and unrelated to revisions.
D. Revisions are described as being automatically deleted and replaced in place whenever any change is made, rather than being added alongside older revisions.

**16.** When a new service revision is created, what happens to traffic and container instances during the transition, before the new revision is confirmed ready?

A. Cloud Run first scales the new revision up to match the current revision's capacity and waits for its container instances to finish starting up; while this happens, the container instances in the current (older) revision continue serving any request traffic to the service.
B. All traffic switches to the new revision the instant it's created, before any of its container instances have finished starting, causing every request to fail until startup completes.
C. The current revision is immediately shut down and all of its container instances are terminated the moment a new revision is created, regardless of the new revision's readiness.
D. Traffic is evenly split 50/50 between the new and current revisions from the very first moment the new revision is created, regardless of whether its instances have started.

**17.** What deployment option lets you perform a gradual rollout of changes to an application by initially sending a new revision zero traffic, and how do you then increase that revision's traffic over time?

A. The `--force-restart` flag, which is described as immediately routing 100% of traffic to a new revision with no gradual option available.
B. The `--no-traffic` option, which configures a new service revision to receive no traffic initially when deployed; you then update the service to specify an incremental percentage value to gradually increase the amount of traffic the new revision receives.
C. The `--max-instances=0` flag, which is described as the only way to prevent any traffic from reaching a revision, permanently, with no path to later increasing it.
D. There is no deployment option described for controlling initial traffic to a new revision — all new revisions are described as always receiving 100% of traffic immediately upon creation.

**18.** What does pinning traffic to a specific service revision accomplish, and how is it configured?

A. Pinning traffic is described as identical to tagging a revision, with the two terms used interchangeably and no functional difference between them anywhere in the module.
B. Pinning traffic permanently deletes all other revisions of the service the moment it's configured, leaving only the pinned revision in existence.
C. Pinning traffic is described as requiring a revision's traffic percentage to be set to 0, which contradicts how the feature is actually meant to keep serving traffic from that revision.
D. Pinning traffic decouples the deployment of a new revision from the migration of request traffic — if you add a new revision after pinning, Cloud Run won't automatically send it traffic; this is useful for rollback or pre-migration testing, and is achieved by setting the pinned revision's traffic percentage to 100.

**19.** What is a tagged revision, and what is its unique URL format?

A. A tagged revision is assigned a tag that lets you access it at a specific URL without serving any traffic — useful for testing and vetting a new revision before it serves traffic; its unique URL is the Cloud Run service's URL with the tag name added as a prefix, e.g. `https://green---hello-xyz-uc.a.run.app` for the tag `green` on service `hello`.
B. A tagged revision automatically begins serving 100% of production traffic the instant the tag is assigned, with no option to test it privately first.
C. Tagged revisions are described as sharing the exact same URL as the service's default endpoint, with no distinguishing prefix or path of any kind.
D. Tags can only be assigned to a service's very first revision and can never be assigned to any subsequent revision created afterward.

**20.** How does traffic splitting work across multiple service revisions, and are traffic routing changes instantaneous?

A. Traffic splitting can only be configured between exactly two revisions at a time, and any transition between splitting configurations is described as completing instantly with all in-flight requests dropped.
B. Traffic splitting percentages are described as fixed permanently at the time a revision is first created and can never be adjusted afterward through any interface.
C. Traffic splitting routes a configurable percentage of requests to each of multiple revisions (configurable via console, gcloud CLI, YAML, or Terraform); traffic routing adjustments are not instantaneous — requests currently being processed continue to completion and are not dropped, potentially being directed to either revision during the transition.
D. Traffic splitting is described as requiring session affinity to be enabled first, with no way to split traffic without it.

**21.** What role does the Cloud Run built-in service account have by default when client libraries use it to call Google Cloud APIs, and what's recommended instead?

A. The built-in service account has the broad Project Editor role, meaning it can call all Google Cloud APIs and has read/write access to nearly all resources in the project; it's recommended to use a per-service identity — a service account with a minimal set of permissions, such as only the Firestore User role if the service only reads from Firestore.
B. The built-in service account has no permissions whatsoever by default, and using it always causes every API call from the service to fail.
C. The built-in service account is described as scoped automatically to only the specific resources the service's code happens to reference, with no broader access possible.
D. Using a per-service identity is described as unsupported for Cloud Run services, with the built-in service account being the only account type client libraries can ever use.

**22.** What are the steps to connect a Cloud Run service to a Memorystore for Redis instance, and how does the Cloud Run Integrations feature simplify this?

A. Memorystore connections are described as requiring no networking configuration of any kind, since Redis instances are always publicly accessible from any Cloud Run service by default.
B. The steps are: determine the Redis instance's authorized VPC network, create a Serverless VPC Access connector in the same region as the Cloud Run service, and attach the connector to that VPC network (then deploy with `--vpc-connector` and Redis host/port environment variables); the Integrations feature automates this entirely, automatically creating a fully configured Redis cache, a new service revision, and the required networking/environment variable configuration.
C. The Integrations feature is described as requiring more manual steps than connecting manually, making it strictly less convenient than the manual Serverless VPC Access process.
D. Connecting to Memorystore is described as only possible using a public IP address, with Serverless VPC Access explicitly unsupported for this specific service.

**23.** How does a Pub/Sub push subscription trigger a Cloud Run service, does the endpoint need to be public, and what happens if the service doesn't acknowledge a message in time?

A. Push subscriptions are described as requiring the target Cloud Run endpoint to always be publicly accessible with no IAM protection possible.
B. There is no acknowledgement deadline described for Pub/Sub messages delivered to Cloud Run — messages are described as never being redelivered under any circumstances.
C. Pub/Sub is described as only able to trigger Cloud Run jobs, never services, contradicting the module's actual description of push subscriptions targeting a service endpoint.
D. A push subscription delivers messages to the service's endpoint as HTTP requests; the endpoint can be protected using IAM and does not need to be public; the service must acknowledge the message by returning a response within 600 seconds (the maximum acknowledgement deadline), or Pub/Sub will redeliver the message, triggering the service again.

**24.** How does a Cloud Run service typically connect to a Cloud SQL instance over a public IP address versus a private IP address, and what best practices does the module recommend for managing the connection?

A. Public and private IP connections to Cloud SQL are described as using exactly the same mechanism, with no distinction made anywhere in the module.
B. There is no connection limit described for Cloud Run services connecting to a Cloud SQL database, and connection pools are described as unnecessary regardless of how many connections a service opens.
C. Over a public IP, Cloud Run connects via the Cloud SQL Auth proxy (using network sockets or a Cloud SQL connector) with encryption provided; over a private IP, the service routes egress traffic through a Serverless VPC Access connector; best practices include storing database credentials in Secret Manager (passed as environment variables or a mounted volume) and using a connection pool to auto-reconnect and limit the number of connections (Cloud Run services are limited to 100 connections per service to a Cloud SQL database).
D. The module recommends hardcoding database credentials directly into the application's source code rather than using Secret Manager, since Secret Manager is described as incompatible with Cloud SQL connections.

---

## Answer Key & Explanations

**1. Correct answer: A.**
To be a good fit for Cloud Run, an application must serve requests, streams, or events delivered over HTTP, HTTP/2, WebSockets, or gRPC (or execute to completion), not require a local persistent file system (an ephemeral local or network file system is fine), be built to handle multiple instances running simultaneously, not require more than 8 CPU and 32 GiB of memory per instance, and either be containerized, be written in Go, Java, Node.js, Python, or .NET, or otherwise be containerizable.

**2. Correct answer: C.**
When running as a Cloud Run service, the container must listen for requests on the correct port and send a response within the configured request timeout setting (max 1 hour, including container startup time) — otherwise the request ends and a 504 error is returned. For Cloud Run jobs, the container must exit with exit code 0 when successfully completed, or a non-zero exit code when failed; because jobs shouldn't serve requests, the container shouldn't listen on a port or start a web server.

**3. Correct answer: B.**
The container should not implement any transport layer security directly, because TLS is terminated by Cloud Run for HTTPS and gRPC — requests are then proxied as HTTP/1 or gRPC to the container. For HTTP/2, the container must handle requests in HTTP/2 cleartext format.

**4. Correct answer: D.**
Cloud Run has two execution environments: first generation (used for services by default, and can be changed) and second generation (used for jobs by default, and cannot be changed for jobs). You can only change the execution environment for services, choosing between the two based on your service's needs.

**5. Correct answer: C.**
The second generation execution environment provides faster CPU performance, faster network performance (especially in the presence of packet loss), full Linux compatibility including support for all system calls, namespaces, and cgroups, and network file system support.

**6. Correct answer: A.**
On Cloud Run, the container has access to a writable in-memory file system; writing to it uses the container instance's allocated memory, and data written to it does not persist when the container instance is stopped — it can be used as a cache or to store disposable per-request data or configuration. To persist data beyond a container instance's lifetime using standard file system semantics via Filestore or another self-managed network file system, you must specify the second generation execution environment when you deploy your service.

**7. Correct answer: D.**
Cloud Code is a set of plugins for popular IDEs (VS Code, IntelliJ, Cloud Shell) that provides IDE support for the full development cycle of Kubernetes and Cloud Run applications, from creating and customizing a new application from sample templates to running the finished application — providing samples, configuration snippets, a tailored debugging experience, and support for log streaming and viewing.

**8. Correct answer: B.**
You can test a container locally using Cloud Code (with its Cloud Run emulator inside the IDE, letting you configure CPU/memory, environment variables, and Cloud SQL connections), the gcloud CLI (which contains a local development environment for emulating Cloud Run that can build a container from source, run it locally, and automatically rebuild upon source code changes — test at `http://localhost:8080/`), or Docker (using the `docker run` command with the container image URL and the port the application listens on — test at `http://localhost:port/`).

**9. Correct answer: C.**
You can build a container image locally with Docker using a Dockerfile (`docker build`, then `docker push` to upload it to a repository), on Google Cloud with Cloud Build using a Dockerfile or Google Cloud's buildpacks (`gcloud builds submit`, adding the `pack` flag to use buildpacks), or via Cloud Run's own source-based deployment, where the `gcloud run deploy` command with the `--source` flag builds your application source code with a Dockerfile (if one is present) or with Google Cloud's buildpacks, uploads the resulting container image to an image repository, and deploys it to Cloud Run.

**10. Correct answer: D.**
Before a container is deployed to Cloud Run, the container image must be stored in a repository that Cloud Run can access — you can use images stored in Artifact Registry (which Google recommends) or Docker Hub, or from other public or private registries by setting up an Artifact Registry remote repository. If you usually host your container images somewhere else, such as an unsupported public or private container registry, you'll need to push them to Artifact Registry first, which you can do with the `docker push` command.

**11. Correct answer: A.**
Artifact Registry is a universal package manager service used to store and manage software artifacts in private repositories, including container images (in a Docker repository) as well as Node.js packages (NPM repository), Java packages (Maven repository), and Python packages (PyPI), among others. After Cloud Run pulls a container image, it copies and stores the image locally in its internal container storage, which is fast and ensures image size doesn't impact container startup time (large images load as quickly as small ones); because Cloud Run copies the image, it won't be an issue if you accidentally delete a deployed container image from Artifact Registry.

**12. Correct answer: B.**
To be able to deploy, you must have one of the Owner or Editor roles, or both the Cloud Run Admin and Service Account User roles (or a custom role that includes the necessary permissions). When you deploy a container image for the first time, Cloud Run creates a service and its first revision — there is only one container image per service.

**13. Correct answer: D.**
To update an application on Cloud Run, you generally: modify your application source code, build and package your application into a container image, push the container image to Artifact Registry, redeploy the container image to the Cloud Run service, and wait for Cloud Run to deploy your changes — when you re-deploy your container image to an existing service, a new revision is automatically created.

**14. Correct answer: C.**
A Cloud Run service's configuration consists of eight components: container image URL, container entrypoint and arguments, secrets and environment variables, request timeout, concurrency, CPU/memory limits, scaling boundaries, and Google Cloud configuration (such as service account and connectors). Changing any configuration setting of the service results in the creation of a new revision, even if there is no change to the container image itself.

**15. Correct answer: B.**
A revision is an immutable copy of the container image and the service configuration. "Immutable" means you can't make changes to a revision after it has been created — you can only add new revisions to make further updates, and Cloud Run creates this immutable copy every time you deploy a change to the service resource.

**16. Correct answer: A.**
After a new service revision is created, Cloud Run first scales up the new revision to match the capacity of the current revision, and waits for the container instances in that revision to finish starting up. While this is happening, the container instances in the current (previous) revision continue to serve any request traffic to the service.

**17. Correct answer: B.**
To perform a gradual rollout of changes to an application, a new service revision can be configured to receive no traffic initially when deployed with the `--no-traffic` option. To gradually increase the amount of traffic received by the new service revision, you can then update the service to specify an incremental percentage value.

**18. Correct answer: D.**
Pinning request traffic to a specific service revision (rather than the latest revision) decouples the deployment of a new revision from the migration of traffic — meaning that if you add a new revision, Cloud Run won't automatically send traffic to it. This is useful if you want to roll back to a previous revision, or want to test a new revision before migrating all request traffic to it, and is achieved by setting the percentage of request traffic to the revision to 100.

**19. Correct answer: A.**
When you deploy a service, you can assign a tag to the new revision that lets you access that revision at a specific URL without it serving traffic — commonly used to test and vet a new revision before it serves any traffic. A tagged revision has its own unique URL: the Cloud Run service's URL with the tag name added as a prefix — for example, tagging a revision `green` on the service `hello` gives the test URL `https://green---hello-xyz-uc.a.run.app`.

**20. Correct answer: C.**
Cloud Run lets you specify which service revisions should receive traffic and the traffic percentages received by each revision, letting you split request traffic between multiple service revisions by specifying a percentage value (configurable via the console, gcloud CLI, a YAML configuration file, or Terraform). Traffic routing adjustments are not instantaneous — when you change the traffic splitting configuration, requests currently being processed continue to completion and aren't dropped, and may be directed to either a new or previous revision during the transition period.

**21. Correct answer: A.**
Client libraries use the built-in service account to transparently authenticate with Google Cloud services; this service account has the Project Editor role, meaning it's able to call all Google Cloud APIs and has read and write access on all resources in the project. It's recommended to use a per-service identity to restrict the APIs and resources a Cloud Run service can access — for example, assigning only the Firestore User IAM role if the service only reads data from Firestore.

**22. Correct answer: B.**
To connect to a Memorystore for Redis instance from a Cloud Run service, you determine the Redis instance's authorized VPC network, create a Serverless VPC Access connector in the same region as the Cloud Run service, and attach the connector to the Redis instance's authorized VPC network — then deploy the service specifying the connector name and environment variables for the Redis instance's host and port. The Integrations feature automates this: it automatically creates a fully configured Redis cache with a configurable memory size, creates a new Cloud Run service revision, and configures the networking and environment variables needed for the service to access the Redis cache.

**23. Correct answer: D.**
Pub/Sub can push messages to the endpoint of a Cloud Run service, where the messages are subsequently delivered to containers as HTTP requests; the endpoint can be protected using IAM and doesn't need to be public. The Cloud Run service must acknowledge the Pub/Sub message by returning a response within 600 seconds (the maximum acknowledgement deadline), otherwise Pub/Sub will redeliver the message, causing the service to be triggered again.

**24. Correct answer: C.**
For a public IP address (Cloud SQL's default), Cloud Run provides encryption and connects using the Cloud SQL Auth proxy, via network sockets or a Cloud SQL connector; for a private IP address, the service can route all egress traffic to the Cloud SQL instance through a Serverless VPC Access connector. Best practices include using Secret Manager to store sensitive database credentials (passed to the application as environment variables or mounted as a volume) and using connection pools in application code, which automatically reconnect broken connections and let you limit the maximum number of connections used by the service — Cloud Run services are limited to 100 connections per service to a Cloud SQL database.
