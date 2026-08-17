# Modül 17 – Introduction to Containers

> Bu, "Developing Containerized Applications on Google Cloud" kursunun Modül 1'idir — handbook'taki yeni bir kurs, 16 modülden oluşan "Developing Applications with Cloud Run Functions on Google Cloud" kursunun ardından geliyor.

---

# Genel Bakış

Bu modülde ele alınan beş alan:

```text
Container kavramları → Docker ile build → Buildpacks ile build → CI/CD araçları → Best practice'ler
```

---

# Container'lar ve Container Image'lar

| Terim | Tanım |
| --- | --- |
| **Container** | Uygulama kodunun, çalışması için gereken bağımlılıklarla (programlama dili runtime'ları, yazılım kütüphaneleri) birlikte paketlenmesi |
| **Container image** | Bir container instance'ının runtime'da nasıl gerçekleneceğini tanımlayan bir şablon; pratikte, dosyalardan oluşan bir arşiv (sistem kütüphaneleri, executable'lar, kaynaklar, kaynak dosyaları) — self-contained, uygulamanın çalışması için gereken her şeyi içerir |
| **Container** (runtime) | Bir container image'ın runtime örneği — uygulamanın çalışan process'lerini temsil eder; çalışan process yoksa, container da yoktur |

Bir container çalıştığında:
- Container image içeriği, container için **private bir dosya sistemi** seed eder.
- Process'ler, bağlanıp (bind) bir portu dinleyebileceği, local IP'li bir **sanal network interface** alır.

**Image'a ne girebilir:** sistem paketleri, runtime, kütüphane bağımlılıkları, kaynak kod, binary'ler, statik varlıklar, configuration. Minimal bir image, tek bir program dosyası + onu çalıştıracak bir komut olabilir.

**Dile göre gereksinimler:**

| Dil | Runtime | Bağımlılık kurulumu | Notlar |
| --- | --- | --- | --- |
| Node.js | Node.js | `npm install` | — |
| Python | Python | `pip install -r requirements.txt` | — |
| Java | JVM | Maven / Gradle | Kaynak kod **compile edilmelidir**; runtime'da sadece compiled binary'ler gerekir |
| Go | Gerekmez (statik binary) | `go get` | Bağımlılıklar + kaynak kod tek bir binary'e compile edilir; asset'ler embed edilebilir |

Bazı uygulamalar, kütüphane bağımlılığı olarak ifade edilemeyen **sistem bağımlılıklarına** da ihtiyaç duyar (headless browser, curl/tar/zip, ek fontlar, ImageMagick, OpenOffice).

**Container configuration** — container'ı bir process olarak nasıl başlatacağın:

| Ayar | Amaç |
| --- | --- |
| Entrypoint | Container başladığında çalıştırılacak komut |
| Environment variable'lar | Uygulamaya configuration geçirmek için |
| Working directory | Programın çalışacağı dizin |
| User | Programı kimin çalıştıracağı — ayarlanmazsa varsayılan **root**'tur (güvenlik best practice'i değildir) |

**Her yerde çalıştır:** Compute Engine, Kubernetes, Cloud Run (Google Cloud) ya da Docker / Podman (local).

---

# Docker ile Build Etmek

Build ve paketleme adımları: sistem bağımlılıklarını kur → runtime kur → kütüphane bağımlılıklarını indir → binary'leri compile et → dosyaları image'a paketle → container configuration'ı ayarla.

**Docker Build**, kaynak kod + bir **Dockerfile** (kaynak kodu image'a dönüştürmek için bir manifest/script) alır. Uygulamayı image'ın *içinde* build edersin: bir "stage" üzerinde bir base image ile başlarsın, ve her talimat o sahnelenmiş image'ı değiştirir.

| Talimat | Ne yapar |
| --- | --- |
| `FROM` | Başlanacak bir base image'ı bir registry'den indirir (örn. `node`, `golang`) |
| `COPY` | **Build context**'ten (kaynak dizini) sahnelenmiş image'a dosya çeker |
| `RUN` | Image'daki bir programı image **üzerinde** çalıştırıp dosyaları günceller (paket kurma, bağımlılık indirme, compile etme) |
| `ENTRYPOINT` | Container'ı bir executable olarak çalıştırmak için program dosyasına işaret eder |
| `CMD` | Varsayılan komutu sağlar; `ENTRYPOINT` ile bir executable belirtilmemişse zorunludur |
| `ENV` | Environment variable ayarlar |
| `WORKDIR` | Working directory'yi ayarlar |
| `USER` | Programı hangi kullanıcıyla çalıştıracağını ayarlar |

```dockerfile
FROM docker.io/library/node:16 AS build
WORKDIR /app
COPY * /app
RUN npm install --production
CMD [ "node", "server.js" ]
```

---

# Buildpacks ile Build Etmek

**Buildpacks**, kaynak kodunu **Dockerfile yazmadan** bir container image'a dönüştürür. Kaynak koduna dayalı deployment'ı mümkün kılmak için Cloud Run'a gömülüdür.

- **Builder** adı verilen OCI image'larında dağıtılır ve çalıştırılır. Her builder bir ya da daha fazla buildpack içerebilir; builder'lar birden fazla dili destekler.
- **Detect fazı:** bir buildpack'in uygulanabilir olup olmadığını kontrol eder (örn. `requirements.txt`, `package-lock.json` arar). Başarısız olursa, o buildpack için build fazı skip edilir.
- **Build fazı:** build/run-time ortamını kurar, bağımlılıkları indirir, gerekirse compile eder, entry point/başlangıç script'ini ayarlar.

```shell
pack build --builder gcr.io/buildpacks/builder:v1 --path ./source-dir sample-app
```

`pack`, bir builder'ı kaynak kodla eşleştirip bir image üreten CLI aracıdır (Cloud Native Buildpacks projesi tarafından bakımı yapılır). Builder seçenekleri: **Google Cloud's buildpacks** (App Engine, Cloud Functions, Cloud Run tarafından dahili olarak kullanılır; Go, Java, Node.js, Python, .NET Core'u destekler; açık kaynaktır), **Paketo Buildpacks**, **Heroku Buildpacks** — hangisiyle build edilirse edilsin, Cloud Run çalıştırabilir.

---

# CI/CD Araçları

| Araç | Amaç |
| --- | --- |
| **Skaffold** | Container'lar için CI/CD/continuous development'ı orkestre eden, açık kaynaklı bir Google CLI'ı; workflow: değişiklikleri tespit et → artifact'leri build et → test et → tag'le → manifestleri render edip deploy et → log'ları tail et/port'ları forward et → temizle. `skaffold.yaml` ile yapılandırılır (`build` + `deploy` bölümleri; ortam başına configuration için `profiles` desteklenir). Kubernetes'e (ham YAML, helm, kpt, kustomize), Docker'a, ya da Cloud Run'a deploy eder. |
| **Artifact Registry** | Google Cloud'un container image'lar ve yazılım paketleri için önerilen registry'si; Cloud Build ile entegre olur. |
| **Cloud Build** | Build'leri Google Cloud üzerinde çalıştırır (tamamen otomatik — kendi projende çalışır, log'lar Cloud Logging üzerinden). Repository/storage'dan kaynak kod içe aktarır, `cloudbuild.yaml` configuration'ındaki `steps`'e (her biri bir `name` builder image'ı ve `args` içerir) göre artifact'ler (Docker container'ları, Java arşivleri) üretir. Herhangi bir dili, GitHub/Bitbucket/GitLab entegrasyonunu, ve Cloud Run/GKE/Cloud Functions/Anthos/Firebase'e deployment'ı destekler. |

