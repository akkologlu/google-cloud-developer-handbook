# Module 14 — Securing Cloud Run Functions: Practice Questions

This set covers identity-based access control (authentication vs. authorization, service accounts vs. user accounts, OAuth 2.0 access tokens vs. ID tokens, the Cloud Functions Admin/Developer/Invoker/Viewer IAM roles, who can invoke event-driven vs. HTTP functions, the developer testing flow, runtime service accounts, and function-to-function calls using `roles/run.invoker` and an audience-scoped ID token), network-based access control (ingress settings, egress settings with Serverless VPC Access, and VPC Service Controls), and protecting data with Cloud KMS customer-managed encryption keys (CMEK) — what CMEK protects, how it's set up, the primary-version-only constraint, and what happens when a key is destroyed or disabled. This module is Module 3 of the "Developing Applications with Cloud Run Functions on Google Cloud" course, following Module 2 (Module 13).

The questions are weighted toward the distinctions that actually trip people up: why functions are private by default, the fixed order of authentication before authorization, which identity the `roles/run.invoker` role belongs on in a function-to-function call, what the ID token's `aud` field actually restricts, the difference between ingress and egress settings, and why destroying a CMEK key doesn't instantly kill running work.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A team deploys a new Cloud Run function without specifying any authentication settings. What is the default behavior, and what must a caller provide?

A. The function is private and requires authentication by default; a caller must be authenticated and authorized to invoke it.
B. The function is public by default with no authentication required, unless the team explicitly locks it down.
C. The function is private by default, but any authenticated Google Account (from any project) can invoke it without further authorization checks.
D. Authentication defaults depend on the function's trigger type — HTTP functions are public by default, event-driven functions are private by default.

**2.** In Cloud Run functions' identity-based access control model, what is the correct sequence of steps, and what does the second step evaluate?

A. Authorization happens first to check the caller's IAM roles, then authentication validates the identity credential.
B. Authentication first validates the identity credential; once confirmed, authorization evaluates the identity's level of access or permissions.
C. Authentication and authorization happen simultaneously as a single combined step, with no defined order.
D. Only authorization is performed; Cloud Run functions does not perform a separate authentication step.

**3.** A workload identity represents a VM, and a separate identity represents an individual person who is part of a Google Group. Which identity kinds are these, respectively?

A. Both are user accounts, since Cloud Run functions treats all workload and human identities the same way.
B. The VM is a user account and the person is a service account — the reverse of typical usage.
C. The VM's identity is a service account (a non-person identity); the person's identity is a user account (an individual Google Account holder or part of a Google Group).
D. Cloud Run functions only supports service accounts; user accounts cannot be used to invoke functions at all.

**4.** Rather than sending a raw service account credential with every request, clients create a token to authenticate to Cloud Run functions. Why, and what two token types does Cloud Run functions use?

A. Tokens are used purely for performance reasons; the two types are refresh tokens and session cookies.
B. Tokens have unlimited lifetimes and are used only to avoid sending credentials over HTTPS; the two types are API keys and bearer tokens.
C. Tokens are optional convenience wrappers around credentials with no security benefit; Cloud Run functions uses SAML assertions and JWTs.
D. Limited-lifetime tokens limit the damage if the underlying credential leaks; Cloud Run functions uses OAuth 2.0 access tokens (for API calls) and ID tokens (for calls to developer-created code), both created via OAuth 2.0 and OIDC.

**5.** A developer is granted the Cloud Functions Invoker role on a function, but not Developer or Admin. What can and can't they do?

A. They can update the function's code and configuration, but cannot delete it.
B. They can invoke (call) the function, but the role does not grant permission to modify or manage the function itself.
C. They can view the function's configuration and logs, but cannot invoke it directly.
D. They can delete the function, since Invoker implicitly includes all lower-privilege administrative actions.

**6.** A Pub/Sub-triggered, event-driven Cloud Run function needs to be called directly by an external HTTP client for a one-off manual test. Is this possible as described?

A. No — event-driven functions can only be invoked by the event source they are subscribed to, not by an arbitrary external caller.
B. Yes — any Cloud Run function, event-driven or HTTP, can always be invoked directly by an external HTTP client.
C. Yes, but only if the caller supplies an OAuth 2.0 access token instead of an ID token.
D. No — event-driven functions cannot be invoked at all once deployed; they must be redeployed as HTTP functions for any testing.

**7.** A developer wants to manually test an HTTP Cloud Run function that requires authentication. What sequence of steps does the module describe?

