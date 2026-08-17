# Module 18 — Introduction to Cloud Run and Google Kubernetes Engine: Practice Questions

This set covers Cloud Run fundamentals (what it is, the developer workflow, container-based vs. source-based deployment with Buildpacks, HTTPS and the port 8080 requirement, the 64-bit Linux binary constraint, and the two pricing models), Cloud Run use cases (REST APIs, complex public sites, microservices communication, event processing, and scheduled tasks with Cloud Scheduler), high availability on Cloud Run (immutable revisions and traffic splitting, autoscaling factors, regions/zones, global load balancing, and Knative-based portability), Google Kubernetes Engine fundamentals (what GKE is versus Kubernetes, cluster architecture with the control plane and nodes, zonal vs. regional clusters, and pods), core Kubernetes resources (Deployments, Services, and Volumes), the GKE development/CI-CD workflow, and Container-Optimized OS. This module is Module 2 of the "Developing Containerized Applications on Google Cloud" course, following Module 1 (Module 17: Introduction to Containers).

The questions are weighted toward the distinctions that actually trip people up: why a container's lifetime is only guaranteed while it's handling requests (and what that means for scheduled work), why Cloud Run apps never need to implement TLS themselves, why autoscaling depends on more than just request volume, why a Service's fixed IP exists specifically because pod IPs aren't stable, why a zonal cluster differs from a regional one, and why Container-Optimized OS trades away a package manager and non-containerized workloads for a smaller attack surface.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** What is Cloud Run, and what determines whether an application written in a given programming language can be deployed on it?

A. Cloud Run is a fully managed compute platform that deploys and runs containers directly on Google's scalable infrastructure; if you can build a container image of your application code — written in any language — you can deploy it on Cloud Run.
B. Cloud Run is a source-code hosting service that only accepts applications written in Go, Node.js, Python, Java, .NET Core, or Ruby, and rejects any container image directly.
C. Cloud Run is a virtual machine product where you must manually install and configure a container runtime before any application can be deployed.
D. Cloud Run is a Kubernetes distribution that requires every application to be packaged as a Kubernetes Deployment manifest before it can run.

**2.** After deploying a container to Cloud Run, what do you get, and what does the module mean when it says Cloud Run is "serverless"?

A. You get a static IP address that must be manually attached to a load balancer, and "serverless" means the platform runs entirely without any CPU or memory allocation.
B. You get an on-premises server address that must be manually registered in DNS, and "serverless" means the container never actually executes any code.
C. You get a unique HTTPS URL, and Cloud Run then starts your container on demand to handle requests, ensuring all incoming requests are handled by dynamically adding and removing containers; "serverless" means you as a developer can focus on building your application instead of building and maintaining the infrastructure that powers it.
D. You get a fixed, single container instance that must be manually restarted after every request, and "serverless" means there is no billing for the service at all.

**3.** What's the difference between Cloud Run's container-based and source-based workflows?

A. The container-based workflow is only available for Go and Java, while the source-based workflow is only available for interpreted languages like Python and Ruby.
B. In the container-based workflow you build the container image yourself, deciding exactly what's packaged in it; in the source-based workflow you deploy your source code instead, and Cloud Run uses Buildpacks to build your source and package the application with its dependencies into a container image for you.
C. Both workflows require you to write a Dockerfile — the only difference is which command-line tool you use to submit the build.
D. The container-based workflow deploys directly to Google Kubernetes Engine, while the source-based workflow deploys only to Compute Engine virtual machines.

**4.** What port must a Cloud Run application listen on to handle web requests, and does the application need to implement HTTPS itself?

A. The application must implement its own HTTPS server and manage its own TLS certificate; Cloud Run only forwards raw TCP packets without any protocol awareness.
B. The application must listen on port 443 directly, since Cloud Run passes encrypted HTTPS traffic straight through to the container without any proxy involved.
C. There is no default port — every Cloud Run container must be manually configured with a random port number chosen at deploy time, and the developer must always implement TLS.
D. Cloud Run expects the container to listen on port 8080 (a configurable default) over plain HTTP; a valid TLS certificate and other HTTPS configuration are provisioned by Cloud Run, which handles incoming HTTPS requests, decrypts them, and forwards them to the application — so the application itself doesn't need to provide an HTTPS server.

