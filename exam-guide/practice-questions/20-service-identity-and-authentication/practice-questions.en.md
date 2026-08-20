# Module 20 — Service Identity and Authentication: Practice Questions

This set covers service accounts and identity (how IAM authorizes API calls, the structure of a policy binding, the three identity types, what makes a service account different from a user account, built-in versus user-managed service accounts, Cloud Run's default service identity, OAuth 2.0 access tokens for Google API calls, asynchronous versus synchronous service-to-service communication, the Cloud Run Invoker role, and OpenID Connect ID tokens for direct service calls), resource hierarchy (Organization/Folder/Project, per-resource IAM policies, policy binding inheritance, and effective IAM policy), the principle of least privilege (the three IAM role types, why Cloud Run's default service account is a security risk, and the three-step mitigation), and secrets and environment variables (how environment variables work, and how Secret Manager secrets differ and are accessed).

The questions are weighted toward the distinctions that actually trip people up: why a newly created service account starts with zero permissions rather than some default access, why permissions granted at a higher level in the resource hierarchy can never be revoked at a lower level, why the default Cloud Run service account is a real security risk rather than just a convenience trade-off, why OAuth 2.0 access tokens and OpenID Connect ID tokens serve different purposes, and why a secret mounted as a volume behaves differently from one passed as an environment variable.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** When application code calls a Google Cloud API such as Pub/Sub, how does IAM decide whether to allow or reject the call?

A. IAM allows every call from any Google Cloud resource unconditionally, since authorization only applies to human users signing in through a browser.
B. IAM inspects the request and identifies the caller from the credentials in the API request, then checks whether there's a policy binding with the required role in the IAM policy attached to the target resource (e.g. the Pub/Sub topic) — rejecting the call if the identity isn't authorized.
C. IAM only checks the caller's IP address and geographic location, with no involvement of policy bindings or roles of any kind.
D. IAM delegates all authorization decisions entirely to the calling application's own code, with no independent verification performed.

**2.** What does a single IAM policy binding actually consist of, and can a member have more than one role?

A. A policy binding binds a member and a resource directly together with no role involved, and a member can never be associated with more than one resource.
B. A policy binding is a temporary, single-use authorization token that expires after one API call, unrelated to any member or role.
C. A policy binding permanently deletes any previous permissions a member had, and a member can only ever hold exactly one role across an entire IAM policy.
D. A policy binding binds one or more members (identities) to a single role, where the role contains a set of permissions that lets the member perform specific actions on a resource; a member can be attached to multiple policy bindings in an IAM policy, giving it more than one role.

**3.** What are the three types of identities IAM supports as policy binding members?

A. Human identities (a Google Account, which can be part of a group or domain), service accounts (used by machines, applications, or services such as a VM, Cloud Run service, or Cloud Run function), and "all users" (a special identifier for public/anonymous access).
B. Only human identities are supported by IAM; service accounts and public access identifiers don't exist as distinct concepts in the model.
C. Root identities, admin identities, and guest identities — a completely different three-way classification than the one IAM actually uses.
D. Billing identities, network identities, and storage identities, categorized by which Google Cloud product they're used with rather than by who or what they represent.

**4.** How does a service account differ from a regular user account?

A. A service account has a password just like a user account, and can sign in through a browser using cookies in exactly the same way.
B. A service account is automatically a member of your Google Workspace domain in the same way a user account is, with no distinction between the two.
C. A service account is a special type of account used by machines, applications, or services, identified by a unique email address; unlike user accounts, it has no password and can't sign in via a browser or cookies, other users or service accounts can be allowed to act on its behalf, and it isn't a member of your Google Workspace domain (though it can still be added to groups).
D. A service account cannot be part of any group under any circumstances, unlike a user account which always can be.

**5.** If you run code on a virtual machine, in a Cloud Run service, or as part of a Cloud Build build, what service account do client libraries use by default, and what's recommended instead?

A. Client libraries never use any service account automatically under any circumstances, requiring every single API call to be manually authenticated by the developer.
B. The built-in service account can never be replaced under any circumstances, making a user-managed service account architecturally impossible to configure.
C. Client libraries require a brand-new service account to be manually created and configured before any authentication of any kind can occur, with no default ever provided.
D. You always have access to a built-in service account that client libraries automatically use for authentication when connecting to Google Cloud APIs, but it's recommended that you replace it with your own user-managed service account.

