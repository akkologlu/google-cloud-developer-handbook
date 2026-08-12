# Module 12 — Introduction to Cloud Run Functions: Practice Questions

This set covers what Cloud Run functions is and how it relates to Cloud Run, the two versions (2nd gen vs. 1st gen), HTTP functions vs. event-driven functions, CloudEvent functions vs. Background functions, language runtime source-code conventions and the entry point, deployment IAM requirements and `gcloud` flags, the three source-location options, and the automatic Cloud Build → Artifact Registry build pipeline. This module is Module 1 of the "Developing Applications with Cloud Run Functions on Google Cloud" course — a course separate from the "Service Orchestration and Choreography on Google Cloud" course covered in modules 09–11.

The questions are weighted toward the distinctions that actually trip people up: why Cloud Run functions isn't a separate product from Cloud Run, why Background functions aren't a free-standing alternative style, the HTTP-vs-event-driven authentication and trigger differences, and what actually happens (automatically) between deploying source code and having a running function.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A developer says: "Cloud Run functions is a completely separate execution product from Cloud Run — they just happen to share a name." Is this accurate?

A. Yes — Cloud Run functions has its own independent execution environment with no relationship to Cloud Run.
B. Yes, but only for HTTP functions; event-driven functions run on a different platform entirely.
C. No — Cloud Run functions (2nd gen) deploys functions as Cloud Run services under the hood, which is also why a function can be moved to plain Cloud Run or Kubernetes if needed.
D. No — Cloud Run functions replaced Cloud Run, which is now a deprecated product.

**2.** A team needs to decide between "Cloud Run functions" and "Cloud Run functions (1st gen)" for a new project. What is the key structural difference between the two?

A. Cloud Run functions (2nd gen) is deployed as a service on Cloud Run and triggered using Eventarc and Pub/Sub; Cloud Run functions (1st gen) is the original version with more limited event triggers and configurability.
B. There is no real difference; the two names refer to identical underlying infrastructure.
C. Cloud Run functions (1st gen) supports more language runtimes than the 2nd gen version.
D. Cloud Run functions (2nd gen) does not support HTTP triggers, only event triggers.

**3.** A function needs to be invoked by an external system calling a webhook, and needs a stable URL to receive requests at. Which function type fits this, and what is true about its default access setting?

A. An event-driven function; it has no default access restrictions of any kind.
B. A Background function; URLs are only assigned to Background functions, never to HTTP functions.
C. An HTTP function; authentication is never available for HTTP functions, only for event-driven functions.
D. An HTTP function; it is assigned a URL to receive requests, and by default requests to it require authentication (unauthenticated access can be enabled at deployment).

**4.** A function should run automatically whenever a new object is uploaded to a specific Cloud Storage bucket, with no external caller involved. Which function type and mechanism does this describe?

A. An HTTP function using a webhook trigger.
B. An event-driven function using an event trigger tied to the Cloud Storage source.
C. A Background function is the only type that can react to Cloud Storage; CloudEvent functions cannot.
D. This requires Cloud Scheduler, since Cloud Run functions cannot react to storage events directly.

**5.** A team is implementing an event-driven function and wants to use the current, industry-standard-based approach that Cloud Run functions supports across all language runtimes. Which implementation style should they use, and what is it built on?

A. CloudEvent functions, based on the CloudEvents industry-standard specification and registered with the Functions Framework, an open-source library that wraps user functions in a persistent HTTP application.
B. Background functions, since they are the newer of the two styles.
C. HTTP functions, since event-driven functions don't have their own implementation style.
D. Either style works identically on every runtime and every generation, with no restrictions.

**6.** Background functions are described in the module as the "older style" of event-driven function. Where are they actually usable?

A. Everywhere — Background functions work identically on both Cloud Run functions and Cloud Run functions 1st gen, on every supported language.
B. Only on Cloud Run functions (2nd gen), and only with .NET, Ruby, and PHP.
C. Only on Cloud Run functions 1st gen, and only with the Node.js, Python, Go, and Java runtimes.
D. Background functions were never actually supported; the module only mentions them as a hypothetical.

**7.** A Node.js Cloud Run function fails to deploy because Cloud Run functions can't locate the function's source. Assuming the default configuration was not overridden, what is the most likely cause?

A. Node.js functions must always be named `main.py` regardless of language.
B. Node.js does not support Cloud Run functions at all.
C. The `package.json` file was deleted, which is unrelated to where Cloud Run functions looks for source code.
D. The source code is not defined in a file named `index.js` at the root of the function directory.

