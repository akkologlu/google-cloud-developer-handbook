# Module 15 — Integrating with Cloud Databases: Practice Questions

This set covers connecting Cloud Run functions to Memorystore (Redis/Memcached, fully managed capabilities, the Serverless VPC Access connection flow with its region-matching requirement), environment variables (storage, scope, how they're set and read), Firestore (the document/collection data model, trigger event types, the Native-mode-only restriction, document path rules, snapshots, `DocumentReference` versus the Firebase Admin SDK), Secret Manager (secrets versus secret versions, the `secretmanager.secretAccessor` role, volume-mount versus environment-variable version behavior, cross-project access), and BigQuery Remote Functions (the `CLOUD_RESOURCE` connection, generation-dependent Invoker roles, and the permissions needed to invoke a remote function from a query). This module is Module 4 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 3 (Module 14).

The questions are weighted toward the distinctions that actually trip people up: why the Serverless VPC Access connector's region must match the function's region rather than the database's, why Firestore triggers only work in Native mode, the difference between a secret and a secret version, why a volume-mounted secret picks up rotation immediately while an environment-variable secret doesn't, and why the BigQuery connection's Invoker role depends on the function's generation.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** The module states that Cloud Run functions can integrate with several Google Cloud databases, but goes deep on only two of them, with a third covered as supplementary reading. Which are those three, and how are they treated?

A. All six databases (Firestore, Cloud SQL, Spanner, Bigtable, BigQuery, Memorystore) are covered in equal depth, with no supplementary reading.
B. Memorystore and Firestore are covered in depth; BigQuery is covered separately as supplementary reading material on BigQuery Remote Functions.
C. Only Cloud SQL and Spanner are covered in this module; Firestore and Memorystore are referred to documentation instead.
D. The module covers Bigtable and BigQuery in depth, with Memorystore mentioned only as a passing reference.

**2.** A team evaluating Memorystore wants to know what operational tasks it automates and which other Google Cloud services it integrates with. What does the module say?

A. Memorystore automates only patching; provisioning, replication, and failover must still be handled manually by the customer.
B. Memorystore requires the customer to manage its own Redis cluster nodes; it only provides a managed network path to them.
C. Memorystore integrates with BigQuery for querying cache contents and with Cloud Build for automated cache warming.
D. Memorystore automates provisioning, replication, failover, and patching, and integrates with IAM for secure access and Cloud Monitoring for service monitoring and alerting.

**3.** A developer needs to distinguish between the two caching engines Memorystore supports. How does the module describe Redis versus Memcached?

A. Redis is an open-source in-memory data structure store usable as a database, cache, message broker, and streaming engine; Memcached is an open-source distributed memory object caching system.
B. Redis is a distributed memory object caching system; Memcached is a data structure store that can also serve as a message broker.
C. Redis and Memcached are simply two different brand names for the identical underlying technology offered by Memorystore.
D. Redis is a Google-proprietary caching engine; Memcached is the only genuinely open-source option Memorystore supports.

**4.** A Redis instance runs in `us-east1`, and the function that will connect to it is deployed in `europe-west1`. In which region should the Serverless VPC Access connector be created?

A. `us-east1`, matching the Redis instance's region, since the connector is dedicated to reaching that instance.
B. Either region works equally well, since the connector automatically operates across all regions.
C. `europe-west1`, matching the function's deployment region — the connector is then attached to the Redis instance's authorized VPC network separately.
D. A third, unrelated region should be chosen to avoid conflicts between the function's and the database's regions.

**5.** After the Serverless VPC Access connector reaches a `Ready` state, what must the function deployment specify, and how is the function subsequently invoked?

A. Only the connector's name; the Redis host and port are auto-discovered at runtime with no configuration needed, and the function is invoked via a POST request.
B. The connector's path/name plus environment variables for the Redis host IP address and port; the function is invoked by sending an HTTP GET request to its URL endpoint.
C. A hardcoded Redis connection string compiled into the function's source code; invocation happens automatically on a fixed schedule.
D. A Secret Manager reference to the Redis credentials; invocation requires a signed ID token issued specifically for Redis access.

**6.** Where are a Cloud Run function's environment variables stored, and what is their scope?

A. They are stored in Secret Manager and shared globally across every function in the project.
B. They are stored in the function's source code repository and shared across all versions of every function.
C. They are stored in Cloud Storage and persist independently of any specific function's lifecycle.
D. They are stored in the Cloud Run functions backend, bound to a single function, and exist within that function's lifecycle.

**7.** A team wants to keep their function's environment variable definitions in source control rather than typing them manually on every deploy, and their function is written in Python. What options does the module describe?

A. Store the key-value pairs in a YAML file in source control and provide the file's name at deployment time; in Python, access the variables at runtime using the `os` module.
B. Environment variables cannot be sourced from a file; they must always be typed manually via the gcloud CLI or console on every deploy.
C. Store them in a `.env` file that Cloud Run functions automatically detects without any deployment flag; Python requires no special module to read them.
D. Store them in a Secret Manager secret exclusively; the `os` module cannot read Cloud Run functions environment variables in Python.

**8.** How does Firestore structure the data it stores, according to the module?

A. Data is stored in rows and columns within tables, similar to a relational database.
B. Data is stored as flat key-value pairs with no grouping mechanism of any kind.
C. A set of key-value pairs is stored as a document, and all documents are stored in collections.
D. Data is stored as binary blobs indexed only by a numeric offset, with no key-value structure.

**9.** Which Firestore event types can a Cloud Run function's code be implemented to handle, and what SDK exposes these events?

A. Only "read" events; Firestore triggers cannot react to writes, only to queries performed against the database.
B. Document created, updated, deleted, or any of these events (a general write event) — exposed by the Cloud Run functions for Firebase SDK.
C. Only "created" events; updates and deletes must be handled through a separate polling mechanism.
D. Collection-level "created" and "deleted" events only; individual document events are not supported.

**10.** A team's Firestore database is running in Datastore mode, and they want to attach a Cloud Run function trigger to it. Is this supported?

A. No — Firestore triggers for Cloud Run functions are available only for Firestore in Native mode, not for Firestore in Datastore mode.
B. Yes — Firestore triggers work identically regardless of whether the database is in Native mode or Datastore mode.
C. Yes, but only for "delete" events; Datastore mode supports a restricted subset of trigger types.
D. No — Firestore triggers are not available in either mode; only direct client-library polling is supported.

**11.** When configuring a Firestore-triggered function, what must be specified regarding the document path, and what syntax rule applies to it?

A. No document path is required at all — only the event type needs to be specified; the trigger applies to the entire database.
B. A document path must be specified, and it must always reference a single, exact document — wildcard patterns are not permitted.
C. A document path must be specified, and it must always end with a trailing slash to indicate the end of the path.
D. A document path must be specified — it can reference a specific document or use a wildcard pattern, and it must not contain a trailing slash.

**12.** A Firestore-triggered function needs to (1) see both the pre-update and post-update state of the document that triggered it, and (2) also update a completely different, unrelated document. What does the module say about achieving each?

A. Neither is possible — triggered functions can only see the post-update state and can never modify any document, triggering or otherwise.
B. Both require the Firebase Admin SDK; the snapshot's `ref` property cannot be used for any document modification.
C. On update events, both the before and after snapshot data are available; the triggering document can be modified via `DocumentReference` (the snapshot's `ref` property), while reading or writing other documents requires the Firebase Admin SDK.
D. Only the after-update state is ever available, and modifying any document — including the triggering one — always requires the Firebase Admin SDK.

**13.** What is the relationship between a "secret" and a "secret version" in Secret Manager, and what must be granted to a function's runtime service account before it can read one?

A. A secret is an object holding metadata (replication locations, labels, permissions) plus its secret versions; a secret version stores the actual data as a text string or binary blob; the runtime service account needs the `roles/secretmanager.secretAccessor` role on the secret.
B. A secret and a secret version are the same thing, used interchangeably; no IAM role is required to read a secret if the function is in the same project.
C. A secret version is the parent container holding metadata, while a secret stores only the raw text or binary value; the function needs the Cloud Functions Admin role to read it.
D. A secret stores only replication metadata with no actual data; the real credential value is always stored separately in Cloud Storage, accessible with the Storage Object Viewer role.

**14.** A team rotates a secret used by a running function and wants the change picked up immediately by already-running instances, without waiting for a redeploy. Which access method achieves this, and why?

A. Environment variable — because environment variables are re-resolved on every function invocation, not just at startup.
B. Mounting the secret as a volume — because reading it from the file always retrieves the latest version, whereas an environment variable is resolved once at instance startup and stays pinned to that version.
C. Neither method picks up a rotated secret automatically; a full project deletion and recreation is required.
D. Environment variable — because Secret Manager pushes rotation events directly into a running instance's process environment.

**15.** A function in Project A needs to read a secret stored in Project B. What must be done, beyond the usual access grant, to make this work?

A. Nothing extra — Secret Manager automatically makes every secret in an organization accessible to any function within it.
B. The secret must first be copied into Project A, since Secret Manager does not support any form of cross-project reference.
C. Project A and Project B must be merged into a single project, since secrets cannot be referenced across project boundaries under any circumstances.
D. Grant the function's runtime service account access to the secret as usual, and reference the secret using a resource path that includes Project B's project ID and the secret name.

**16.** A team wants to call a Cloud Run function directly from a Google Standard SQL query in BigQuery. What mechanism enables this, what role must be granted to the connection's service account (and how does it depend on the function's generation), and what permissions does a caller need to actually run the query?

A. This isn't possible; BigQuery can only call other BigQuery-native functions, never external services like Cloud Run functions.
B. A BigQuery remote function enables this, but the required role is identical regardless of generation (`roles/run.invoker` for both 1st and 2nd gen), and no dataset-level permission is needed to run the query.
C. A BigQuery remote function, created via a `CLOUD_RESOURCE` connection, enables this; the connection's service account needs Cloud Functions Invoker (1st gen) or Cloud Run Invoker (2nd gen) on the function, and the caller needs `roles/bigquery.dataViewer` on the dataset plus `roles/bigquery.connectionUser` on the connection.
D. A BigQuery scheduled query enables this, requiring only `roles/bigquery.admin` on the project with no connection or function-level IAM configuration at all.

---

## Answer Key & Explanations

**1. Correct answer: B.**
The module lists Firestore, Cloud SQL, Spanner, Bigtable, BigQuery, and Memorystore as integratable databases, but goes into depth only on Memorystore and Firestore, with BigQuery covered separately through supplementary reading on BigQuery Remote Functions.

**2. Correct answer: D.**
Memorystore is a fully managed service that automates provisioning, replication, failover, and patching, and it's integrated with IAM for secure access and with Cloud Monitoring for service monitoring and alerting.

**3. Correct answer: A.**
Redis is described as an open-source in-memory data structure store used as a database, cache, message broker, and streaming engine; Memcached is described as an open-source distributed memory object caching system.

**4. Correct answer: C.**
The Serverless VPC Access connector must be created in the same region as the function — here, `europe-west1` — and is then attached to the Redis instance's authorized VPC network as a separate step.

**5. Correct answer: B.**
The function deployment must specify the connector's path/name along with environment variables for the Redis host IP address and port; the function is invoked by sending an HTTP GET request to its URL endpoint.

**6. Correct answer: D.**
Environment variables are stored in the Cloud Run functions backend, are bound to a single function, and exist within that function's lifecycle.

**7. Correct answer: A.**
You can store environment variable key-value pairs in a YAML file kept in source control and provide the file's name during function deployment; in Python, you access runtime environment variables using the `os` module.

**8. Correct answer: C.**
Firestore stores a set of key-value pairs as a document, and all documents are stored in collections.

**9. Correct answer: B.**
Function code can be implemented to handle Firestore events that occur when a document is created, updated, deleted, or when any of these events occur; these events are exposed by the Cloud Run functions for Firebase SDK.

**10. Correct answer: A.**
Firestore triggers for Cloud Run functions are available only for Firestore in Native mode — they are not available for Firestore in Datastore mode.

**11. Correct answer: D.**
A function triggered on Firestore events must specify a document path (which can reference a specific document or use a wildcard pattern) and must not contain a trailing slash.

**12. Correct answer: C.**
On update events, the document snapshot data before and after the update is available to the function. The triggering document can be modified through the `DocumentReference` found in the snapshot's `ref` property (from the Firestore Node.js SDK), while reading or writing documents other than the one that triggered the function requires the Firebase Admin SDK.

**13. Correct answer: A.**
A secret is an object containing a collection of metadata (replication locations, labels, permissions, and other information) and its secret versions; a secret version stores the actual secret data as a text string or binary blob. To access a secret, the function's runtime service account must be granted the `roles/secretmanager.secretAccessor` role on the secret.

**14. Correct answer: B.**
Mounting the secret as a volume lets the function reference the latest version of the secret each time the file is read. Passing the secret as an environment variable resolves it once at function instance startup time, so the function is pinned to whichever version was current at that moment.

**15. Correct answer: D.**
You grant the function's runtime service account access to the secret as usual, and you make the secret available by specifying its resource path, which includes the project ID (Project B's) and the secret name.

**16. Correct answer: C.**
A BigQuery remote function, created through a `CLOUD_RESOURCE` connection, lets a Google Standard SQL query invoke a Cloud Run function. The connection's service account must be granted the Cloud Functions Invoker role for a 1st gen function or the Cloud Run Invoker role for a 2nd gen function. To invoke the remote function from a query, the caller needs `roles/bigquery.dataViewer` on the dataset and `roles/bigquery.connectionUser` on the connection.