**Build'leri çalıştırmak:** `gcloud builds submit --tag $REPO/my-image .` (Dockerfile ile) ya da `--config=cloudbuild.yaml .` — Buildpacks ile de çalışır (Dockerfile'sız).

**Cloud Build trigger'ları** (önce repository'yi bağlamalısın):

| Trigger türü | Ne zaman tetiklenir |
| --- | --- |
| Repository trigger | Cloud Source Repositories, GitHub, ya da Bitbucket'ta push/tag/PR event'leri (bir Dockerfile ya da Cloud Build config dosyası gerektirir) |
| Manual trigger | Manuel olarak tetiklenir, örn. çekilen kodu başka bir ortama deploy etmek için |
| Pub/Sub trigger | Pub/Sub event'leri (örn. Artifact Registry / Cloud Storage push, tag, delete) |
| Webhook trigger | Özel bir URL'e gelen webhook event'leri — GitLab, Bitbucket.com gibi harici SCM sistemlerini bağlar |

---

# Best Practice'ler

**Image boyutu — base image'lar şişirilmiştir:** naif bir Node.js Dockerfile build'i ~950 MB'a ulaşabilir, çünkü `node` base image'ı runtime'da ihtiyaç duyulmayan build tooling'i (apt-get, GCC, git, curl, Python, Perl, Bash) taşır — production'da bir güvenlik riski.

**Çözüm: Distroless ile multi-stage build** — bir **Distroless** image'ı (sadece runtime bağımlılıkları) sahnelemek için `FROM` talimatını tekrarla, ardından uygulamayı ve bağımlılıklarını son sahneye `COPY --from=<önceki-sahne>` ile kopyala. Örnekteki image'ı ~80 MB'a indirir.

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