**8.** A developer deploys a function and specifies `--entry-point processOrder`. What does this flag configure, and where must `processOrder` be defined?

A. It sets the deployment region; `processOrder` must be a valid Google Cloud region name.
B. It specifies the function (or class, depending on language) that is executed when the function is invoked; it must be defined in the main source file or root package of the function.
C. It names the Cloud Storage bucket used to stage the source code.
D. It is optional metadata with no effect on which code actually runs.

**9.** A function needs to process requests that individually run for up to 45 minutes, and is invoked directly via HTTP by client applications. Is this within Cloud Run functions' supported limits?

A. Yes — HTTP functions support a run time limit of up to 60 minutes, versus up to 10 minutes for event-driven functions.
B. No — no Cloud Run function can run longer than 10 minutes under any circumstances.
C. No — only event-driven functions can run longer than a few seconds.
D. Yes, but only if the function is deployed as Cloud Run functions (1st gen).

**10.** A high-traffic function instance needs to handle many requests at once without spinning up a large number of separate instances. What concurrency capability does Cloud Run functions provide, and what benefit does the module attribute to it?

A. Cloud Run functions instances can only ever process one request at a time, by design.
B. Concurrency is configurable up to 10 requests per instance, with no effect on cold starts.
C. Concurrency only applies to event-driven functions, never to HTTP functions.
D. Each instance can process up to 1000 concurrent requests, which reduces cold starts and improves overall latency when scaling.

**11.** A user attempting to deploy a Cloud Run function has the Cloud Functions Developer IAM role but the deployment still fails with a permissions error related to the runtime service account. What is most likely missing?

A. Nothing else should be required; the Cloud Functions Developer role alone is always sufficient.
B. The user needs Organization Admin, since function deployment always requires organization-level permissions.
C. The user also needs the Service Account User IAM role on the Cloud Run functions runtime service account.
D. IAM roles are not involved in deploying Cloud Run functions; only API keys are used.

**12.** A team wants to deploy a function's source code from a zip file stored in a Cloud Storage bucket. What are the structural and permission requirements for this to work?

A. Cloud Storage cannot be used as a source location; only local machine and source repositories are supported.
B. The zip's source files must be at the root of the zip file, and the deploying account (1st gen) or the Cloud Run functions service agent (2nd gen) needs permission to read from the bucket.
C. The zip file can have the source nested at any depth, since Cloud Run functions searches the entire archive.
D. No permissions are required to read from the bucket, regardless of generation.

**13.** A team wants to deploy a function from a specific revision of their source repository, using only the code in a subdirectory of that repository. How is this expressed in the `--source` value, and what IAM role does the deploying service agent need on the repository?

A. The source repository path includes `revisions/<revision_name>` for the revision, with `paths/<source_directory_path>` appended to point at the subdirectory; the Cloud Run functions service agent needs the Source Repository Reader (`roles/source.reader`) role on the repository.
B. There is no way to deploy from a subdirectory; the entire repository root must always be used.
C. Subdirectories are specified using the `--entry-point` flag instead of `--source`.
D. Deploying from a source repository never requires any IAM role, unlike the other two source options.

**14.** An architect is choosing a deployment region for a new Cloud Run function and argues the only factor that matters is picking the region physically closest to end users. What does the module say is missing from this reasoning?

A. Nothing — proximity to end users is the only factor the module discusses.
B. Region selection has no effect on pricing under any circumstances.
C. Cloud Run functions can only be deployed to a single global region, so region selection is not a real decision.
D. Latency and availability are the primary considerations, but the location of the other Google Cloud services the app depends on also matters, since services spread across multiple locations can affect both latency and pricing.

**15.** After a developer runs a deploy command pointing at their local source directory, what happens automatically before the function becomes runnable, and which two services are responsible for turning source code into something Cloud Run functions can execute?

A. Nothing happens automatically; the developer must manually build and push a container image themselves.
B. Cloud Scheduler builds the container image, and Cloud Tasks stores it.
C. The source is stored in a Cloud Storage bucket, then Cloud Build automatically builds it into a container image and pushes that image to Artifact Registry, which Cloud Run functions then accesses to run the function.
D. The source is executed directly as uploaded, with no container image ever being built.

**16.** A finance team wants to understand what drives their Cloud Run functions bill. According to the module, what is the pricing model based on?

A. A fixed monthly fee per deployed function, regardless of usage.
B. The number of function invocations, how long the function runs (compute time), and any data transfer fees for outbound network traffic — a pay-as-you-go model.
C. Only the amount of source code storage used in Cloud Storage.
D. A flat per-region licensing fee unrelated to actual invocations.