A. Generate an OAuth 2.0 access token from any Google Account and pass it in a query parameter named `token`.
B. No token is needed for developer testing; the console provides a built-in bypass for any authenticated Google user.
C. Request a service account key file, embed it in the function's environment variables, and call the function without any header.
D. Have a user account with a role granting the necessary permissions on the function, generate an ID token using that account, and pass the token in the request's `Authorization` header.

**8.** A function is deployed to production using the default Compute Engine service account as its runtime identity, because it was the path of least resistance during a hackathon. What does the module say about this choice?

A. This is the recommended production configuration, since the default service account is scoped tightly to each individual function.
B. Runtime service accounts are irrelevant in production, since functions never access other Google Cloud resources at runtime.
C. The default service account should be used for testing and development only; production functions should use a dedicated runtime service account granted only the minimum permissions needed.
D. The default service account cannot be used at all, even in development — Cloud Run functions requires a custom service account from the first deployment.

**9.** Function A needs to call function B, and only function A (not any other function) should be allowed to invoke B. What configuration achieves this, and what must A present?

A. Grant `roles/run.invoker` on function B to function A's service account, and have A present a Google-signed ID token with the `aud` field set to B's URL in the `Authorization` header.
B. Grant `roles/run.invoker` on function A to function B's service account, and have B present an OAuth 2.0 access token scoped to A's URL.
C. No IAM configuration is needed; any function within the same project can call any other function by default.
D. Grant the Cloud Functions Admin role on function B to function A's service account, since only Admin permits function-to-function calls.

**10.** A team wants a function to accept invocations from Workflows and VPC networks within the same project, but reject requests originating from outside the project or its VPC Service Controls perimeter. Which ingress setting matches this, and does it also allow public internet traffic?

A. Allow all traffic — this is the only setting, and it always allows internet traffic regardless of source.
B. Allow internal traffic and traffic from Cloud Load Balancing — this permits public internet access via the load balancer.
C. There is no ingress setting for this; it can only be achieved by disabling the function's public URL entirely.
D. Allow internal traffic only — internal traffic is defined as traffic from Workflows and VPC networks in the same project or VPC Service Controls perimeter, and public internet traffic is not allowed.

**11.** A function needs its outbound requests to private IP addresses routed through a VPC network, while leaving other outbound traffic to route normally. What must be configured first, and which egress option matches the requirement?

A. Nothing needs to be configured first; egress settings work independently of any VPC connectivity mechanism.
B. A Serverless VPC Access connector must first connect the function to the VPC network; then the "route only requests to private IPs through the connector" egress option matches the requirement.
C. A Cloud NAT gateway must first be created; then the "route all outbound traffic through the connector" option matches the requirement.
D. An ingress setting of "allow internal traffic only" must first be applied; egress settings are only available once ingress is restricted.

**12.** An organization sets up a service perimeter with VPC Service Controls for its Cloud Run functions and configures the required organization policies. What three behaviors does the module say result from this?

A. Functions become fully public regardless of ingress settings, functions no longer need runtime service accounts, and egress traffic is unrestricted.
B. Only HTTP functions are affected; event-driven functions and their egress behavior remain completely unrestricted.
C. HTTP functions only accept traffic originating from a VPC network within the perimeter, all functions must use a Serverless VPC Access connector, and all functions must route all egress traffic through the VPC network.
D. Functions are automatically granted CMEK protection with no further configuration, and ingress/egress settings become irrelevant.

**13.** A security team wants encryption keys for Cloud Run functions that they themselves own and control, rather than keys managed entirely by Google. What are these keys called, and what categories of function data do they protect at rest?

A. Google-managed encryption keys; they protect only the function's environment variables, nothing else.
B. Secret Manager keys; they protect only secrets explicitly referenced by the function's code.
C. Default encryption keys; they protect network traffic in transit, not data at rest.
D. Customer-managed encryption keys (CMEK); they protect function source code, build results (container image and deployed instances), and the at-rest data of internal event transport channels.

**14.** While setting up CMEK for a function, an engineer creates the Artifact Registry repository with a different key than the one used for the function itself, and grants only the Cloud Storage service account access to the key. Is this setup correct?

A. No — the repository must use the same key as the function, and the Cloud Run Functions, Artifact Registry, and Cloud Storage service accounts must each be granted the Cloud KMS CryptoKey Encrypter/Decrypter role on the key.
B. Yes — repositories may use any key independently of the function, and granting access to only one service account is sufficient.
C. No — the repository doesn't need CMEK enabled at all; only the function itself needs a key.
D. Yes — as long as the Cloud Storage service account has access, the other two service accounts inherit access automatically.