**5.** What is the one real constraint the module describes on what programming language an application running on Cloud Run can be written in?

A. The application must be written in one of the six source-based languages Cloud Run explicitly names (Go, Node.js, Python, Java, .NET Core, or Ruby), with no exceptions for other languages.
B. The application must avoid any library dependencies whatsoever, regardless of language, since Cloud Run containers are not permitted to include a dependency layer.
C. As long as the application can be compiled to a 64-bit Linux binary and packaged in a container image, it can be developed in any programming language and run on Cloud Run.
D. The application must be written in a language that supports Buildpacks natively, since Cloud Run refuses to run any container image built through a different process.

**6.** How does Cloud Run's default pricing model work, and what does the module say about the alternative pricing model?

A. Under the default model, you only pay for the system resources used while a container is handling requests and while it's starting up or shutting down; Cloud Run also supports an alternative model that charges for the entire container lifecycle, with CPU always allocated even when there are no requests — which may be more economical for most steady-state workloads.
B. Under the default model, you pay a flat monthly fee regardless of usage, and the alternative model charges purely based on the number of deployments performed, not on compute time at all.
C. Under the default model, you pay only once per container image build, and there is no alternative pricing model — Cloud Run charges nothing for actually running the container afterward.
D. Under the default model, billing is based solely on the number of distinct client IP addresses that connect to the service, and the alternative model instead bases billing on the size of the container image.

**7.** A team wants to build a more complex public website, such as an ecommerce site, on Cloud Run. What additional Google Cloud services does the module mention for improving performance and filtering malicious traffic, alongside the backend connections it describes?

A. VPC Service Controls for performance improvement and Binary Authorization for traffic filtering, with no mention of Redis, PostgreSQL, or third-party API connections.
B. Cloud Armor to improve performance and Cloud CDN to filter malicious traffic — the two services' purposes are reversed from what most people assume, and no backend database connections are mentioned as possible.
C. Only Identity-Aware Proxy is mentioned for both performance and traffic filtering, and the module states Cloud Run services cannot connect to any external database.
D. Cloud CDN to improve performance and Google Cloud Armor to filter inbound traffic using content-based rules, with the backend able to connect to a relational database (e.g. PostgreSQL), a Redis store for user sessions, and third-party APIs.

**8.** In a Cloud Run-based microservices architecture, what two communication patterns does the module describe between services, and how does Pub/Sub fit in?

A. Services can only communicate through a shared Cloud SQL database, and Pub/Sub is described as being used exclusively for logging, not for actual service-to-service communication.
B. Services on Cloud Run can communicate directly with each other using REST APIs or gRPC, or asynchronously through Pub/Sub, which is well integrated with Cloud Run using push subscriptions that forward (and optionally authenticate) messages as HTTP requests to a service's endpoint.
C. Services can only communicate asynchronously; direct request/response communication between two Cloud Run services is described as architecturally impossible.
D. Pub/Sub is described as requiring a pull subscription model only, with Cloud Run services needing to poll Pub/Sub continuously since push delivery to Cloud Run isn't supported.

**9.** In the module's event processing example, what triggers the workflow, and what happens after the first Cloud Run service finishes processing?

A. A scheduled Cloud Scheduler job triggers the entire workflow every hour regardless of whether any new files exist, and the first service's only output is a log entry with no further downstream action.
B. An administrator manually triggers each step of the pipeline through the Google Cloud console, and no Pub/Sub messaging is involved at any stage.
C. Uploading an image of a medical scan to Cloud Storage generates a storage event that triggers a Cloud Run service to process and convert the scan; that service then publishes a message to Pub/Sub, which triggers another Cloud Run service to label and watermark the image and a separate VM application (with GPU) to detect anomalies, with both generating output stored back in Cloud Storage.
D. The workflow is triggered by a webhook from a third-party vendor with no Google Cloud service integration, and the module states Cloud Run cannot integrate with Cloud Storage events at all.

