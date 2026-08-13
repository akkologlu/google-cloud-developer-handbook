# Module 15 – Integrating with Cloud Databases

> This is Module 4 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 3 (Module 14: Securing Cloud Run Functions).

---

# Overview

Cloud Run functions can integrate with Firestore, Cloud SQL, Spanner, Bigtable, BigQuery, and Memorystore. This module goes deep on two connection models, plus supporting mechanisms:

```text
Memorystore (function actively connects out) ↔ Firestore (database changes trigger the function) + environment variables + Secret Manager + BigQuery Remote Functions
```

---

# Memorystore

**Memorystore** is a fully managed, highly available, scalable, and secure in-memory cache service for **Redis** and **Memcached**. It automates provisioning, replication, failover, and patching, and integrates with IAM (secure access) and Cloud Monitoring (observability).

| Technology | What it is |
| --- | --- |
| Redis | Open-source in-memory data structure store — used as a database, cache, message broker, and streaming engine |
| Memcached | Open-source distributed memory object caching system |

**Connecting a function to a Redis instance** (via Serverless VPC Access):

1. Determine the Redis instance's authorized VPC network.
2. Create a Serverless VPC Access connector **in the same region as the function**.
3. Attach the connector to the Redis instance's authorized VPC network (specifying network, region, IP address range at creation).
4. Confirm the connector is in a `Ready` state.
5. Deploy the function specifying the connector's path/name and environment variables for the Redis host IP and port.
6. Function code uses those environment variables to instantiate a Redis client; invoke via an HTTP `GET` request.

> **Exam trap:** The connector's region must match the **function's** region, not the Redis instance's region — it is then attached to the Redis instance's authorized network separately.

---

# Environment Variables

Key-value pairs set for a Cloud Run function **at deployment time**.

| Property | Detail |
| --- | --- |
| Access | Read by function code at runtime, or used as buildpack configuration |
| Storage/scope | Stored in the Cloud Run functions backend; bound to a single function; exist within that function's lifecycle |
| How to set | gcloud CLI, Google Cloud console, or a YAML file in source control (name provided at deploy time) |
| Reading them (Python) | The `os` module |

---

# Firestore

**Firestore** is a fully managed, serverless NoSQL document database with high availability, no-maintenance-window scalability, and multi-region replication. Data is stored as **documents** (sets of key-value pairs), grouped into **collections**.

**Extending Firestore with Cloud Run functions:** write function code to handle events triggered by changes in Firestore — exposed via the **Cloud Run functions for Firebase SDK**.

| Event type | Fires on |
| --- | --- |
| Created | Document creation |
| Updated | Document update |
| Deleted | Document deletion |
| (any) | Any of the above — a general "write" event |

> **Exam trap:** Firestore triggers for Cloud Run functions work **only for Firestore in Native mode** — they are **not available for Firestore in Datastore mode**.

**Required trigger configuration:** a **document path** (a specific document or a wildcard pattern, never with a trailing slash) and an **event type**.

| Concept | Detail |
| --- | --- |
| Snapshot | A snapshot of the event's data reaches the function; on update events, both the before and after snapshots are available |
| `DocumentReference` | `snapshot.ref` (from the Firestore Node.js SDK) — lets you modify the document that triggered the function |
| Firebase Admin SDK | Needed to read/write documents **other than** the one that triggered the function |

---

# Secrets and Secret Manager

Sensitive credentials (DB username/password, API keys) are stored in **Secret Manager** rather than embedded in code or config.

| Concept | Detail |
| --- | --- |
| Secret | An object holding metadata (replication locations, labels, permissions, etc.) and its secret versions |
| Secret version | Stores the actual secret data (a text string or binary blob) |
| Prerequisite | Enable the Secret Manager API to create/manage secrets |

**Granting access:** grant the function's **runtime service account** the `roles/secretmanager.secretAccessor` role on the secret.

**Making a secret available to a function:**

| Method | Version behavior |
| --- | --- |
| Mount as a volume | Function reads it from a file; each read gets the **latest** version |
| Pass as an environment variable | Resolved at **instance startup time** — the function is pinned to a **specific** version until a new instance starts |

**Cross-project access:** grant the runtime service account access as above, then reference the secret via a resource path that includes the **project ID** and secret name.

> **Exam trap:** Need the function to pick up a rotated secret immediately? Use a volume mount, not an environment variable — env vars are resolved once, at instance startup.