**6.** By default, what service account does a Cloud Run service or job run as if you don't specify one, and what is this account also known as?

A. Every Cloud Run service or job is linked to a service account known as the "service identity" — by default, this is the default Compute Engine service account with the Editor role.
B. By default, no service account whatsoever is linked to a Cloud Run service or job, making any Google Cloud API call from it impossible regardless of configuration.
C. By default, Cloud Run services and jobs run as a special "Cloud Run only" service account that has zero permissions and cannot be changed.
D. By default, Cloud Run services and jobs run using the credentials of whichever human user most recently deployed them, rather than any service account at all.

**7.** When application code in a Cloud Run service uses a client library to call a Google Cloud API, what kind of token does the client library acquire, and what does it use to authenticate?

A. The client library requires the end user's personal password to be hardcoded directly into the application source code before any API call can succeed.
B. The client library bypasses IAM entirely for any call originating from inside a Cloud Run container, regardless of which API is being called.
C. The client library authenticates using the service's runtime service account and, for most Google APIs, automatically acquires an OAuth 2.0 access token to make the call — which IAM then verifies against the policy binding on the target resource.
D. The client library uses a permanent, unchanging API key that is generated once and never expires or requires any further verification by IAM.

**8.** If your application architecture uses multiple Cloud Run services that need to communicate with each other, what mechanisms does the module describe for asynchronous versus synchronous communication?

A. Both asynchronous and synchronous communication are described as requiring exactly the same mechanism, with no distinction made between them anywhere in the module.
B. For asynchronous communication, various Google Cloud services such as Cloud Tasks, Pub/Sub, Cloud Scheduler, or Eventarc can be used; for synchronous communication, a service calls another service's endpoint URL directly over HTTP, with IAM and an individual service identity for the calling service recommended as a best practice.
C. Asynchronous communication is described as impossible between Cloud Run services under any circumstances, leaving synchronous HTTP calls as the only available option.
D. Synchronous communication is described as requiring Cloud Tasks exclusively, while asynchronous communication is described as requiring a direct HTTP call to the receiving service's endpoint.

**9.** To let a calling Cloud Run service invoke a private receiving Cloud Run service synchronously, what specific setup does the module describe?

A. You configure the receiving service to accept requests from the calling service by making the calling service's service account a principal on the receiving service, and grant that service account the Cloud Run Invoker (`roles/run.invoker`) role — e.g. via `gcloud run services add-iam-policy-binding RECEIVING_SERVICE --member='serviceAccount:CALLING_SERVICE_IDENTITY' --role='roles/run.invoker'`.
B. You disable IAM entirely on both services, since private-to-private Cloud Run communication is described as not supporting any authentication mechanism.
C. You grant the calling service's service account the Owner role on the entire Google Cloud project, since no more targeted role exists for this purpose.
D. You configure both services to share a single, identical service account with no distinct roles or policy bindings involved at any point.

**10.** For synchronous service-to-service calls, what must the request from the calling service present as proof of identity, and how does this relate to OpenID Connect?

A. The request must present the calling service's raw, unencrypted service account password directly in the HTTP request body, with OpenID Connect playing no role in the process.
B. The request must present a CAPTCHA response solved by a human operator before any service-to-service call can be authenticated, since OpenID Connect only applies to human sign-in flows.
C. The request must present a Google-signed OpenID Connect ID token as proof of identity; OpenID Connect is an identity protocol based on OAuth 2.0 that enables identity verification of a client based on authentication performed by an authorization server, and can also be used to obtain basic profile information about the client.
D. The request must present a permanently valid, non-expiring API key that has no relationship whatsoever to OpenID Connect or any other identity protocol.

**11.** How are Google Cloud resources organized, and how many parents does a given resource have in that structure?

A. Resources exist in a completely flat structure with no hierarchy of any kind, and the concept of a "parent" resource doesn't apply anywhere in Google Cloud.
B. Google Cloud resources are organized hierarchically: the organization is the root node, folders are an optional grouping mechanism beneath it (e.g. mapped to departments or teams), and the project is the base-level entity required to create resources, use APIs, manage permissions, and enable billing; every resource has exactly one parent, except the top organization node, which has none.
C. Every resource has exactly two parents at all times, one for billing purposes and one for access-control purposes, with no exceptions anywhere in the hierarchy.
D. Projects are described as the root node of the hierarchy, with organizations existing as an optional grouping mechanism beneath individual projects — the reverse of the module's actual structure.