**10.** A team wants to run a scheduled task, like generating invoices, and is considering implementing the schedule directly inside a long-running Cloud Run container. What does the module say about this approach, and what does it recommend instead?

A. This approach is recommended as the best practice, since a Cloud Run container's lifetime is described as guaranteed indefinitely regardless of whether it's handling requests.
B. Scheduled tasks are described as entirely unsupported on Cloud Run in any form, with Compute Engine being the only viable alternative.
C. The module recommends implementing the schedule directly in the container specifically because container instances are guaranteed to never be shut down once started, making external triggering unnecessary.
D. A container's lifetime is only guaranteed while it's handling requests, so a task scheduled to run later inside the container risks the container having already been shut down or stopped by then; the module instead recommends using Cloud Scheduler — a fully managed cron job scheduler — to securely trigger the Cloud Run service on a schedule, with the service completing its task within the configured request timeout.

**11.** What happens to a Cloud Run service each time you deploy a change to it, and how can you reduce the impact of a bad deployment?

A. Each deployment creates a new, immutable revision consisting of the container image and the service configuration (environment variables, memory limits, etc.); you can split request traffic by percentage between the new and previous revisions, allowing rollback to a stable revision or a gradual shift to 100% traffic on the new one.
B. Each deployment silently overwrites the previous revision in place, so there is no way to roll back to an earlier version once a new deployment completes.
C. Each deployment requires manually deleting the entire Cloud Run service and recreating it from scratch, with no concept of a "revision" existing at all.
D. Each deployment automatically routes 100% of traffic to the new revision immediately, with no mechanism described for splitting traffic between revisions.

**12.** Beyond the raw rate of incoming requests, what factors does the module say influence how many container instances Cloud Run's autoscaling maintains for a service?

A. Only the total number of registered users of the application, with no other factor described as relevant to the number of container instances.
B. The CPU utilization of existing instances while processing requests (targeting around 60% utilization), the maximum concurrency setting (how many requests can be sent in parallel to a given instance), and the configured minimum and maximum number of container instances.
C. Exclusively the geographic distance between the client and the nearest Google Cloud region, with CPU utilization and concurrency described as having no effect.
D. The number of container images currently stored in Artifact Registry, since the module ties instance count directly to registry storage usage.

**13.** What is a region, what is a zone, and how does Cloud Run use them for high availability?

A. A region and a zone are described as identical, interchangeable terms with no distinction between them in the module.
B. A zone is described as spanning multiple regions, and a region is described as a subset of a single zone — the reverse of the module's actual hierarchy.
C. A region is a set of exactly two zones located on different continents, and Cloud Run is described as always deploying to only one of them at a time.
D. A region is a specific geographic location where Google Cloud resources are hosted, consisting of three or more zones (each a deployment area considered a single failure domain within the region); for high availability, Cloud Run distributes containers over multiple zones in a region, making the application resilient against the failure of a single zone.

**14.** What does the global external Application Load Balancer let you do with multiple regional Cloud Run services, and what benefit does it provide clients around the world?

A. It merges multiple regional Cloud Run services into a single region automatically, eliminating the need for more than one region entirely.
B. It requires every client worldwide to be routed through a single fixed region regardless of their physical location, which the module describes as intentionally increasing latency for load-testing purposes.
C. It exposes a single, global IP address in front of multiple regional Cloud Run services and routes each client's requests to the region closest to them, improving application availability and decreasing latency for clients worldwide.
D. It replaces the need for Cloud Run revisions entirely, since the module describes the load balancer as also handling all traffic-splitting between revisions.

**15.** In what two ways does the module describe Cloud Run applications as being portable, and why does portability matter to a developer?