**15.** An engineer wants CMEK for a function to always use a specific, pinned key version, even after the key rotates to a new primary version. Does Cloud Run functions support this?

A. Yes — specify the desired key version number when enabling CMEK, and it remains pinned indefinitely.
B. Yes, but only for 1st generation functions; 2nd generation functions always use the primary version.
C. No — Cloud Run functions always uses the key's primary version for CMEK protection; a specific key version cannot be selected.
D. Yes, but only if the key is stored in an HSM cluster rather than as a software key.

**16.** A CMEK key protecting a function's data is destroyed while several executions are already in progress and one function instance is actively running. What happens to those in-flight executions and the running instance, versus new executions?

A. Everything is terminated immediately, including in-progress executions and the active instance, to prevent any further data exposure.
B. Active instances are not shut down and in-progress executions continue to run; new executions, and any execution requiring a new instance, fail as long as the key remains inaccessible.
C. In-progress executions fail immediately, but new executions are permitted to start normally using cached, unencrypted data.
D. Nothing is affected at all, including future executions, because destroyed keys are automatically restored after 24 hours.

---

## Answer Key & Explanations

**1. Correct answer: A.**
Functions are deployed as private by default and require authentication; deploying a function as public and skipping authentication is something you must opt into explicitly.

**2. Correct answer: B.**
Authentication comes first — validating the identity credential to confirm the requestor is who it claims to be. Only once that's confirmed does authorization evaluate the identity's level of access or permissions.

**3. Correct answer: C.**
Service accounts represent non-person identities like a function, application, or VM; user accounts represent people, either as individual Google Account holders or as part of a Google Group.

**4. Correct answer: D.**
Token-based authentication uses tokens with a limited lifetime specifically to limit potential damage if a service or user account credential leaks. Cloud Run functions uses OAuth 2.0 access tokens to authenticate API calls and ID tokens to authenticate calls to developer-created code, both created using the OAuth 2.0 framework and OpenID Connect (OIDC).

**5. Correct answer: B.**
The Cloud Functions Invoker role grants permission to invoke the function; it does not include the ability to modify code, configuration, or manage the function the way Developer or Admin roles do.

**6. Correct answer: A.**
Event-driven functions can only be invoked by the event source they are subscribed to — an external HTTP client cannot call them directly, regardless of what token it presents.

**7. Correct answer: D.**
The developer testing flow requires a user account with a role that grants appropriate permissions on the function, generating an ID token from that account, and passing that token in the `Authorization` header of the request.

**8. Correct answer: C.**
The module explicitly states that the default runtime service account (Compute Engine default service account, or App Engine default service account for 1st gen) should be used for testing and development only; production deployments should specify a dedicated runtime service account with only the minimum permissions required.

**9. Correct answer: A.**
To let one function call another and restrict access to just that caller, you grant `roles/run.invoker` on the receiving function (B) to the calling function's (A's) service account. The calling function must also provide a Google-signed ID token with the `aud` field set to the receiving function's URL, sent in the `Authorization` header.

**10. Correct answer: D.**
"Allow internal traffic only" restricts invocations to internal traffic, which is defined as traffic from Workflows and VPC networks in the same project or VPC Service Controls perimeter — it does not allow traffic originating from the public internet.

**11. Correct answer: B.**
Egress settings require the function to be connected to a VPC network via a Serverless VPC Access connector first; "route only requests to private IPs through the connector" is the egress option that routes just private-IP-bound traffic through the connector while leaving other outbound traffic unaffected.

**12. Correct answer: C.**
With VPC Service Controls organization policies in place, HTTP functions only accept traffic that originates from a VPC network within the service perimeter, all functions must use a Serverless VPC Access connector, and functions must route all egress traffic through the VPC network.

**13. Correct answer: D.**
Customer-managed encryption keys (CMEK) are owned by the customer, not Google, and protect function source code, the results of the build process (the container image and each deployed function instance), and the at-rest data of internal event transport channels.

**14. Correct answer: A.**
The Artifact Registry repository must use the same key as the one used to enable CMEK for the function, and the Cloud Run Functions, Artifact Registry, and Cloud Storage service accounts must each be granted access — specifically the Cloud KMS CryptoKey Encrypter/Decrypter role — on the key.

**15. Correct answer: C.**
Cloud Run functions always uses the primary version of a key for CMEK protection; you cannot specify a particular key version to use when enabling CMEK for a function.

**16. Correct answer: B.**
If a key is destroyed, disabled, or its required permissions are revoked, active function instances are not shut down and executions already in progress continue to run; new executions, and executions that require a new function instance, fail as long as Cloud Run functions does not have access to the key.
