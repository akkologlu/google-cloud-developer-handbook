# Module 14 – Securing Cloud Run Functions

> This is Module 3 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 2 (Module 13: Calling and Connecting Cloud Run Functions).

---

# Overview

This module covers three layers for securing Cloud Run functions:

```text
Identity-based access control (who can call it) → Network-based access control (where can traffic come from/go to) → Encryption with CMEK (who can read the data at rest)
```

---

# Identity-Based Access Control

Functions are **private by default and require authentication**; you can opt into deploying a function as public.

| Step | Detail |
| --- | --- |
| 1. Authentication | Validate the identity credential — confirm the requestor is who it claims to be |
| 2. Authorization | Evaluate the authenticated identity's permissions |

**Two identity kinds:** service accounts (non-person identities — a function, application, or VM) and user accounts (individual Google Account holders or Google Groups).

**Token-based authentication:** clients create a token from a service/user account credential; the token has a limited lifetime, travels with the request, and limits the damage if the underlying credential leaks.

| Token type | Used for |
| --- | --- |
| OAuth 2.0 access token | Authenticating API calls |
| ID token | Authenticating calls to developer-created code (e.g., a function calling another function) |

Both are created using the OAuth 2.0 framework and OpenID Connect (OIDC).

**IAM roles (Cloud Run functions predefined roles):**

| Role | Grants |
| --- | --- |
| Cloud Functions Admin | Full administrative management |
| Cloud Functions Developer | Develop and deploy functions |
| Cloud Functions Invoker | Invoke the function only |
| Cloud Functions Viewer | Read-only visibility |

You authorize principals (user or service account emails) with these roles via the console or gcloud CLI.

> **Exam trap:** A function is private and requires authentication by default — public access is something you opt into, not the starting state.

---

# Who Can Call a Function

| Function type | Who can invoke it |
| --- | --- |
| Event-driven function | Only the event source it is subscribed to |
| HTTP function | Different identity kinds (a developer testing it, a calling service), each providing an ID token with the right permissions in the `Authorization` header |

**Developer testing flow:** have a user account with a role granting appropriate permissions → generate an ID token from that account → pass the token in the `Authorization` header of the request.

---

# Runtime Service Account

Every function is associated with a **runtime service account** — the identity it uses when it accesses other Google Cloud resources.

| Setting | Detail |
| --- | --- |
| Default | Compute Engine default service account (or App Engine default service account for 1st gen) — testing/development only |
| Production | Specify a dedicated runtime service account, granted only the minimum permissions required |

---

# Function-to-Function Calls

To let one function call another and restrict which functions may call which:

| Generation | Role to grant on the receiving function |
| --- | --- |
| Cloud Run functions | `roles/run.invoker`, granted to the calling function's service account |
| Cloud Run functions (1st gen) | `roles/cloudfunctions.invoker`, granted to the calling function's service account |

The calling function must also provide a **Google-signed ID token**, with the `audience` (`aud`) field set to the receiving function's URL, sent in the `Authorization` header.

> **Exam trap:** The Invoker role is granted **on the receiving function**, to the **calling function's identity** — not the other way around. And the ID token's `aud` field ties it to one specific target; a token issued for function B is rejected if presented to function C.

---

# Network-Based Access Control

**Ingress settings** restrict whether a function can be invoked by resources outside your Google Cloud project or VPC Service Controls perimeter:

| Ingress option | Meaning |
| --- | --- |
| Allow all traffic | No restriction |
| Allow internal traffic only | Only Workflows and VPC networks in the same project/perimeter |
| Allow internal traffic and traffic from Cloud Load Balancing | Internal traffic plus Cloud Load Balancing |

**Egress settings** control routing of outbound HTTP requests — they require connecting the function to a VPC network via a **Serverless VPC Access connector**:

| Egress option | Meaning |
| --- | --- |
| Route all outbound traffic through the connector | Everything goes through the connector |
| Route only requests to private IPs through the connector | Only private-IP-bound traffic goes through the connector |

**VPC Service Controls** add a further layer: create a service perimeter, add project(s) (host + service projects for Shared VPC), and restrict the Cloud Functions API via organization policies. With those policies in place:

- HTTP functions only accept traffic from a VPC network within the perimeter.
- All functions must use a Serverless VPC Access connector.
- All functions must route all egress traffic through the VPC network.

