# Modül 18 – Introduction to Cloud Run and Google Kubernetes Engine

> Bu, "Developing Containerized Applications on Google Cloud" kursunun Modül 2'sidir, Modül 1'i (Modül 17: Introduction to Containers) takip eder.

---

# Genel Bakış

Bir container image'ı çalıştırmanın üç yolu, azalan yönetilmişlik sırasına göre:

```text
Cloud Run (tam yönetilen, serverless) → Google Kubernetes Engine (yönetilen, ince taneli kontrol) → Container-Optimized OS (VM'i kendin yönetirsin)
```

---

# Cloud Run

**Nedir:** Container'ları doğrudan Google'ın ölçeklenebilir altyapısı üzerinde deploy edip çalıştıran tam yönetilen bir compute platformu. Herhangi bir dil çalışır — tek şart, **64-bit Linux binary'li** bir container image'a compile edilebilmesi.

**Developer workflow (3 adım):** kodu yaz (web isteklerini dinleyen bir sunucu başlatmalı) → build edip bir container image'a paketle → Cloud Run'a deploy et (console, gcloud CLI, YAML, ya da Terraform). Benzersiz bir HTTPS URL alırsın; Cloud Run, tüm gelen istekleri karşılamak için container'ları istek üzerine başlatır ve dinamik olarak ekler/kaldırır.

**İki workflow:**

| Workflow | Nasıl çalışır |
| --- | --- |
| Container-based | Container image'ı kendin build edersin (paketlenen şey üzerinde tam şeffaflık/esneklik) |
| Source-based | Kaynak kodunu deploy edersin (Go, Node.js, Python, Java, .NET Core, Ruby); Cloud Run, senin için **Buildpacks** kullanarak onu build edip bir container image'a paketler |

**HTTPS ve networking:** Cloud Run, geçerli bir TLS sertifikası provizyonlar ve HTTPS terminasyonunu handle eder — uygulaman sadece düz HTTP üzerinden **8080 portunu** (yapılandırılabilir) dinlemelidir; bir container'daki bir process, kendi private, sanal network stack'ine sahiptir. Cloud Run service'leri sürekli çalışır ve web isteklerine/event'lere yanıt verir; **job'lar** bir iş yapar ve bittiğinde sonlanır.

**Pricing:**

| Model | Nasıl çalışır |
| --- | --- |
| Request-based (varsayılan) | Yalnızca istekleri handle ederken kullanılan kaynaklar için, artı startup/shutdown için öde — boştayken hiçbir maliyet yok |
| Instance-based | Container'ın tüm lifecycle'ı için öde; hiç istek olmasa bile CPU her zaman tahsis edilir — steady-state workload'lar için genellikle daha ekonomik |

Fiyat, container'a tahsis edilen vCPU ve bellek miktarına göre ölçeklenir.

---

# Cloud Run Use Case'leri