A. Portability only matters for avoiding vendor lock-in and is never connected to data sovereignty requirements in the module's own examples.
B. Cloud Run applications are portable because containers can run anywhere by nature, and because the Cloud Run platform is API-compatible with Knative (an open source project implementing the same container runtime contract), which enables running the same application in Kubernetes-based environments — useful when an application must run in a region with no Google Cloud presence (for data sovereignty) or when a developer wants to avoid vendor lock-in.
C. Cloud Run applications are portable solely because they can be exported as virtual machine images compatible with any hypervisor, with no mention of Knative anywhere in the module.
D. Portability is described as a limitation unique to Cloud Run that doesn't apply to any other container-based platform, including Kubernetes-based ones.

**16.** A team is deploying a Cloud Run service that can autoscale to a large number of instances very quickly. What two considerations does the module raise about this, separate from the general topic of migrating VM-based workloads?

A. If the service scales to many instances, you'll incur cost for running those instances (which can be limited by setting a maximum instance count), and a rapid scale-up might send more traffic to downstream systems than their throughput capacity can handle, requiring you to understand those systems' capacity when configuring the service.
B. The only consideration described is that autoscaling is entirely free regardless of instance count, and downstream systems are described as always able to absorb unlimited additional traffic without any configuration.
C. The module states that autoscaling must always be disabled for any service that has a downstream dependency of any kind, with no maximum instance setting available as an alternative.
D. The only consideration described is that Cloud Run services cannot scale beyond a single instance under any circumstances, making downstream throughput irrelevant.

**17.** What is Kubernetes, what is Google Kubernetes Engine (GKE), and what advanced cluster management features does GKE provide that you'd otherwise have to build yourself?

A. Kubernetes is a proprietary Google product with no open-source component, and GKE is described as functionally identical to running Kubernetes on any other cloud provider with no added management features.
B. Kubernetes is an open source container orchestration system for automating software deployment, scaling, and management, now maintained by the Cloud Native Computing Foundation (CNCF) after originally being designed by Google; GKE is a fully managed Kubernetes service that adds features like easy cluster creation and management, load balancing, automatic scaling, automatic node software upgrades, automatic node repair, and integrated logging/monitoring through Google Cloud's operations suite.
C. Kubernetes and GKE are described as entirely unrelated products that happen to share similar names, with GKE not actually running Kubernetes underneath.
D. GKE is described as a lower-level product than raw Kubernetes, stripping out cluster management features rather than adding them, to give users more manual control.

**18.** What is the role of a GKE cluster's control plane, and what's the difference between a zonal cluster and a regional cluster?

A. The control plane manages everything running on the cluster's nodes — scheduling container workloads and managing their lifecycle, scaling, upgrades, and network/storage resources — via the Kubernetes API server (`kube-apiserver`); a zonal cluster has a single control plane running in one zone, while a regional cluster has multiple control plane replicas running in multiple zones within a region, for higher availability.
B. A zonal cluster and a regional cluster are described as functionally identical, with the terms used interchangeably and no actual architectural difference between them.
C. The control plane's only function is to run application containers directly, with nodes existing purely as passive storage for container images.
D. The control plane only stores billing information for the cluster and has no role in scheduling or managing workloads, which is instead handled entirely by individual nodes acting independently.

**19.** What is a GKE node, what does the `kubelet` do on that node, and what is a pod?

A. A node is a container image stored in Artifact Registry, `kubelet` is a command-line tool used only by human operators (never running as an agent), and a pod is described as a permanent, non-ephemeral Kubernetes object that survives node failure.
B. A node is a physical server that cannot run virtual machines, `kubelet` is described as a billing agent unrelated to container execution, and a pod is defined as a synonym for an entire cluster.
C. A node is a Compute Engine virtual machine running containerized workloads; `kubelet` is the Kubernetes node agent that communicates with the control plane and is responsible for starting and running containers scheduled on the node; a pod is the smallest deployable compute unit in Kubernetes — a group of one or more containers with shared storage and network resources and a specification for how to run them.
D. A node is a Kubernetes API object with no relationship to any underlying compute resource, `kubelet` runs only on the control plane rather than on nodes, and a pod is described as always containing exactly one container with no possibility of more.