---

## Answer Key & Explanations

**1. Correct answer: C.**
Cloud Run functions (2nd gen) deploys functions as services on Cloud Run — it is not an independent execution product. This relationship is exactly why a function built on Cloud Run functions can be moved to plain Cloud Run or Kubernetes, since it's already built on Cloud Run's container platform.

**2. Correct answer: A.**
Cloud Run functions (2nd gen, formerly Cloud Functions 2nd generation) deploys functions as Cloud Run services, triggerable via Eventarc and Pub/Sub. Cloud Run functions (1st gen, formerly Cloud Functions 1st generation) is the original version, with more limited event triggers and configurability — not a difference in supported languages or in whether HTTP triggers exist.

**3. Correct answer: D.**
HTTP functions are assigned a URL to receive HTTP(S) requests, and by default those requests require authentication; you can choose to allow unauthenticated requests at deployment time. This makes HTTP functions the right fit for webhook/API scenarios where an external system calls the function directly.

**4. Correct answer: B.**
Automatically reacting to a new object landing in a Cloud Storage bucket, without any external caller, is exactly what event-driven functions with an event trigger are for — Cloud Storage is one of the explicitly supported event trigger sources.

**5. Correct answer: A.**
CloudEvent functions are based on the CloudEvents industry-standard specification and are registered with the Functions Framework, an open-source library that wraps user functions within a persistent HTTP application; they are used by Cloud Run functions across all language runtimes (as well as by Cloud Run functions 1st gen for .NET, Ruby, and PHP).

**6. Correct answer: C.**
Background functions are the older-style event-driven implementation, used specifically by Cloud Run functions 1st gen with the Node.js, Python, Go, and Java runtimes — they are not a generation- or language-agnostic option.

**7. Correct answer: D.**
By default, Cloud Run functions loads Node.js source code from a file named `index.js` at the root of the function directory (a different main file can be specified via the `main` field in `package.json`). `main.py` is the Python convention, not Node.js, and Node.js is a fully supported runtime.

**8. Correct answer: B.**
The `--entry-point` flag specifies the function (or class, depending on the language) that is executed when the Cloud Run function is invoked; your source code must define this entry point in your main file or root package, and you specify it explicitly at deploy time.

**9. Correct answer: A.**
Cloud Run functions supports a run time limit of up to 60 minutes for HTTP functions, and up to 10 minutes for event-driven functions — a 45-minute HTTP function invoked directly is within the supported HTTP limit.

**10. Correct answer: D.**
A single Cloud Run functions instance can process up to 1000 concurrent requests. The module attributes two specific benefits to this: reduced cold starts and improved overall latency when scaling, since fewer new instances need to be spun up to absorb bursts of traffic.

**11. Correct answer: C.**
Deploying Cloud Run functions requires both the Cloud Functions Developer IAM role (or an equivalent) and the Service Account User IAM role on the Cloud Run functions runtime service account — having only the first role without the second produces exactly this kind of permissions failure.

**12. Correct answer: B.**
When deploying from Cloud Storage, the function's source files must be located at the root of the zip file, not nested arbitrarily. Depending on generation, either the deploying account (Cloud Run functions 1st gen) or the Cloud Run functions service agent (Cloud Run functions/2nd gen) needs permission to read from the bucket.

**13. Correct answer: A.**
Deploying from a source repository lets you specify a revision using `revisions/<revision_name>` in the source path, and append `paths/<source_directory_path>` to point at a location other than the repository root. The Cloud Run functions service agent must have the Source Repository Reader (`roles/source.reader`) IAM role on the repository for this to work.

**14. Correct answer: D.**
The module frames latency and availability as the primary considerations for region selection, but explicitly adds that you should also consider the location of the other Google Cloud products and services your app uses, since using services across multiple locations can affect both latency and pricing — proximity to users alone is an incomplete picture.

**15. Correct answer: C.**
When you deploy function source code, it is first stored in a Cloud Storage bucket; Cloud Build then automatically builds that source into a container image and pushes the image to Artifact Registry, entirely automatically with no direct input required from the developer. Cloud Run functions accesses that image from Artifact Registry whenever it needs to run the function.

**16. Correct answer: B.**
The module describes a pay-as-you-go pricing model based on the number of function invocations, how long the function runs (compute time), and any data transfer fees for outbound network traffic — not a flat fee tied to deployment count, storage alone, or region licensing.