| Use case | Desen |
| --- | --- |
| REST API / website | İsteğe bağlı olarak veriyi kalıcı hale getirmek için bir veritabanına (örn. PostgreSQL) bağlı bir Cloud Run service'i |
| Karmaşık public site (örn. e-ticaret) | Cloud CDN (performans) ve Google Cloud Armor (content-based trafik filtreleme) ekle; backend bir ilişkisel DB'ye, Redis'e (session'lar), üçüncü parti API'lere bağlanır |
| Mikroservisler | Servisler REST/gRPC (doğrudan request/response) ya da **Pub/Sub** (asenkron, garantili teslimat) üzerinden iletişim kurar — Pub/Sub, push subscription'ları aracılığıyla entegre olur, mesajları kimlik doğrulanmış HTTP istekleri olarak forward eder |
| Event processing | Cloud Run, event-driven workflow'lar inşa etmek için Cloud Storage, Cloud Build, Pub/Sub, Eventarc, ve diğer event kaynaklarıyla entegre olur |
| Zamanlanmış görevler | **Cloud Scheduler** (tam yönetilen bir cron scheduler), bir Cloud Run service'ini güvenle zamanlanmış olarak tetikler — zamanlanmış bir işi container'ın *içinde* çalıştırmak güvenilir değildir, çünkü bir container'ın ömrü yalnızca istekleri handle ederken garanti edilir; servis, yapılandırılmış istek timeout'u içinde tamamlanmalıdır |

---

# Cloud Run'da Yüksek Erişilebilirlik

| Özellik | Detay |
| --- | --- |
| **Revision'lar** | Her deployment, yeni, **immutable** bir revision oluşturur (container image + service configuration — env variable'lar, memory limit'leri, vb.). Hata etkisini azaltmak için revision'lar arasında yüzdeyle trafik böl, stabil bir revision'a geri dön (rollback), ya da yeni revision'a kademeli olarak %100'e kadar geçiş yap. |
| **Autoscaling** | Cloud Run, mevcut tüm instance'lar meşgul olduğunda container instance'ları ekler, talep azaldıkça boşta olanları kapatır. Şunlara göre şekillenir: gelen istek oranı, **CPU utilization** (hedef ~%60), **concurrency** ayarı (instance başına maksimum paralel istek), ve min/max instance sayısı. |
| **Region'lar ve zone'lar** | Cloud Run regional'dır — bir region seçersin (her biri ≥3 zone; bir zone tek bir hata alanıdır). HA için, Cloud Run container'ları region içindeki birden fazla zone'a dağıtır. |
| **Global load balancing** | Global external Application Load Balancer, birden fazla regional Cloud Run service'inin önünde tek bir global IP açığa çıkarır, her client'ı en yakın region'a yönlendirir — erişilebilirliği iyileştirir ve dünya çapında gecikmeyi azaltır. |
| **Portability** | Container'lar doğası gereği her yerde çalışır; Cloud Run ayrıca **Knative** (açık kaynak) ile API-uyumludur, aynı container runtime contract'ını implement eder — bu sayede uygulamalar Kubernetes tabanlı ortamlarda da çalışabilir (örn. veri egemenliği ya da vendor lock-in'den kaçınmak için). |

**Dikkat edilmesi gerekenler:** autoscaling maliyete yol açar (max instance ile sınırla), hızlı scale-up downstream sistemlerin throughput kapasitesini aşabilir, ve VM tabanlı workload'ların Cloud Run ya da GKE'deki container'lara taşınması için bir migration planı gerekir.

---

# Google Kubernetes Engine (GKE)

**Nedir:** GKE, Google'ın tam yönetilen **Kubernetes** servisidir — Kubernetes, deployment, ölçeklendirme, ve yönetimi otomatikleştirmek için açık kaynaklı container orkestrasyon sistemidir (orijinal olarak Google tarafından tasarlanmış, artık **CNCF** tarafından bakımı yapılıyor). GKE; cluster oluşturma, load balancing, autoscaling, otomatik node upgrade/repair, ve Google Cloud'un operations suite'i üzerinden logging/monitoring'i yönetir.

## Cluster Mimarisi

| Bileşen | Rol |
| --- | --- |
| **Control plane** | Cluster'ın node'larında çalışan her şeyi yönetir: workload'ları zamanlar, yaşam döngülerini/ölçeklendirmelerini/upgrade'lerini, ve network/storage kaynaklarını yönetir. Kubernetes API server'ı (`kube-apiserver`) çalıştırır — tüm cluster bileşenlerinin konuştuğu merkez (hub), HTTP/gRPC, `kubectl`, ya da console üzerinden erişilebilir. **Zonal cluster** = tek bir zone'da tek bir control plane; **regional cluster** = birden fazla zone'da birden fazla control plane replica'sı (daha erişilebilir). |
| **Node'lar** | Containerized workload'larını çalıştıran Compute Engine VM'leri. Her biri bir container runtime ve Kubernetes node agent'ı (`kubelet`) çalıştırır — bu, control plane ile konuşur ve zamanlanan container'ları başlatır/çalıştırır. |
| **Pod** | Kubernetes'teki en küçük deployable birim — storage/network paylaşan, bir ya da daha fazla container, ve bunların nasıl çalıştırılacağına dair bir spesifikasyon. Nadiren doğrudan oluşturulur (genellikle Deployment'lar ya da Job'lar aracılığıyla). Ephemeral'dır: tamamlanana, silinene, tahliye edilene, ya da node başarısız olana kadar node'unda kalır. |

## Temel Kubernetes Kaynakları

| Kaynak | Amaç |
| --- | --- |
| **Deployment** | Bir **ReplicaSet** (istenen replica sayısı) aracılığıyla pod'ları deklaratif olarak oluşturur/yönetir; Deployment Controller, gerçek durumu kontrollü bir hızda istenen duruma yaklaştırır. YAML'da tanımlanır: replica sayısı, bir selector label, ve container'ları içeren bir pod template'i. |
| **Service** | Bir label ile seçilen pod kümesi için stabil bir network soyutlaması (ömrü boyunca sabit IP + load balancing) — gereklidir çünkü ephemeral pod'lar yeniden oluşturuldukça pod IP'leri değişir. Türler arasında `LoadBalancer` (externally accessible), `ClusterIP`, `NodePort` bulunur. Bir selector, type, ve `port`/`targetPort` içeren `kind: Service` YAML'ı ile tanımlanır. |
| **Volume** | Bir pod'daki tüm container'lara erişilebilir bir dizin. **Ephemeral** türler (ConfigMap, Secret) pod'un ömrünü paylaşır; **durable** türler (PersistentVolume) bağımsız var olur ve verisini pod sonlandıktan sonra da korur. |
| Diğerleri | DaemonSet, StatefulSet, Job'lar, ve custom resource türleri de mevcuttur. |

Kubernetes nesneleri **imperatif ya da deklaratif** olarak yapılandırılabilir; Kubernetes, bildirilen istenen durumu sürekli olarak korumaya çalışır.

## GKE İçin Geliştirme

- **Cloud Code** — uygulamaları Google Cloud ile oluşturmak, deploy etmek, ve entegre etmek için IDE plugin'leri (Kubernetes/Cloud Run explorer'larıyla).
- **`kubectl`** — cluster kaynaklarını ve workload'ları yönetmek için CLI.
- **CI/CD pipeline'ı:** geliştir (Cloud Code + bir kaynak repository) → build/test (**Cloud Build**, image'ı yeniden build eder, **Artifact Registry**'de saklar, build artifact'lerini Cloud Storage'da saklar, testleri çalıştırır) → **Google Cloud Deploy** ile bir **staging** GKE cluster'ına deploy et → onaydan sonra **production**'a terfi ettir → GKE üzerinde yönet → Google Cloud'un operations suite'i ile izle.

---

# Container-Optimized OS

Google tarafından bakımı yapılan, açık kaynaklı **Chromium OS** projesine dayanan, Docker container'ları çalıştırmak için optimize edilmiş, Compute Engine VM'leri için bir işletim sistemi image'ı.

| Faydalar | Sınırlamalar |
| --- | --- |
| Kutudan çıktığı gibi container çalıştırır (Docker + `cloud-init` önceden kurulu, host üzerinde kurulum yok) | Package manager dahil değildir (debugging/admin araçları için CoreOS toolbox kullan) |
| Daha küçük saldırı yüzeyi | Containerized olmayan uygulamaları desteklemez |
| Varsayılan olarak kilitli (firewall + güvenlik ayarları) | Üçüncü parti kernel modülü/driver kuramazsın |
| Otomatik haftalık güncellemeler (sadece bir reboot gerekir) | Google Cloud dışında desteklenmez |

**Ne zaman kullanılır:** minimal kurulumla Docker container'ları, küçük footprint'li güvenli bir OS, ya da Compute Engine instance'larında Kubernetes çalıştırman gerektiğinde. **Ne zaman kullanılmaz:** uygulaman containerized değilse, mevcut olmayan kernel modüllerine/driver'lara/paketlere ihtiyaç duyuyorsa, ya da Google Cloud dışında tam destek gerekiyorsa.

---

# Modül Özeti

Bu modül, bir container image'ı çalıştırmak için bir spektrumdaki üç noktayı kapsıyor: **Cloud Run** (tam yönetilen, serverless, pay-per-use, mikroservisler, event processing, ve zamanlanmış görevler dahil web isteklerine hizmet veren uygulamalar için ideal), **Google Kubernetes Engine** (cluster'lar, control plane/node'lar, pod'lar, ve Deployment/Service/Volume gibi deklaratif kaynaklar aracılığıyla ince taneli kontrol sunan, yönetilen bir Kubernetes servisi), ve **Container-Optimized OS** (kendi Compute Engine instance'larını yönettiğinde kullanılan, minimal, container odaklı bir VM image'ı). Her biri, operasyonel kontrolü yönetilmiş kolaylıkla farklı şekilde takas eder.