**20.** What does a Kubernetes Deployment define, and what is the relationship between a Deployment and a ReplicaSet?

A. A Deployment's YAML manifest is described as never including a selector label or pod template, containing only the container image name and nothing else.
B. A Deployment is purely an imperative, one-time command with no persistent object created in the cluster, and a ReplicaSet is described as an entirely unrelated, standalone Kubernetes resource with no connection to Deployments.
C. A Deployment can only manage exactly one pod at a time, and the concept of a ReplicaSet is described as deprecated and no longer used by Deployments.
D. A Deployment is a declarative way to create and manage pods and ReplicaSets in Kubernetes, defining a desired state (including the desired number of pod replicas via a ReplicaSet, whose purpose is to maintain a stable set of replica pods running at any given time); the Deployment Controller changes the actual state toward the desired state at a controlled rate.

**21.** Why does a Kubernetes Service need its own fixed IP address instead of clients calling pod IP addresses directly, and what does a Service provide?

A. A Service is a network abstraction that provides a stable (fixed) IP address lasting for the life of the service, along with load balancing across its member pods, selected via a selector; this exists because pods are ephemeral and their IP addresses change as they are deleted and re-created, so it doesn't make sense to use pod IP addresses directly — clients call the service IP instead, and requests are load-balanced across member pods.
B. A Service exists purely for cosmetic naming purposes; pod IP addresses are described as permanently fixed for the pod's entire lifetime, making a Service's IP address technically redundant.
C. A Service is described as providing storage persistence for pods, with IP addressing and load balancing being the sole responsibility of individual pods rather than the Service.
D. A Service can only route traffic to a single, specific named pod and cannot represent or load-balance across a group of pods selected by a label.

**22.** What's the difference between an ephemeral Kubernetes volume type (like ConfigMap or Secret) and a durable one (like PersistentVolume)?

A. Ephemeral and durable volume types are described as functionally identical, with the only difference being which YAML keyword is used to declare them.
B. Ephemeral volume types have the same lifetime as their enclosing pod — created when the pod is created and removed when the pod is terminated — while a volume type backed by durable storage, such as a PersistentVolume, has a lifecycle independent of the pod, with its data preserved when the pod is terminated.
C. Durable volume types are described as being deleted automatically the moment the pod that created them terminates, while ephemeral volume types are described as persisting forever regardless of pod state.
D. Both volume types are described as requiring a pod to be recreated before any container can access them, with no distinction in persistence behavior.

**23.** In the module's described GKE development and deployment workflow, what does Cloud Build do automatically after a change is committed to the source repository, and what happens before a container is promoted to production?

A. Cloud Build only compiles the code locally on the developer's own machine and never touches Artifact Registry or Cloud Storage at any point in the workflow.
B. Nothing happens automatically after a commit — a human must manually trigger every single step of the build, test, and deploy process described in the workflow, including image storage.
C. Cloud Build is described as being responsible only for running tests, with container image building and storage handled by a completely separate, unrelated tool not mentioned in the workflow.
D. Cloud Build rebuilds the application container image, stores it in Artifact Registry, stores build artifacts in a Cloud Storage bucket, runs tests on the container, and calls Google Cloud Deploy to deploy the container to a staging GKE environment; if the build and tests succeed, Cloud Deploy is then used to promote the container to production after approval.

**24.** What is Container-Optimized OS, and what are two of its notable limitations described in the module?

A. Container-Optimized OS is a general-purpose desktop operating system unrelated to Compute Engine, and its main limitation described is that it cannot run Docker containers at all.
B. Container-Optimized OS is described as having no limitations whatsoever compared to a general-purpose Linux distribution, making it suitable for every possible workload without exception.
C. Container-Optimized OS is a Google-maintained operating system image for Compute Engine VMs, based on the open source Chromium OS project and optimized for running Docker containers; two of its notable limitations are that it doesn't include a package manager (so software can't be installed directly on an instance) and that it doesn't support running non-containerized applications.
D. Container-Optimized OS is described as fully supported on any cloud provider or on-premises environment, with its only limitation being a lack of automatic security updates.