> **Exam trap:** Ingress/egress settings are per-function opt-in choices; VPC Service Controls organization policies make the strictest versions of both **mandatory** across the perimeter — not just available as an option.

---

# Protecting Data with Cloud KMS and CMEK

**Cloud KMS** lets you create and manage encryption keys — **customer-managed encryption keys (CMEK)** — that protect Cloud Run functions and related data at rest. Keys are owned by you, not Google, and can be software keys, HSM-backed, or external.

**What CMEK protects:**

| Data type | Detail |
| --- | --- |
| Function source code | Uploaded for deployment, stored in Cloud Storage, used in the build |
| Build results | The built container image, and each deployed function instance |
| Internal event transport channels | Their at-rest data |

If the key is disabled or destroyed, **no one** — including you — can access the data it protects.

**Setup steps:**

| Step | Detail |
| --- | --- |
| 1. Create a key | A single-region encryption key |
| 2. Create a CMEK-enabled Artifact Registry repository | Must use the **same key** as the function |
| 3. Grant service account access | Cloud Run Functions, Artifact Registry, and Cloud Storage service accounts each need the **Cloud KMS CryptoKey Encrypter/Decrypter** role, added as principals of the key |
| 4. Enable CMEK on the function | Specify the key and repository at deploy time |

**Use case — Cloud Storage:** objects can use CMEK individually or as a bucket default; a function triggered via Eventarc on a Cloud Storage change can retrieve the decrypted object, or a function can encrypt objects before uploading them.

**Constraints and failure behavior:**

- Cloud Run functions uses the key's **primary version only** — you cannot pin a specific key version.
- If the key is destroyed/disabled, or required permissions are revoked: active instances are **not** shut down, in-progress executions **continue**, but new executions and executions requiring a new instance **fail**.

> **Exam trap:** Destroying/disabling a CMEK key does not instantly stop running functions — it blocks only *new* access (new executions, new instances), while already-running work finishes.

---

# Module Summary

Cloud Run functions are secured on two fronts: identity-based access control (authentication via OAuth 2.0 access tokens or ID tokens, authorization via Cloud Functions Admin/Developer/Invoker/Viewer IAM roles, per-function runtime service accounts scoped to least privilege, and `roles/run.invoker` plus an audience-scoped ID token for function-to-function calls) and network-based access control (ingress settings restricting who can call in, egress settings routing outbound traffic through a Serverless VPC Access connector, and VPC Service Controls making the strictest versions of both mandatory across a perimeter). Separately, Cloud KMS customer-managed encryption keys (CMEK) protect function source code, build results, and internal event transport data at rest — owned exclusively by the customer, using only the key's primary version, with destruction/disabling blocking new access without killing already-running work.

---

# Key Points

- Functions are private and require authentication by default; public access is opt-in.
- Authentication (verifying identity) always precedes authorization (evaluating permissions).
- OAuth 2.0 access tokens authenticate API calls; ID tokens authenticate calls to developer-written code, including function-to-function calls.
- Cloud Functions Admin/Developer/Invoker/Viewer are the predefined IAM roles; Invoker grants call-only access.
- Event-driven functions can only be invoked by their subscribed event source; HTTP functions require an ID token in the `Authorization` header from any calling identity.
- The default runtime service account (Compute Engine/App Engine default) is for testing only — production needs a dedicated service account with minimum required permissions.
- Function-to-function calls require `roles/run.invoker` (or `roles/cloudfunctions.invoker` for 1st gen) on the receiving function, granted to the caller's service account, plus a Google-signed ID token with `aud` set to the receiving function's URL.
- Ingress settings (all / internal only / internal + Cloud Load Balancing) and egress settings (all traffic / private IPs only, via a Serverless VPC Access connector) control network-level access; VPC Service Controls can make the strictest versions of both mandatory across a perimeter.
- CMEK protects function source code, build results, and internal event transport data at rest with a customer-owned key; the function and its Artifact Registry repository must share the same key, and Cloud Run Functions/Artifact Registry/Cloud Storage service accounts each need the Cloud KMS CryptoKey Encrypter/Decrypter role.
- Only the key's primary version is used for CMEK; destroying or disabling the key blocks new executions and new instances without stopping work already in progress.