**12.** Does every resource in the Google Cloud resource hierarchy have its own IAM policy, and how do you grant permissions on it?

A. Only the organization node has an IAM policy; individual resources beneath it, such as a specific Pub/Sub topic, have no IAM policy of their own.
B. IAM policies exist only for billing-related resources, and non-billing resources such as a Pub/Sub topic can never have permissions granted on them directly.
C. Only projects have IAM policies; folders and individual resources beneath a project are described as entirely unable to have any policy attached.
D. Every resource in the hierarchy has an IAM policy, and you grant permissions on it using policy bindings — for example, granting the "Pub/Sub Publisher" role on a specific topic's IAM policy.

**13.** If you add a policy binding to a Google Cloud project instead of to an individual Pub/Sub topic within it, what happens to that binding with respect to the topics in the project?

A. The binding applies only to the project resource itself and has absolutely no effect on any child resource, including any Pub/Sub topics within it.
B. The binding is automatically deleted within 24 hours unless manually re-applied to each individual child resource one at a time.
C. Lower-level resources such as the Pub/Sub topics inherit the policy binding from their parent resource (the project) — useful for granting permission (e.g. to publish messages) across all topics in a project rather than just a single one.
D. The binding is converted into a completely different role automatically, unrelated to the role that was originally specified when the binding was created.

**14.** What does the "effective IAM policy" on a resource consist of, and can permissions granted to a parent resource be removed at a child resource?