---

## Answer Key & Explanations

**1. Correct answer: A.**
Cloud Run is a fully managed compute platform that lets you deploy and run containers directly on top of Google's scalable infrastructure. If you can build a container image of your application code — written in any language — you can deploy that application on Cloud Run.

**2. Correct answer: C.**
Once you've deployed your container, you get a unique HTTPS URL. Cloud Run then starts your container on demand to handle requests, and ensures that all incoming requests are handled by dynamically adding and removing containers. Cloud Run is serverless, meaning that as a developer you can focus on building your application rather than building and maintaining the infrastructure that powers it.

**3. Correct answer: B.**
If you build the container image yourself (container-based workflow), you decide exactly which files are packaged in it. If you use the source-based approach instead, you deploy your source code rather than a container image; using Buildpacks, Cloud Run then builds your source and packages the application along with its dependencies into a container image for you.

**4. Correct answer: D.**
Cloud Run expects your container to listen on port number 8080 to handle web requests — a configurable default you can change if that port is unavailable to your application. Cloud Run provisions a valid TLS certificate and other configuration to support HTTPS requests, and it handles incoming requests, decrypts them, and forwards them to your application — you don't need to provide an HTTPS server yourself.

**5. Correct answer: C.**
One major advantage of Cloud Run is that it runs containers, meaning you can develop your applications in any programming language and run them on Cloud Run, as long as they can be compiled to a 64-bit Linux binary and packaged in a container image.

**6. Correct answer: A.**
The Cloud Run pricing model is unique in that you only pay for the system resources used while a container is handling requests, and when it's starting up or shutting down. Cloud Run also supports a pricing model that charges for the entire container lifecycle, with CPU always allocated to container instances even when there are no requests to the application — this model may be more economical for most steady-state workloads.

**7. Correct answer: D.**
For a more complex public website like an ecommerce site on Cloud Run, you could enable Cloud CDN to improve performance and add Google Cloud Armor to filter malicious inbound traffic using content-based policies. In the backend, you can connect with a relational database, a Redis store for user sessions, and third-party APIs.

**8. Correct answer: B.**
Services on Cloud Run can communicate with each other using REST APIs or gRPC for direct request/response, or send and receive asynchronous messages with guaranteed delivery using Pub/Sub. Pub/Sub is well integrated with Cloud Run using push subscriptions, which forward and optionally authenticate messages as HTTP requests to the endpoint of a Cloud Run service.

**9. Correct answer: C.**
In the module's example, when an image of a medical scan is uploaded to Cloud Storage, a Cloud Run service is triggered to process the scanned image and convert it into a modern format. That service then pushes a message to Pub/Sub, which triggers another Cloud Run service to label and watermark the converted image, and a separate VM application (with a GPU) to detect anomalies in the scan data — both generate output that is stored back in Cloud Storage.

**10. Correct answer: D.**
The limitation of running a scheduled job in the container itself is that a container's lifetime is only guaranteed while it's handling requests — if you schedule tasks on a container to run later, the container might be shut down or stopped by the time the task has to run. The module recommends using Cloud Scheduler, a fully managed cron job scheduler, to securely trigger a Cloud Run service on a schedule, noting that the service must complete its task within the configured request timeout.

**11. Correct answer: A.**
On Cloud Run, each deployment of your container image to a service creates a new, immutable revision consisting of the container image and the service configuration (settings such as environment variables and memory limits). You can reduce the impact of request processing failures by splitting request traffic between the new and previous revisions, letting you roll back to a previous stable revision or gradually send 100% of traffic to the new one.

**12. Correct answer: B.**
In addition to the rate of incoming requests, the number of container instances is impacted by the CPU utilization of existing instances while they're processing requests (with a target of 60% utilization), the maximum concurrency setting, and the minimum and maximum number of container instances setting.

**13. Correct answer: D.**
A region is a specific geographical location where Google Cloud resources are hosted, and a region has three or more zones, each a deployment area considered a single failure domain within the region. For high availability, Cloud Run distributes containers over multiple zones in a region, making the application resilient against the failure of a zone.