**Use case flow (external API key):** store the API key as a secret → grant the runtime service account the Secret Manager Secret Accessor role → at deploy time, specify the secret name and access method (mounted file path or environment variable) → function code reads the file/env var and calls the external API with the value.

---

# BigQuery Remote Functions

A **BigQuery remote function** lets Google Standard SQL queries call out to a **Cloud Run function** — integrating SQL with arbitrary code.

**Setup:**

1. Enable the BigQuery Connection API.
2. Have the necessary IAM role (e.g., `roles/bigquery.admin`).
3. Create a `CLOUD_RESOURCE` connection (console, `bq` CLI, or the connection API):
   ```bash
   bq mk --connection --display_name='friendly name' \
     --connection_type=CLOUD_RESOURCE \
     --project_id=my_project_id --location=US my-connection
   ```
4. Grant the connection's service account the correct Invoker role **on the function**:

| Generation | Role to grant |
| --- | --- |
| 1st gen | Cloud Functions Invoker |
| 2nd gen | Cloud Run Invoker |

5. Have the required roles on the dataset and connection (e.g., `roles/bigquery.admin`), then create the remote function:
   ```sql
   CREATE FUNCTION my_project_id.my_dataset.function_name(x INT64, y INT64) RETURNS INT64
   REMOTE WITH CONNECTION `my_project_id.us.my-connection`
   OPTIONS (endpoint = 'https://us-east1-my_gcf_project.cloudfunctions.net/function_name')
   ```
6. To invoke it: have `roles/bigquery.dataViewer` on the dataset and `roles/bigquery.connectionUser` on the connection, then call it in a query:
   ```sql
   SELECT val, my_project_id.my_dataset.function_name(val, 2)
   FROM UNNEST([NULL, 2, 3, 5, 8]) AS val;
   ```

> **Exam trap:** The Invoker role granted to the connection's service account depends on the function's generation — Cloud Functions Invoker for 1st gen, Cloud Run Invoker for 2nd gen — mirroring the same two roles used for function-to-function calls.

---

# Module Summary

Cloud Run functions integrate with data two ways: actively connecting out to Memorystore's Redis/Memcached cache over a Serverless VPC Access connector (region-matched to the function, attached to the instance's authorized network, with connection details passed as environment variables), and reacting to Firestore document changes via triggers (create/update/delete/write, Native mode only, scoped by document path, delivering before/after snapshots and a `DocumentReference` for the triggering document, with the Firebase Admin SDK needed for any other document). Environment variables carry deployment-time configuration bound to a single function's lifecycle, while Secret Manager protects sensitive credentials — granted to a function via its runtime service account's `secretAccessor` role, delivered as a volume mount (always latest version) or environment variable (pinned at instance startup). Separately, BigQuery Remote Functions let SQL queries invoke a Cloud Run function through a `CLOUD_RESOURCE` connection, with an Invoker role (generation-dependent) granted to the connection's service account.

---

# Key Points

- Memorystore is a fully managed Redis/Memcached cache; connecting a function to it requires a Serverless VPC Access connector in the function's region, attached to the Redis instance's authorized VPC network.
- Environment variables are deployment-time key-value pairs bound to a single function and its lifecycle, set via gcloud, console, or a YAML file; Python reads them via the `os` module.
- Firestore triggers fire on document create/update/delete/write events, require a document path (no trailing slash) and event type, and work only for Firestore in Native mode — not Datastore mode.
- A triggered function receives a data snapshot (before and after, on updates); `snapshot.ref` gives a `DocumentReference` for the triggering document, while the Firebase Admin SDK is needed to touch other documents.
- Secret Manager stores sensitive credentials as secrets (metadata + versions); a secret version holds the actual text/binary data.
- Function access to a secret requires granting `roles/secretmanager.secretAccessor` to its runtime service account; volume-mounted secrets always read the latest version, while environment-variable secrets are pinned to whatever version was current at instance startup.
- Cross-project secret access needs the same IAM grant plus a resource path that includes the target project ID.
- BigQuery Remote Functions let a SQL query call a Cloud Run function via a `CLOUD_RESOURCE` connection; the connection's service account needs Cloud Functions Invoker (1st gen) or Cloud Run Invoker (2nd gen) on the function, and callers need `bigquery.dataViewer` on the dataset plus `bigquery.connectionUser` on the connection.
