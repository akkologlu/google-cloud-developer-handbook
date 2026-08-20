# Module 20 – Service Identity and Authentication

> This is the second module of the same course as `deep-dive/19-fundamentals-of-cloud-run/` (the Cloud Run-focused course). Builds on module 19's brief IAM introduction with a deep look at service accounts, resource hierarchy, least privilege, and secrets/environment variables.

---

# Overview

```text
Service account & identity → Resource hierarchy → Principle of least privilege → Secrets & environment variables
```

---

# Google Cloud APIs and Authorization

Google Cloud is a collection of APIs; you call them from Cloud Run code via client libraries (available for Go, Java, Node.js, Python, Ruby, PHP, C#, C++). Example: publishing to Pub/Sub via `pubsub_v1.PublisherClient()`, which handles the call to `pubsub.googleapis.com`.

**Every API call is authorized by IAM:** IAM identifies the caller from the credentials in the request, then checks policy bindings in the IAM policy attached to the target resource to see if the identity has the required role.

**Policy binding** = one or more members (identities) bound to a single role. A role bundles permissions (e.g. Pub/Sub Publisher → `pubsub.topics.publish`). A member can have multiple bindings (multiple roles).

| Identity type | Description |
| --- | --- |
| Human | Your Google Account (can belong to a group/domain) |
| Machine — service account | Used by machines/apps: a VM, a Cloud Run service, a Cloud Run function, etc. |
| All users | Special identifier for public/anonymous access |

---

# Service Accounts

A **service account** is a special identity for machines, identified by a unique email address. Unlike user accounts: no password, can't sign in via browser/cookies; other users/service accounts can act on its behalf; not a member of your Workspace domain (though it can join groups).

Any code you run (VM, Cloud Run service, Cloud Build build) gets access to a **built-in service account**, automatically used by client libraries for authentication — but you can (and should) replace it with your own **user-managed** one.

**Service accounts in Cloud Run:** every service/job is linked to a service account — its "service identity." By default this is the **default Compute Engine service account with the Editor role**. Best practice: one user-managed service account per service, with the minimal permissions required.

**Runtime flow (calling a Google API, e.g. Pub/Sub):** app code → client library → authenticates with the service's service account → acquires an **OAuth 2.0 access token** → calls the API → IAM verifies the token and checks the policy binding on the target resource.

**Service-to-service communication:**

| Pattern | Mechanism |
| --- | --- |
| Asynchronous | Cloud Tasks, Pub/Sub, Cloud Scheduler, Eventarc |
| Synchronous | Direct HTTP call to the other service's endpoint URL — best practice: dedicated service identity per calling service, granted `roles/run.invoker` on the receiving service: `gcloud run services add-iam-policy-binding RECEIVING_SERVICE --member='serviceAccount:CALLING_SERVICE_IDENTITY' --role='roles/run.invoker'` |

For synchronous calls, the request must present a **Google-signed OpenID Connect (OIDC) ID token** as proof of identity (obtained via Google's authentication client libraries in the calling service; parsed/verified with the same libraries in the receiving service) — distinct from the OAuth 2.0 access token used for Google API calls.

---

# Resource Hierarchy

```text
Organization (root) → Folder (optional) → Project (base-level entity) → resources
```

- **Organization:** root node; central visibility/control over everything beneath it.
- **Folder:** optional grouping (departments, teams, business units).
- **Project:** base-level entity — required to create resources, use APIs/services, manage permissions, enable billing.
- Every resource has exactly one parent (except the organization node).

**Every resource in the hierarchy has its own IAM policy.** A policy binding grants an identity a role on that specific resource — e.g. "Pub/Sub Publisher" on a topic.

**Policy binding inheritance:** a binding added at a higher-level resource (e.g. the project) is inherited by all lower-level resources (e.g. every Pub/Sub topic in it) — useful for granting access across many resources at once.

**Effective IAM policy** on a resource = its own bindings + all bindings inherited from its parent and ancestors. **Permissions granted at a higher level can't be revoked at a lower level.**

---

# Principle of Least Privilege

| Role type | Scope | Examples |
| --- | --- | --- |
| **Basic** | Very powerful, span all services — avoid granting by default in production | Owner, Editor, Viewer |
| **Predefined** | Granular access to a specific service, Google-managed | Cloud Run Admin, Pub/Sub Publisher, Cloud Tasks Enqueuer |
| **Custom** | Granular access from a user-specified permission list | — |

**The default service account risk:** if you don't specify one, Cloud Run uses the default Compute Engine service account with the broad **Editor** role — which, due to policy binding inheritance, grants read/write on most resources in the project. Convenient, but an inherent security risk (resources can be created/modified/deleted through it).

**Mitigation (3 steps):**
1. Create a new service account for the Cloud Run service.
2. Configure it as the service's identity (settable at create/update time, or on a new revision — console, gcloud CLI, YAML, or Terraform).
3. Add policy bindings for that identity with predefined/custom roles on only the resources the service needs.

A newly created service account starts with **zero permissions** — any Google Cloud API call from it is rejected by IAM until you explicitly grant access:

```shell
gcloud pubsub topics add-iam-policy-binding my-topic \
  --member="my-service-account-email" --role="roles/pubsub.publisher"
```

---

# Environment Variables and Secrets

**Environment variables:** key-value pairs injected into the container, accessible to app code at runtime. Set at service/job creation, update, or new-revision deploy time (console, gcloud CLI, YAML, or Terraform); certain reserved names can't be set (see the container runtime contract). A Dockerfile `ENV` default is overridden by a Cloud Run variable of the same name.

```shell
gcloud run deploy my-service --image my-container-image-url --update-env-vars FOO=bar,BAZ=boo
gcloud run jobs create my-job --image my-container-image-url --update-env-vars FOO=bar,BAZ=boo
```

| Language | Access |
| --- | --- |
| Python | `os.environ.get("key")` |
| Node.js | `process.env.key` |
| Java | `System.getenv("key")` |

**Secrets (Secret Manager):** for sensitive config (API keys, passwords). A **secret** object holds metadata (replication locations, labels, permissions) plus **secret versions**, each storing the actual data (text/binary).

| Access method | Behavior | Best for |
| --- | --- | --- |
| Mount as a volume | Available to the container as a file; always fetches the latest value from Secret Manager on read | Using `latest` |
| Pass as an environment variable | Resolved once at instance startup | Pinning to a specific version |

```shell
# as a mounted volume
gcloud run deploy my-service --image my-container-image --update-secrets=SECRET_FILE_PATH=my_secret:VERSION
# as an environment variable
gcloud run deploy my-service --image my-container-image --update-secrets=ENV_VAR_NAME=my_secret:VERSION
```

Any config change (including updating a secret) creates a new revision, which subsequent revisions inherit. To grant access, give the service account the **Secret Manager Secret Accessor** role:

```shell
gcloud secrets add-iam-policy-binding my-secret-id \
  --member="my-service-account-email" --role="roles/secretmanager.secretAccessor"
```

---

# Module Summary

A Cloud Run service is more than running code — it's also an identity (service account) used to call Google Cloud APIs and other services. This module covers how that identity authenticates (access tokens for Google APIs, OIDC ID tokens for service-to-service calls), how permissions propagate through the resource hierarchy (Organization → Folder → Project → resources, with binding inheritance that can't be revoked downward), why the default service account is a security risk and how to apply least privilege instead, and how to manage non-sensitive (environment variables) versus sensitive (Secret Manager) configuration.

---

# Key Points

- A policy binding is member + role; a member can hold multiple roles via multiple bindings.
- Service accounts have no password and can't sign in via a browser; they're identified by a unique email.
- Every Cloud Run service/job has a service identity (service account) — replace the default with a user-managed, minimally-permissioned one.
- OAuth 2.0 access tokens authorize calls to Google Cloud APIs; OIDC ID tokens prove identity for direct service-to-service HTTP calls.
- Resources form a strict hierarchy (Organization → Folder → Project → resource) with exactly one parent each.
- Policy bindings are inherited down the hierarchy, and permissions granted higher up can't be revoked lower down.
- Basic roles (Owner/Editor/Viewer) are dangerously broad — prefer predefined or custom roles in production.
- The default Compute Engine service account (Editor role) is Cloud Run's default identity and, due to inheritance, has broad access across the project — a real risk, not just a convenience trade-off.
- A newly created service account has zero permissions by default; grant only what's needed via policy bindings.
- A Dockerfile's `ENV` default is overridden by a same-named Cloud Run environment variable.
- Secrets are versioned Secret Manager objects, not plain environment variables — mount as a volume for always-fresh `latest` values, or pass as an env var pinned to a specific version.
- Accessing a secret requires granting the service account the Secret Manager Secret Accessor role — the same policy-binding pattern used throughout IAM.
