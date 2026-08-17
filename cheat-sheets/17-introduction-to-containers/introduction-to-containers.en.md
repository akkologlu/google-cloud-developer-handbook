# Module 17 – Introduction to Containers

> This is Module 1 of the "Developing Containerized Applications on Google Cloud" course — a new course in the handbook, following the 16-module "Developing Applications with Cloud Run Functions on Google Cloud" course.

---

# Overview

Five areas covered in this module:

```text
Container concepts → Building with Docker → Building with Buildpacks → CI/CD tools → Best practices
```

---

# Containers and Container Images

| Term | Definition |
| --- | --- |
| **Container** | A package of application code together with dependencies (programming language runtimes, software libraries) needed to run it |
| **Container image** | A template defining how a container instance is realized at runtime; practically, an archive of files (system libraries, executables, resources, source files) — self-contained, includes everything the app needs to run |
| **Container** (runtime) | A runtime instance of a container image — represents the application's running processes; if there are no running processes, there is no container |

When a container runs:
- Container image contents seed a **private file system** for the container.
- Processes get a **virtual network interface** with a local IP to bind to and listen on a port.

**What can be in an image:** system packages, a runtime, library dependencies, source code, binaries, static assets, configuration. A minimal image can be a single program file plus a command to run it.

**Per-language requirements:**

| Language | Runtime | Dependency install | Notes |
| --- | --- | --- | --- |
| Node.js | Node.js | `npm install` | — |
| Python | Python | `pip install -r requirements.txt` | — |
| Java | JVM | Maven / Gradle | Source must be **compiled**; only compiled binaries are needed at runtime |
| Go | None (static binary) | `go get` | Dependencies + source compile into a single binary; assets can be embedded |

Some apps also need **system dependencies** not expressible as a library dependency (headless browser, curl/tar/zip, extra fonts, ImageMagick, OpenOffice).

**Container configuration** — how to start the container as a process:

| Setting | Purpose |
| --- | --- |
| Entrypoint | The command to run when the container starts |
| Environment variables | Pass configuration to the app |
| Working directory | Where the program runs |
| User | Who runs the program — defaults to **root** if unset (not a security best practice) |

**Run anywhere:** Compute Engine, Kubernetes, Cloud Run (Google Cloud) or Docker / Podman (local).

---

# Building with Docker

Build & package steps: install system dependencies → install runtime → download library dependencies → compile binaries → package files into the image → set container configuration.

**Docker Build** takes source code + a **Dockerfile** (a manifest/script for turning source into an image). You build the application *inside* the image: start with a base image on a "stage," and every instruction modifies that staged image.

| Instruction | What it does |
| --- | --- |
| `FROM` | Downloads a base image from a registry to start from (e.g. `node`, `golang`) |
| `COPY` | Pulls files from the **build context** (source directory) into the staged image |
| `RUN` | Runs a program **from** the image **on** the image, to update files (install packages, download deps, compile) |
| `ENTRYPOINT` | Points to the program file to run the container as an executable |
| `CMD` | Provides the default command; required if no executable is set via `ENTRYPOINT` |
| `ENV` | Sets environment variables |
| `WORKDIR` | Sets the working directory |
| `USER` | Sets the user to run the program with |

```dockerfile
FROM docker.io/library/node:16 AS build
WORKDIR /app
COPY * /app
RUN npm install --production
CMD [ "node", "server.js" ]
```

---

# Building with Buildpacks

**Buildpacks** turn source code into a container image **without writing a Dockerfile**. Built into Cloud Run to enable source-based deployment.

- Distributed and executed in OCI images called **builders**. Each builder can have one or more buildpacks; builders support multiple languages.
- **Detect phase:** checks if a buildpack applies (e.g. looks for `requirements.txt`, `package-lock.json`). If it fails, the build phase for that buildpack is skipped.
- **Build phase:** sets up build/run-time environment, downloads dependencies, compiles if needed, sets the entry point/startup script.

```shell
pack build --builder gcr.io/buildpacks/builder:v1 --path ./source-dir sample-app
```

`pack` is the CLI tool (maintained by the Cloud Native Buildpacks project) that pairs a builder with source code to produce an image. Builder choices: **Google Cloud's buildpacks** (used internally by App Engine, Cloud Functions, Cloud Run; supports Go, Java, Node.js, Python, .NET Core; open source), **Paketo Buildpacks**, **Heroku Buildpacks** — Cloud Run can run an image built by any of them.

---

# CI/CD Tools

| Tool | Purpose |
| --- | --- |
| **Skaffold** | Google open-source CLI orchestrating CI/CD/continuous development for containers; workflow: detect changes → build artifacts → test → tag → render & deploy manifests → tail logs/forward ports → cleanup. Configured via `skaffold.yaml` (`build` + `deploy` sections; supports `profiles` for per-environment config). Deploys to Kubernetes (raw YAML, helm, kpt, kustomize), Docker, or Cloud Run. |
| **Artifact Registry** | Google Cloud's recommended registry for container images and software packages; integrates with Cloud Build. |
| **Cloud Build** | Executes builds on Google Cloud (fully automatic — runs in your project, logs via Cloud Logging). Imports source from repos/storage, produces artifacts (Docker containers, Java archives) per a `cloudbuild.yaml` configuration of `steps` (each with a `name` builder image and `args`). Supports any language, GitHub/Bitbucket/GitLab integration, and deployment to Cloud Run, GKE, Cloud Functions, Anthos, Firebase. |