**Process ve signal handling:** container platformları (Docker, Kubernetes, Cloud Run) signal'ları (örn. sonlandırma) yalnızca **PID 1** olan process'e gönderebilir. Process'ini (bir shell wrapper değil) `CMD`/`ENTRYPOINT` ile başlat ki signal'ları alsın ve graceful shutdown yapabilsin; signal handler'ları uygulama kodunda kendin kaydet — PID 1 için otomatik değildir.

**Docker build cache:** her talimat yeniden kullanılabilir bir katman oluşturur, ama bir talimatın cache'i, yalnızca **önceki tüm** adımlar da cache'i kullandıysa kullanılabilir. Sık değişen adımları (özellikle kaynak kod kopyalamayı) Dockerfile'da **mümkün olduğunca sona** koy.

**Diğer best practice'ler:**

| Pratik | Neden |
| --- | --- |
| Mümkün olan en küçük image'ı build et | Yükleme/indirme süresini azaltır |
| Uygulamayı non-root bir kullanıcı olarak çalıştır | Saldırganların bir package manager ile root'a ait dosyaları değiştirmesini önler; `sudo`'yu devre dışı bırak/kaldır; read-only container'ları düşün |
| Ortak katmanlarla image'lar oluştur | Bir kez indirilen standart base image'lar, ekip genelinde build süresini azaltır |
| Gereksiz araçları kaldır | Saldırı yüzeyini azaltır |

**Vulnerability scanning ve patching:**

- **Container Analysis**, Artifact Registry'deki image'ları zafiyetler için tarar ve metadata'yı bir API üzerinden saklar; push'ta otomatik tetiklenir, ya da **on-demand scanning** ile manuel çalıştırılır.
- Otomatik patching akışı: Artifact Registry'de scanning'i etkinleştir → zafiyetlerden job/Pub/Sub bildirimiyle haberdar ol → rebuild tetikle → CI/CD ile yeniden build edilen image'ı staging'e deploy et → test et → production'a deploy et.
- Bir Cloud Build pipeline'ında on-demand scanning: Cloud Build service account'una `roles/ondemandscanning.admin` rolünü ver, build step'inden sonra `gcloud artifacts docker images scan <image>` çalıştır, ve belirtilen önem derecesinde zafiyet bulunursa build'den çık.

---

# Modül Özeti

Bu modül, bir uygulamayı bir **container image**'a nasıl paketleyeceğini (kavramlar, Docker, Buildpacks), bu süreci nasıl otomatikleştireceğini (Skaffold, Artifact Registry, trigger'lı Cloud Build), ve sonuçtaki image'ı nasıl küçük, güvenli, ve doğru davranan hale getireceğini (multi-stage/Distroless build'ler, PID 1 signal handling, build cache sıralaması, non-root kullanıcılar, vulnerability scanning) öğretiyor.

---

# Anahtar Noktalar

- Bir container image, statik bir arşiv/şablondur; container, onun runtime örneğidir — çalışan process yoksa container da yoktur.
- Container configuration, açıkça ayarlanmadıkça kullanıcıyı varsayılan olarak **root** yapar.
- Docker, uygulamayı sahnelenmiş image'ın *içinde* build eder: `FROM` (base image) → `COPY` (build context) → `RUN` (build/kurulum) → configuration talimatları (`ENTRYPOINT`/`CMD`/`ENV`/`WORKDIR`/`USER`).
- Buildpacks, kaynak kodu bir Dockerfile olmadan, **detect** ve ardından **build** fazlarıyla bir image'a dönüştürür, OCI **builder**'larında dağıtılır; `pack build --builder ... --path ...` kullanılır.
- Cloud Build kendi projende tamamen otomatik çalışır; `cloudbuild.yaml` step'leriyle yapılandırılır, repository/manual/Pub-Sub/webhook trigger'larıyla otomatikleştirilir.
- `node:16` gibi base image'lar build-time tooling ile şişirilmiştir — image boyutunu ciddi şekilde azaltmak için **Distroless** ile **multi-stage build** kullan.
- Platformlar yalnızca **PID 1**'i sinyalleyebilir — uygulamanı doğrudan `CMD`/`ENTRYPOINT` ile başlat, ve graceful shutdown için signal handler'ları kendin kaydet.
- Docker'ın build cache'i, yalnızca önceki her adım da cache'e isabet ettiyse geçerlidir — sık değişen adımları (kaynak kod kopyalama gibi) en sona koy.
- Otomatik ya da on-demand vulnerability scanning için **Container Analysis**'i kullan, ve patching'i image'ı build eden aynı CI/CD pipeline'ı üzerinden otomatikleştir.