---

# Anahtar Noktalar

- Cloud Run uygulamaları sadece düz HTTP üzerinden 8080 portunu dinlemelidir — TLS terminasyonu tamamen Cloud Run proxy'si tarafından handle edilir.
- Cloud Run'da bir container çalıştırmanın tek gerçek kısıtlaması: 64-bit Linux binary'sine compile edilebilmesi.
- Request-based pricing'de boşta maliyet yoktur; instance-based pricing CPU'yu her zaman tahsisli tutar ve steady-state workload'lar için daha ucuz olabilir.
- Bir container'ın ömrü yalnızca istekleri handle ederken garanti edilir — işi container içinde değil, Cloud Scheduler ile harici olarak zamanla.
- Cloud Run service revision'ları immutable'dır; traffic splitting rollback ya da kademeli rollout'u mümkün kılar.
- Autoscaling; istek oranı, ~%60 CPU utilization hedefi, concurrency, ve min/max instance'lara göre şekillenir — sadece istek hacmine değil.
- Bir region ≥3 zone içerir, her biri tek bir hata alanıdır; Cloud Run, HA için container'ları zone'lara dağıtır.
- Cloud Run'ın Knative uyumluluğu, uygulamaları Kubernetes tabanlı ortamlara taşınabilir kılar.
- GKE'de control plane, `kube-apiserver` üzerinden her şeyi zamanlar/yönetir; node'lar (`kubelet` ile) container'ları fiilen çalıştırır.
- Zonal bir cluster'ın tek bir zone'da tek bir control plane'i vardır; regional bir cluster'ın zone'lar arasında birden fazla control plane replica'sı vardır (daha yüksek erişilebilirlik).
- Bir pod, en küçük deployable Kubernetes birimidir, ephemeral'dır, ve genellikle doğrudan değil bir Deployment ya da Job aracılığıyla oluşturulur.
- Pod'lar yeniden oluşturuldukça pod IP'leri değişir — bir Service'in sabit IP'si ve load balancing'i tam olarak bunu çözmek için vardır.
- Volume'ler ya ephemeral'dır (ConfigMap, Secret — pod'un ömrüne bağlı) ya da durable'dır (PersistentVolume — bağımsız, veri hayatta kalır).
- Container-Optimized OS'ta package manager yoktur, containerized olmayan uygulamaları çalıştırmaz, üçüncü parti kernel modüllerini reddeder, ve yalnızca Google Cloud'da çalışır.