**Running builds:** `gcloud builds submit --tag $REPO/my-image .` (Dockerfile) or `--config=cloudbuild.yaml .` — also works with Buildpacks (no Dockerfile needed).

**Cloud Build triggers** (must connect the repo first):

| Trigger type | Fires on |
| --- | --- |
| Repository trigger | Push/tag/PR events in Cloud Source Repositories, GitHub, or Bitbucket (requires a Dockerfile or Cloud Build config file) |
| Manual trigger | Manually invoked, e.g. to deploy fetched code to another environment |
| Pub/Sub trigger | Pub/Sub events (e.g. Artifact Registry / Cloud Storage push, tag, delete) |
| Webhook trigger | Incoming webhook events at a custom URL — connects external SCM systems (GitLab, Bitbucket.com) |

---

# Best Practices

**Image size — base images are bloated:** a naive Node.js Dockerfile build can hit ~950 MB, because the `node` base image ships build tooling (apt-get, GCC, git, curl, Python, Perl, Bash) not needed at runtime — a security risk in production.

**Fix: multi-stage build with Distroless** — repeat the `FROM` instruction to stage a **Distroless** image (runtime-only dependencies), then `COPY --from=<previous-stage>` the app and its dependencies into the final stage. Reduces the example to ~80 MB.

```dockerfile
FROM docker.io/library/node:16 AS build
WORKDIR /app
COPY * /app
RUN npm install --production

FROM gcr.io/distroless/nodejs:16 AS run
WORKDIR /app
COPY --from=build /app /app
USER nonroot
CMD [ "nodejs/bin/node", "server.js" ]
```

**Process & signal handling:** container platforms (Docker, Kubernetes, Cloud Run) can only send signals (e.g. terminate) to the process with **PID 1**. Launch your process via `CMD`/`ENTRYPOINT` (not a shell wrapper) so it receives signals and can shut down gracefully; register signal handlers in application code — they aren't automatic for PID 1.

**Docker build cache:** each instruction creates a reusable layer, but the cache for an instruction is only usable if **all previous** steps also used the cache. Put frequently-changing steps (especially copying source code) as **late as possible** in the Dockerfile.

**More best practices:**

| Practice | Why |
| --- | --- |
| Build the smallest image possible | Reduces upload/download time |
| Run the app as a non-root user | Avoids attackers modifying root-owned files via a package manager; disable/uninstall `sudo`; consider read-only containers |
| Create images with common layers | Standard base images downloaded once reduce build time across a team |
| Remove unnecessary tools | Reduces the attack surface |

**Vulnerability scanning & patching:**

- **Container Analysis** scans images in Artifact Registry for vulnerabilities and stores metadata via an API; auto-triggers on push, or run manually via **on-demand scanning**.
- Automated patching flow: enable scanning on Artifact Registry → job/Pub/Sub notification of vulnerabilities → trigger rebuild → deploy rebuilt image to staging via CI/CD → test → deploy to production.
- On-demand scanning in a Cloud Build pipeline: grant the Cloud Build service account the `roles/ondemandscanning.admin` role, run `gcloud artifacts docker images scan <image>` after the build step, and exit the build if vulnerabilities of a given severity are found.

---

# Module Summary

This module teaches how to package an application into a **container image** (concepts, Docker, Buildpacks), how to automate that process (Skaffold, Artifact Registry, Cloud Build with triggers), and how to make the resulting image small, secure, and well-behaved (multi-stage/Distroless builds, PID 1 signal handling, build cache ordering, non-root users, vulnerability scanning).

---

# Key Points

- A container image is a static archive/template; a container is its runtime instance — no running processes means no container.
- Container configuration defaults the user to **root** unless explicitly set.
- Docker builds the application *inside* the staged image: `FROM` (base image) → `COPY` (build context) → `RUN` (build/install) → configuration instructions (`ENTRYPOINT`/`CMD`/`ENV`/`WORKDIR`/`USER`).
- Buildpacks turn source into an image without a Dockerfile, via a **detect** then **build** phase, distributed in OCI **builders**; use `pack build --builder ... --path ...`.
- Cloud Build runs fully automatically in your own project; configure it with `cloudbuild.yaml` steps and automate it with repository/manual/Pub-Sub/webhook triggers.
- Base images like `node:16` are bloated with build-time tooling — use a **multi-stage build** with **Distroless** to cut image size drastically.
- Platforms can only signal **PID 1** — launch your app directly via `CMD`/`ENTRYPOINT`, and register signal handlers yourself for graceful shutdown.
- Docker's build cache only applies if every prior step also hit the cache — put frequently-changing steps (like copying source) last.
- Use **Container Analysis** for automatic or on-demand vulnerability scanning, and automate patching through the same CI/CD pipeline that built the image.