**14. Correct answer: C.**
Cloud Run integrates with the Google Cloud global external Application Load Balancer, which lets you expose a single, global IP address in front of multiple regional Cloud Run services. The load balancer routes requests from a client to the region closest to them, improving application availability and decreasing latency for clients worldwide.

**15. Correct answer: B.**
Applications on Cloud Run are portable in two ways: Cloud Run uses containers, which can run anywhere, making applications inherently portable; and the Cloud Run platform is API-compatible with Knative, an open source project implementing the same container runtime contract, enabling serverless applications to run in Kubernetes-based environments. Portability matters for use cases such as running in a region with no Google Cloud presence (for data sovereignty) or avoiding vendor lock-in.

**16. Correct answer: A.**
If you deploy a service that scales up to many container instances, you'll incur costs for running those containers — you can limit this by setting the maximum number of container instances. Separately, if your service scales up to many instances in a short period, downstream systems might not be able to handle the additional traffic load, so you need to understand their throughput capacity when configuring the service.

**17. Correct answer: B.**
Kubernetes is an open source container orchestration system for automating software deployment, scaling, and management; originally designed by Google, it's now maintained by the Cloud Native Computing Foundation (CNCF). GKE is a fully managed Kubernetes service providing advanced cluster management features including easy cluster creation and management, load balancing, automatic scaling, automatic node software upgrades, automatic node repair, and logging/monitoring through Google Cloud's operations suite.

**18. Correct answer: A.**
The control plane manages everything that runs on all of a cluster's nodes — it schedules container workloads and manages their lifecycle, scaling, and upgrades, as well as network and storage resources, communicating via the Kubernetes API server (`kube-apiserver`). A zonal cluster has a single control plane running in one zone, while a regional cluster has multiple replicas of the control plane running in multiple zones within a given region, for higher availability.

**19. Correct answer: C.**
Nodes are Compute Engine virtual machines that run containerized applications and other workloads. A node runs the services needed to support its containers, including the runtime and the Kubernetes node agent (`kubelet`), which communicates with the control plane and is responsible for starting and running containers scheduled on the node. A pod is the smallest deployable compute unit in Kubernetes — a group of one or more containers with shared storage and network resources and a specification for how to run them.

**20. Correct answer: D.**
A Deployment is a declarative way to create and manage pods and ReplicaSets in Kubernetes, defining a desired state for the pods in a cluster. It defines a ReplicaSet specifying the desired number of pod replicas, whose purpose is to maintain a stable set of replica pods running at any given time. The Deployment Controller changes the actual state of the deployment to the desired state at a controlled rate.

**21. Correct answer: A.**
A Kubernetes Service is a network abstraction that provides a stable endpoint for a group of pods selected via a selector, with a fixed IP address lasting for the life of the service and load balancing across its member pods. Because pods are ephemeral, their IP addresses change as they're deleted and re-created, so it doesn't make sense to use pod IP addresses directly — clients call the service IP address instead, and their requests are load-balanced across the pods that are members of the service.

**22. Correct answer: B.**
Ephemeral volume types have the same lifetime as their enclosing pod — they're created when the pod is created and removed when the pod is terminated. Volume types backed by durable storage, such as a PersistentVolume, exist independently of the pod, and their data is preserved when the pod is terminated.

**23. Correct answer: D.**
When a change is committed to the source repository, Cloud Build rebuilds the application container image, stores the image in Artifact Registry, stores any build artifacts in a Cloud Storage bucket, runs tests on the container, and calls Google Cloud Deploy to deploy the container to a staging environment containing a GKE cluster. If the build and tests are successful, Cloud Deploy is used to promote the container to a production environment after approval.

**24. Correct answer: C.**
Container-Optimized OS is an operating system image for Compute Engine VMs, maintained by Google and based on the open source Chromium OS project, optimized for running Docker containers. Among its limitations: a package manager isn't included, so you can't install software packages directly on an instance, and it doesn't support execution of non-containerized applications.
