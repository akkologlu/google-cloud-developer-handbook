# Module 17 — Introduction to Containers: Practice Questions

This set covers container and container image concepts (the archive-vs-runtime-instance distinction, what happens to the file system and network when a container runs, per-language requirements, system dependencies, and container configuration including the default root user), building images with Docker (the "build inside the staged image" model, the `FROM`/`COPY`/`RUN` instructions, and the `CMD`/`ENTRYPOINT` relationship), building images with Buildpacks (the detect and build phases, builders, and the `pack` CLI), CI/CD tooling (Skaffold's workflow and `skaffold.yaml`, Cloud Build's automatic nature and `cloudbuild.yaml` steps, Cloud Build trigger types, and Artifact Registry), and best practices (why base images are bloated and how Distroless multi-stage builds fix it, PID 1 and signal handling, Docker's build cache, and vulnerability scanning with Container Analysis). This module is Module 1 of the "Developing Containerized Applications on Google Cloud" course — a new course in the handbook, following the 16-module "Developing Applications with Cloud Run Functions on Google Cloud" course.

The questions are weighted toward the distinctions that actually trip people up: why a container only exists while its processes are running, why the default container user is root unless explicitly changed, why `RUN` can only execute programs already present in the staged image, why a builder needs both a detect and a build phase, why Cloud Build's automation requires no direct input once triggered, why a bloated base image is a security risk and not just a size inconvenience, why only PID 1 can receive termination signals, and why Docker's build cache breaks as soon as one earlier step changes.

Try to answer all questions first, then check your answers against the [Answer Key & Explanations](#answer-key--explanations) section below.

---

## Questions

**1.** A team has a container image sitting in Artifact Registry that has never been run. Separately, they deployed it once, and the deployment later scaled down to zero running instances. In which of these situations does a "container" (as opposed to a container image) actually exist?

A. A container exists in both situations, since a container image and a container are the same thing under different names.
B. Neither situation has a container — a container image is a template/archive of files, while a container is a runtime instance of that image representing running processes; if there are no running processes, there is no container.
C. A container exists only in the first situation (sitting unused in the registry), since containers are created the moment an image is pushed, not when it's run.
D. A container exists only in the second situation (after scaling to zero), because scaling to zero converts the image into a permanent container object.

**2.** When a container actually starts running from a container image, what happens to its file system and its ability to receive network traffic?

A. The container shares the host machine's file system directly with no isolation, and network access requires manually assigning a public IP address first.
B. Nothing changes — a container has no independent file system or network interface; it uses the host's file system and network stack directly.
C. A new container image is generated from the running container's file system, and network access remains disabled until configured after startup.
D. The container image's contents seed a private file system for the container, and the container's processes get access to a virtual network interface with a local IP so the application can bind to it and listen on a port for incoming traffic.

**3.** A Dockerfile never sets a `USER` instruction. What user does the resulting container actually run as by default, and why does the module flag this as worth paying attention to?

A. The root user (system administrator) is used by default, which the module says is not a security best practice.
B. A randomly generated, unprivileged user ID is automatically assigned to every container by default.
C. The container refuses to start entirely until a `USER` instruction is explicitly added to the Dockerfile.
D. The user is inherited from whichever account happens to be logged into the host machine running Docker at that moment.

**4.** A team is comparing what has to go into a container image for a Java application versus a Go application. What does the module say distinguishes these two?

A. Both languages require the exact same set of files at runtime — source code, a runtime, and separately installed library dependencies.
B. Go applications always require a separate runtime to be installed in the image, in exactly the same way Java requires a JVM.
C. Java source code must be compiled first, and only the compiled binaries plus the Java Virtual Machine are needed at runtime — the source code itself is not; a Go application, by contrast, compiles its dependencies and source code together into a single binary, which can even have static assets embedded into it.
D. Java applications never need a runtime inside the container image at all, only the compiled class files by themselves.

**5.** An application needs a headless browser to turn HTML into a PDF, and separately needs ImageMagick to process images. How does the module categorize needs like these?

A. These must be declared as ordinary application library dependencies, exactly like an npm package or a Python pip package.
B. Containers cannot support needs like these at all — they require a separate virtual machine outside the container entirely.
C. These are automatically bundled into every base image by default, regardless of whether a given application actually needs them.
D. These are system dependencies — needs that can't be expressed as an application library dependency — and the module lists exactly these kinds of examples: a headless browser, tools like curl/tar/zip, extra system fonts, ImageMagick, and OpenOffice.

**6.** What does the module mean when it says that with Docker, you build your application "inside" the container image, and what role does the `FROM` instruction play in that process?

A. You put a container image on a stage, and every subsequent Dockerfile instruction changes that staged image; `FROM` starts this process by downloading a base image — one pre-installed with the tools needed to build and run software — from a registry onto that stage.
B. The application is built entirely on the host machine outside of any image, and the finished binary is only copied into an otherwise-empty image at the very end using `FROM`.
C. `FROM` uploads your completed container image to a registry once the build finishes, rather than downloading anything at the start.
D. Each Dockerfile instruction produces a completely separate, unrelated image with no connection to the instructions that came before it.

**7.** What exactly does the `COPY` instruction do in a Dockerfile, and what term does Docker use for the set of files it can pull from?

A. `COPY` creates a full duplicate of the staged image as a second, identical image, purely for backup purposes.
B. `COPY` downloads a new base image from a registry, functioning identically to the `FROM` instruction.
C. `COPY` pulls files into the staged image from the source code directory, which Docker refers to as the "build context."
D. `COPY` runs a program that's already present on the staged image to modify files already inside it, without pulling in anything from outside the image.

**8.** What does the module say about which programs the `RUN` instruction is able to execute, and which files those programs can access?

A. `RUN` can execute any program present anywhere on the host machine's file system, whether or not that program exists inside the container image.
B. `RUN` lets you run a program from the image, on the image, to update files — but the program file you execute must already be present in the container image, and the only files accessible to it are files that already exist in the container image.
C. `RUN` exists only to define environment variables and never actually executes a program of any kind.
D. `RUN` automatically has unrestricted network access to download any file from the internet, even without a networking tool present in the image.

**9.** What's the relationship between the `CMD` and `ENTRYPOINT` instructions as described by the module?

A. `ENTRYPOINT` points to the program file to start and run the container as an executable, while `CMD` provides defaults for the executing container, including the command to run when it starts; if no executable command is otherwise specified, `ENTRYPOINT` is required.
B. `CMD` and `ENTRYPOINT` are two names for the exact same instruction, and a Dockerfile that uses both instructions will always fail to build.
C. `ENTRYPOINT` is used to set environment variables, while `CMD` is used to set the container's working directory.
D. `CMD` only applies to images built with Buildpacks, and `ENTRYPOINT` only applies to images built with Docker.

**10.** When a builder processes a source code directory, what do its detect phase and build phase actually each do?

A. The detect phase compiles the source code directly, and the build phase only verifies that the resulting binary is syntactically valid.
B. Both phases perform the exact same work — the process simply runs twice, purely as redundancy in case the first pass fails for any reason.
C. The detect phase runs against the source code to determine whether a given buildpack applies (for example, a Python buildpack checking for a `requirements.txt` file) — if detection fails, the build phase for that buildpack is skipped; the build phase then sets up the build/run-time environment, downloads dependencies, compiles the source if needed, and sets an appropriate entry point.
D. The detect phase runs exactly once per builder for its entire lifetime, not per project, and its result is cached forever regardless of what source code is later supplied.

**11.** What is the `pack` command-line tool, and what does it require in order to turn a source directory into a container image?

A. `pack` is a Dockerfile linter that validates Dockerfile syntax before a Docker Build runs, and it has no relationship to buildpacks at all.
B. `pack` is a command-line tool maintained by the Cloud Native Buildpacks project; it needs a builder — which contains one or more buildpacks — to turn a source directory into a container image, and it can work with builders from multiple projects, such as Google Cloud's buildpacks, Paketo Buildpacks, or Heroku Buildpacks.
C. `pack` additionally requires a Dockerfile alongside a builder, since buildpacks are described as simply a thin wrapper around ordinary Dockerfile builds.
D. `pack` is restricted to working only with Google Cloud's buildpacks and is architecturally incapable of using a builder from any other project.

**12.** Across Skaffold's multi-stage workflow, and in a `skaffold.yaml` file, what do the `build` and `deploy` sections each configure?

A. Skaffold is described as only capable of building container images, with no deployment capability of any kind — deployment must always be handled by a completely separate, unrelated tool.
B. The `build` section of `skaffold.yaml` defines where to deploy the image, while the `deploy` section defines which Dockerfile to build it with.
C. Skaffold requires a Kubernetes cluster to already be running before it can build any container image, even for a purely local Docker-only workflow.
D. Skaffold detects source code changes, builds artifacts (using tools like Dockerfiles or Buildpacks, locally or via Cloud Build), tests and tags them, then renders and deploys manifests to a target like Kubernetes, Docker, or Cloud Run; the `build` section of `skaffold.yaml` defines how the container image is built, and the `deploy` section defines how it's deployed (for example, with Kustomize).

**13.** What does the module say about how much manual input the Cloud Build process itself requires once started, and what must every step in a `cloudbuild.yaml` file contain?

A. Cloud Build requires manually approving each individual build step in the console before it's allowed to proceed to the next one.
B. Each step in `cloudbuild.yaml` must contain a `dockerfile` field rather than a `name` field, since Cloud Build is described as only supporting Docker-based builds.
C. The process of building a container image with Cloud Build is entirely automatic and requires no direct input from you once triggered; all resources run in your own user project with build logs available through Cloud Logging, and every step must contain a `name` field specifying a cloud builder — a container image that runs a common tool, such as `docker`.
D. Cloud Build always executes entirely outside of your own Google Cloud project, on infrastructure whose logs you cannot access at all.

**14.** A team wants three separate things automated: builds firing automatically on GitHub pushes, builds firing in response to image push/tag/delete events on Artifact Registry, and builds triggered from an external source-code system Cloud Build doesn't natively integrate with. Which trigger type does the module recommend for each?

A. A single manual trigger is described as sufficient for all three cases, since manual triggers can be configured to fire automatically on any kind of event.
B. A repository trigger connected to the GitHub repo (firing on push/tag/PR events) for the first case, a Pub/Sub trigger for the second case (since Artifact Registry and Cloud Storage events can publish to Pub/Sub), and a webhook trigger for the third case, which authenticates and processes incoming webhook events at a custom URL to connect external systems such as GitLab or Bitbucket.com.
C. None of these three scenarios can be automated with Cloud Build triggers at all — only fully manual builds are described as possible for external or Pub/Sub-driven scenarios.
D. A repository trigger is described as the correct choice for all three cases, since Pub/Sub and webhook triggers are described as being used only for Cloud Build's own internal logging.

**15.** A Node.js application's container image, built with a straightforward `FROM node:16`-based Dockerfile, ends up around 950 MB — even though the Node.js runtime and the app's own dependencies together total under 100 MB. What does the module say is responsible for the rest, and what does it recommend as the fix?

A. The extra size comes from the application's own source code and static assets, which the module says are simply large by nature in any Node.js project.
B. This is described as a display bug in whatever tool was used to measure the image's size, with the actual runtime footprint already being minimal.
C. The extra size is described as unavoidable overhead that Docker itself always adds to every image, regardless of which base image is chosen.
D. The `node` base image ships extra system packages meant for building software rather than running it (such as the Debian package manager `apt-get`, compilers, and version-control tools), which is a security risk in production; the fix is a multi-stage build that repeats the `FROM` instruction to stage a minimal Distroless image, then copies over only the application and its dependencies with `COPY --from=<previous stage>`.

**16.** Why does the module say a container's application process should be launched directly via `CMD` or `ENTRYPOINT` rather than, say, through a wrapper shell script, and what does it say you must additionally do in your application code?

A. Container platforms such as Docker, Kubernetes, and Cloud Run can only send signals (for example, to terminate a process) to the process holding PID 1 inside the container; launching the process directly via `CMD`/`ENTRYPOINT` lets it receive those signals and shut down gracefully, and because signal handlers aren't automatically registered for PID 1, you must implement and register them yourself in application code.
B. This is described purely as a performance optimization unrelated to signals — shell wrapper scripts are said to always run measurably slower for any workload.
C. This is described as relevant only to HTTP applications; event-driven or background-processing containers are said to be entirely unaffected by which process holds PID 1.
D. Signal handlers for the process with PID 1 are described as being registered automatically by the container runtime, so no application code changes are ever required.

**17.** Docker looks for reusable layers in its build cache for each Dockerfile instruction. What condition does the module say must hold for that cache to actually be used, and what does it recommend positioning late in a Dockerfile as a result?

A. The build cache is described as being entirely independent per instruction, so changing an early instruction never affects whether a later instruction's cache can be reused.
B. The build cache can be used for an instruction only if every instruction before it in the Dockerfile also used the cache; since source code is usually what changes with each new build, the module recommends adding it to the image as late as possible in the Dockerfile.
C. The build cache only applies to the very first instruction in a Dockerfile (`FROM`) and has no effect on any instruction that comes after it.
D. The module recommends adding source code as early as possible in the Dockerfile, since the build cache is described as working best when frequently changing instructions come first.

**18.** What service does the module describe for scanning container images for vulnerabilities, when is it triggered automatically, and what lets you scan an image manually?

A. Vulnerability scanning is described as requiring a completely separate, third-party product with no Google Cloud integration of any kind.
B. Manual scanning is described as impossible — only automatic scanning on push is supported, with no on-demand option at all.
C. Vulnerability metadata is described as being permanently unavailable through any API, viewable only inside the Google Cloud console by a human.
D. Container Analysis scans images stored in Artifact Registry for vulnerabilities and stores the resulting metadata, made available through an API; it can be triggered automatically whenever a new image is pushed to Artifact Registry, and the on-demand scanning API additionally lets you manually scan images stored in a registry or stored locally, for example via the `gcloud artifacts docker images scan` command.

**19.** Beyond using a multi-stage build with Distroless, what other best practices does the module recommend for keeping a container image secure and efficient, and why?

A. Run the application as a non-root user (to avoid attackers modifying root-owned files via a package manager, and to allow disabling `sudo` or using read-only containers), remove unnecessary tools and utilities to reduce the attack surface, build the smallest image possible to reduce upload/download time, and standardize on common base images across a team so the layers only need to be downloaded once.
B. Always run the application as the root user in production, since the module describes this as required for the application to have full access to its own file system.
C. Include every development and debugging tool available in the production image, since the module describes this as making incident response faster at the cost of image size.
D. Avoid standardizing on shared base images across a team, since the module describes each project needing its own unique, independently-downloaded base image for isolation.

**20.** What is Artifact Registry, and how does it relate to Cloud Build in the module's description?

A. Artifact Registry is described as a local-only cache that exists solely on a developer's individual machine, with no relationship to Cloud Build whatsoever.
B. Artifact Registry is described as being able to store only Docker container images, and explicitly not any other kind of software package.
C. Artifact Registry is a Google Cloud service used to store and manage software artifacts — including container images and software packages — in private repositories; it's the recommended container registry for Google Cloud and integrates with Cloud Build to store the packages and container images produced by Cloud Build's builds.
D. Artifact Registry is described as a real-time vulnerability scanner that replaces the need for a separate registry service entirely.

---

## Answer Key & Explanations

**1. Correct answer: B.**
A container image is a template — practically, an archive of files — that defines how a container instance is realized at runtime; a container is a runtime instance of that image, representing the application's running processes. If there are no running processes, there is no container, so neither an unused image sitting in a registry nor a deployment scaled to zero running instances has an actual container at that moment.

**2. Correct answer: D.**
When a container runs, two things happen beyond simply executing a program: the contents of the container image are used to seed a private file system for the container (all the files the application's processes can see), and the processes in the container get access to a virtual network interface with a local IP, letting the application bind to that interface and listen on a port for incoming traffic.

**3. Correct answer: A.**
Container configuration includes the user to run the program with; if it's not set, the root user (system administrator) is used as the default, which the module explicitly says is not a best practice for security reasons.

**4. Correct answer: C.**
An application written in Java needs to be compiled first — the source code is no longer required to run the application, but the compiled binaries (plus the Java Virtual Machine) are. A Go application, by contrast, has its dependencies and source code compiled together into a single binary, and static assets can even be embedded directly into that binary.

**5. Correct answer: D.**
Some applications have dependencies on system tools that can't be expressed as an application library dependency; the module's own examples include a headless browser (e.g. to turn HTML into a PDF), tools such as curl/tar/zip, additional system fonts, ImageMagick (for processing images), and OpenOffice (for converting document formats).

**6. Correct answer: A.**
With Docker, you build your application inside the container image: you put a container image on a stage, and every Dockerfile instruction changes that staged image. The `FROM` instruction starts this process by downloading a base image — pre-installed with tools needed to build and run software — from a registry onto that stage, to be modified by the instructions that follow.

**7. Correct answer: C.**
The `COPY` instruction pulls in source code; Docker refers to the set of files in the source code directory as the "build context," and `COPY` is used to bring those files into the staged image that was downloaded with the `FROM` instruction.

**8. Correct answer: B.**
The `RUN` instruction lets you run a program from the image, on the image, meaning: the program file you execute needs to already be present in the container image, and the only files accessible to that program are those that already exist in the container image — for example, to install a system package, download library dependencies, or compile source code into binaries.

**9. Correct answer: A.**
`ENTRYPOINT` points to the program file to start and run the container as an executable. `CMD` provides defaults for an executing container, including the command to run when the container starts; if the executable command isn't specified via `CMD`, the `ENTRYPOINT` instruction is required.

**10. Correct answer: C.**
If a builder starts to process a source directory, it executes two phases of a buildpack. The detect phase runs against the source code to determine if a buildpack is applicable (e.g. a Python buildpack looking for a `requirements.txt` or `setup.py` file, or a Node buildpack looking for `package-lock.json`); if detection fails, the build phase for that specific buildpack is skipped. The build phase then runs against the source code to set up the build-time and run-time environment, download dependencies and compile source code if needed, and set an appropriate entry point and startup scripts.

**11. Correct answer: B.**
`pack` is a command-line tool maintained by the Cloud Native Buildpacks project. It needs a builder — distributed as an OCI image that can contain one or more buildpacks — to turn a source directory into a container image, and it can work with builders from multiple projects that use the buildpacks standard, including Google Cloud's buildpacks, Paketo Buildpacks, and Heroku Buildpacks.

**12. Correct answer: D.**
Skaffold uses a multi-stage workflow: it detects source code changes in the project, builds artifacts with the tool of choice (Dockerfiles, Cloud Native Buildpacks, and others, built locally or remotely with Cloud Build), tests and tags the artifacts, then renders and deploys manifests to targets such as Kubernetes, Docker, or Cloud Run. In `skaffold.yaml`, the `build` section defines configuration for how the container image should be built (e.g. which Dockerfile to use), and the `deploy` section defines configuration for how the container should be deployed (e.g. using Kustomize).

**13. Correct answer: C.**
The process of building a container image with Cloud Build is entirely automatic and requires no direct input from you once started; all resources used in the build process execute in your own user project, and you have access to all build logs through Cloud Logging. Instructions in a `cloudbuild.yaml` file are written as a set of steps, and each step must contain a `name` field specifying a cloud builder — a container image that runs common tools, such as an image running Docker.

**14. Correct answer: B.**
A Cloud Build trigger automatically starts a build whenever changes are made to source code in a connected Google Cloud Source repository, GitHub, or Bitbucket repository — matching the first scenario. Cloud Build Pub/Sub triggers enable builds in response to Pub/Sub events, with use cases including Artifact Registry and Cloud Storage events like pushing, tagging, or deleting images — matching the second scenario. Webhook triggers authenticate and process incoming webhook events at a custom URL, letting external source code management systems such as GitLab or Bitbucket.com connect to Cloud Build — matching the third scenario.

**15. Correct answer: D.**
The Node.js runtime is roughly 80 MB and the app plus library dependencies are under 1 MB, yet the image built directly from the `node` base image reaches around 950 MB because that base image comes packed with system packages needed to build software rather than run it — including the Debian package manager `apt-get`, build tools like the GCC compiler and make, version control tools, and more — which is a security risk in production since these extra packages might contain vulnerabilities. The fix Docker offers is a multi-stage build: repeating the `FROM` instruction to stage a new image from the Distroless project (which contains only runtime, not build-time, dependencies), then using `COPY --from=<previous stage>` to bring over just the application and its dependencies.

**16. Correct answer: A.**
The first process launched in a container gets PID 1, and container platforms such as Docker, Kubernetes, and Cloud Run can only send signals — most notably to terminate processes — to the process with PID 1 inside a container. Because of this, you must launch your container's application process with the `CMD` or `ENTRYPOINT` instruction in your Dockerfile, which allows that process to receive signals and gracefully shut down when terminated. Additionally, because signal handlers aren't automatically registered for the process with PID 1, you must implement and register these signal handlers in your own application code.

**17. Correct answer: B.**
Docker can use its build cache for an image only if all previous build steps also used it. Because a new Docker image is usually built for each new version of the source code, the module recommends adding the source code to the image as late as possible in the Dockerfile — and more generally, positioning build steps that involve frequent changes near the bottom of the file — so that unrelated, unchanged earlier steps can still hit the cache.

**18. Correct answer: D.**
Container Analysis is a Google Cloud service that provides vulnerability scanning and metadata storage for containers; the scanning service performs vulnerability scans on images in Artifact Registry, then stores the resulting metadata and makes it available through an API. When enabled, it can automatically scan an image and is triggered whenever a new image is pushed to Artifact Registry. With the on-demand scanning API — for example, via the `gcloud artifacts docker images scan` command — you can also manually scan container images stored in these registries or stored locally.

**19. Correct answer: A.**
Beyond multi-stage builds with Distroless, the module recommends: avoiding running the app as the root user inside the container (to prevent attackers from modifying root-owned files using a package manager, and to support disabling `sudo` or launching the container read-only); removing unnecessary tools and utilities to reduce the app's attack surface; building the smallest image possible to reduce upload and download times; and creating images with common, standard base images across an organization, so each base image only needs to be downloaded once and only the unique layers need to be built afterward.

**20. Correct answer: C.**
Artifact Registry is a Google Cloud service used to store and manage software artifacts in private repositories, including container images and software packages; it's the recommended container registry for Google Cloud, and it integrates with Cloud Build to store the packages and container images produced from Cloud Build's builds.