A. The effective IAM policy consists solely of bindings added directly to that specific resource, entirely excluding any bindings from parent resources.
B. The effective IAM policy on a resource includes bindings on the resource itself plus bindings inherited from its parent (and that parent's ancestors); permissions granted to a parent resource are bindings you cannot take away at a lower level.
C. The effective IAM policy is recalculated from scratch every 24 hours and has no persistent relationship to any parent resource's bindings.
D. Permissions granted at a parent resource can always be selectively revoked for individual child resources without any restriction, making inheritance purely advisory rather than binding.

**15.** What are the three types of IAM roles, and which type should generally be avoided by default in production environments?

A. The three role types are Basic, Predefined, and Temporary, with temporary roles automatically expiring after 24 hours regardless of configuration.
B. Predefined roles are described as the most powerful and dangerous type, spanning all services, while basic roles are described as the safest and most granular option available.
C. Custom roles are described as being managed entirely by Google Cloud with no user input of any kind, making "custom" a misleading name for this role type.
D. Basic roles (Owner, Editor, Viewer) are very powerful roles with permissions spanning all services and should not be granted by default in production; predefined roles (e.g. Cloud Run Admin, Pub/Sub Publisher) provide granular access to a specific service and are managed by Google Cloud; custom roles let you build your own role from a user-specified list of permissions.

**16.** Why does the module describe Cloud Run's default service account as an inherent security risk rather than just a convenience trade-off?

A. The default service account used is the Compute Engine service account, which has the broad Editor role on the project; because of policy binding inheritance, this default account has read and write permissions on most resources in the project, meaning resources can be created, modified, or deleted through it — a real security risk, not merely an inconvenience.
B. The default service account is described as only ever having access to a single, specific resource that you explicitly select at deploy time, with no broader access possible.
C. The risk is described as purely theoretical and never actually materializes in practice, since Cloud Run is said to prevent any resource modification regardless of the service account's role.
D. The default service account is described as having zero permissions of any kind, making it perfectly safe but functionally useless for any real workload.

**17.** What three steps does the module recommend to mitigate the default service account's security risk in a Cloud Run service?

A. Disable IAM entirely for the project, since IAM itself is described as the underlying cause of the risk rather than a tool for mitigating it.
B. Create a new service account for the Cloud Run service, configure that service account as the service's identity, and add policy bindings for that identity with predefined or custom roles only on the resources the service actually needs to access.
C. Grant the Owner basic role to the existing default service account, since Owner is described as more restrictive than Editor and therefore safer.
D. Delete the Cloud Run service entirely and replace it with a Compute Engine virtual machine, since Compute Engine is described as immune to this type of risk.

**18.** When you first create a new, user-managed service account for a Cloud Run service, what permissions does it have, and what happens if the service tries to call a Google Cloud API before you grant it any?

A. The new service account automatically inherits every permission the default Compute Engine service account had, making it no more restrictive than the default it was meant to replace.
B. The new service account is automatically granted the Owner role on the entire organization the moment it's created, requiring no further configuration of any kind.
C. The new service account can call any Google Cloud API immediately upon creation, since service account creation itself is treated as an implicit grant of universal access.
D. By default, the newly created service account has no permissions at all and doesn't appear in any policy binding; if code running as part of the Cloud Run service calls any Google Cloud API before a policy binding grants it a role, the call is rejected by IAM.

**19.** How do environment variables work in Cloud Run, and what happens if a variable with the same name is set both as an `ENV` default in the Dockerfile and directly on the Cloud Run service?

A. Environment variables set on the Cloud Run service or job are set as key-value pairs, injected into the application container, and made accessible to application code at runtime (e.g. via `os.environ.get("key")` in Python, `process.env.key` in Node.js, or `System.getenv("key")` in Java); a same-named variable set on the Cloud Run service or job overrides the default value set via `ENV` in the Dockerfile.
B. Environment variables set on a Cloud Run service are visible only to the Google Cloud console and are never actually injected into or accessible from the running application container.
C. A Dockerfile's `ENV` default always takes priority over any same-named variable set on the Cloud Run service, making the Cloud Run-level setting purely cosmetic and non-functional.
D. Setting an environment variable on a Cloud Run service requires rebuilding and repackaging the container image itself, since environment variables are described as being permanently baked into the image at build time only.

**20.** What does a Secret Manager secret consist of, and what are the two ways to make a secret accessible to a Cloud Run service?

A. A secret consists only of a single, unversioned string with no metadata of any kind, and can only ever be accessed as a plain environment variable with no volume-mounting option available.
B. Secrets can only be accessed by hardcoding their value directly into the application's source code before building the container image, since Cloud Run provides no runtime mechanism for accessing Secret Manager.
C. A secret consists of metadata (such as replication locations, labels, and permissions) plus secret versions that store the actual secret data as a text string or binary blob; you can mount the secret as a volume (available to the container as a file, always fetching the latest value from Secret Manager on read) or pass it as an environment variable (resolved once at instance startup, so it's recommended to pin it to a specific version rather than using "latest").
D. A secret is identical in every respect to a regular Cloud Run environment variable, with no distinct service, versioning, or access-control mechanism involved.

---

## Answer Key & Explanations

**1. Correct answer: B.**
IAM inspects the API request and identifies the calling application by the credentials in the request. It then checks policy bindings in the IAM policy attached to the target resource (such as a Pub/Sub topic) to determine what operations are allowed for that identity, rejecting the call if it isn't authorized.

**2. Correct answer: D.**
A policy binding binds one or more members (identities) to a single role. A role contains a set of permissions that allows the member identity to perform specific actions on Google Cloud resources. A member can be attached to multiple policy bindings within an IAM policy, enabling that member to have more than one role.

**3. Correct answer: A.**
IAM supports human identities (a Google Account, which can be part of a group or domain), service accounts (used by machines or applications, such as a virtual machine, a Cloud Run service, or a Cloud Run function), and "all users" (a special identifier to allow everyone or public access to a service).

**4. Correct answer: C.**
A service account is a special type of account used by machines, applications, or services, identified by its unique email address. Service accounts differ from user accounts in that they don't have passwords and can't sign in using browsers or cookies, other users or service accounts can be let to act on their behalf, and they aren't members of your Google Workspace domain (though they can be added to groups).

**5. Correct answer: D.**
If you run code on a virtual machine, in a Cloud Run service, or as part of a Cloud Build build, you have access to a built-in service account, which client libraries automatically use for authentication when connecting to Google Cloud APIs. You can always replace this service account with your own user-managed service account, which is recommended.

**6. Correct answer: A.**
Every Cloud Run service or job is linked to a service account, also known as the "service identity." By default, Cloud Run services or jobs run as the default Compute Engine service account with the Editor role.

**7. Correct answer: C.**
If a client library is used in application code to call a Google Cloud API, the library automatically acquires appropriate tokens to authenticate the code's requests using the service's runtime service account; when accessing most Google APIs, OAuth 2.0 access tokens are used. IAM verifies the access token and checks whether there's a policy binding with the required roles on the target resource.

**8. Correct answer: B.**
For asynchronous communication between Cloud Run services, various Google Cloud services such as Cloud Tasks, Pub/Sub, Cloud Scheduler, or Eventarc can be used. For synchronous communication, a service calls another service's endpoint URL directly over HTTP; it's a best practice to use IAM and an individual service identity for the calling service, granted the minimum set of permissions required.

**9. Correct answer: A.**
To set up synchronous access, you configure the receiving service to accept requests from the calling service by making the calling service's service account a principal on the receiving service, then grant that service account the Cloud Run Invoker (`roles/run.invoker`) role — for example, via the `gcloud run services add-iam-policy-binding` command with the calling service's identity as the member and `roles/run.invoker` as the role.

**10. Correct answer: C.**
The request made by the calling service must present proof of its identity in the form of a Google-signed OpenID Connect token. OpenID Connect is an identity protocol based on OAuth 2.0 that enables identity verification of a client based on the authentication performed by an authorization server, and it's also used to obtain basic profile information about the client.

**11. Correct answer: B.**
Google Cloud resources are organized hierarchically: the organization resource is the root node providing central visibility and control, folders are an optional grouping mechanism beneath the organization (which can map to departments, teams, or business units), and the project is the base-level entity required to create resources, use Cloud APIs and services, manage permissions, and enable billing. Each resource has exactly one parent, except the top organization node, which has none.

**12. Correct answer: D.**
Every resource in the hierarchy has an IAM Policy, and permissions are granted on it using policy bindings — for example, granting the "Pub/Sub Publisher" role via a policy binding on a specific topic's attached IAM policy.

**13. Correct answer: C.**
If a policy binding is added to a higher-level resource like a project instead of an individual Pub/Sub topic, it's inherited by lower-level resources — in this case, the Pub/Sub topics inherit the policy binding from their parent resource, the Google Cloud project. This is useful when you need to grant permission to publish messages on all topics in a project, rather than just a single topic.

**14. Correct answer: B.**
When IAM evaluates access to a resource, it also evaluates policy bindings from the parent resource (and that parent's own ancestors). The effective IAM policy on the resource includes those bindings granted to a parent, which you cannot take away at the lower level.

**15. Correct answer: D.**
There are three types of IAM roles: basic roles (Owner, Editor, Viewer) are very powerful roles with permissions that span all services and shouldn't be granted by default in production environments; predefined roles (such as Cloud Run Admin, Pub/Sub Publisher, or Cloud Tasks Enqueuer) provide granular roles to use a specific service and are managed by Google Cloud; custom roles let you build your own role from a list of permissions for your specific use case.

**16. Correct answer: A.**
The default service account used by Cloud Run is the Compute Engine service account, which has the broad Editor role on the project. Because of policy binding inheritance, this default service account has read and write permissions on most resources in the project — while convenient, this is an inherent security risk, since resources can be created, modified, or deleted using this service account.

**17. Correct answer: B.**
To mitigate this security risk, the module recommends: creating a new service account for a Cloud Run service, configuring that service account as the Cloud Run service's identity, and adding policy bindings for that identity with predefined or custom roles only on the resources the service needs to access.

**18. Correct answer: D.**
By default, a newly created service account doesn't have any permissions — it doesn't yet appear in any policy binding. If any Google Cloud API is called from code running as part of the Cloud Run service before a policy binding grants the account a role, the call will be rejected by IAM.

**19. Correct answer: A.**
Cloud Run environment variables are set as key-value pairs, injected into the application container, and accessed by application code at runtime — using functions such as `os.environ.get("key")` in Python, `process.env.key` in Node.js, or `System.getenv("key")` in Java. Default environment variables can be set in the container with the `ENV` statement in a Dockerfile, and an environment variable set with the same name on a Cloud Run service or job overrides the value set in that default.

**20. Correct answer: C.**
A secret in Secret Manager is an object that contains a collection of metadata (such as replication locations, labels, and permissions) and secret versions, which store the actual secret data — such as an API key or password — as a text string or binary blob. A secret can be made accessible to a Cloud Run service or job either by mounting it as a volume (available to the container as a file, with reads always fetching the value from Secret Manager, suitable for use with the latest version) or by passing it as an environment variable (resolved at instance startup time, so it's recommended to pin it to a particular version rather than using latest).
