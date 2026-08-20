# Modül 21 – Application Development, Testing, and Integration

> `deep-dive/19-fundamentals-of-cloud-run/` ve `deep-dive/20-service-identity-and-authentication/` ile aynı kursun (Cloud Run odaklı kurs) üçüncü modülü. Local'de geliştirme/test etmeyi, deployment ve revision yönetimini, ve diğer Google Cloud servisleriyle entegrasyonu kapsar.

---

# Genel Bakış

```text
Development & testing → Deployment ve revision yönetimi → Google Cloud servisleriyle entegrasyon
```

---

# Uygulaman Cloud Run İçin Uygun mu?

Şunların **tümünü** karşılamalıdır:

- HTTP/HTTP2/WebSockets/gRPC isteklerini, stream'lerini, ya da event'lerini serve eder — ya da tamamlanana kadar çalışır (bir job).
- Local **persistent** bir dosya sistemi gerekmez (ephemeral local FS ya da network FS uygundur).
- Birden fazla instance'ın aynı anda çalışmasını handle edecek şekilde build edilmiştir.
- Instance başına ≤8 CPU ve ≤32 GiB bellek.
- Containerized'dır, ya da Go/Java/Node.js/Python/.NET'te yazılmıştır (ya da başka şekilde containerize edilebilir).

---

# Container Runtime Contract

| Gereksinim | Detay |
| --- | --- |
| Herhangi bir dil, herhangi bir base image | Executable'lar **Linux 64-bit** için compile edilmelidir |
| Desteklenen image formatları | Docker Image Manifest V2 (Schema 1/2), OCI |
| Service olarak | Doğru portta istekleri dinle; yapılandırılmış timeout içinde yanıt ver (**maks. 1 saat**, startup süresi dahil) yoksa istek **504** ile sonlanır |
| Job olarak | Bir portu dinlememeli ya da web sunucusu başlatmamalıdır; başarıda `0`, başarısızlıkta sıfır olmayan exit code ile çık |
| TLS | Kendin implement etme — Cloud Run, HTTPS/gRPC için TLS'i terminate eder ve HTTP/1 ya da gRPC olarak proxy'ler; HTTP/2 isteklerini cleartext formatta handle et |

---

# Execution Environment'lar

| | First generation | Second generation |
| --- | --- | --- |
| Varsayılan | Service'ler (değiştirilebilir) | Job'lar (değiştirilemez) |
| En uygun olduğu durum | Hızlı scale-out, kısa cold start, seyrek trafik, <512 MiB bellek | Network dosya sistemi, daha yavaş cold start'lara tolerant istikrarlı trafik, CPU-yoğun workload'lar, sorun yaratan implement edilmemiş sistem çağrıları |
| Sistem çağrıları | Çoğunu emüle eder (hepsini değil) | Tam Linux uyumluluğu (tüm syscall'lar, namespace'ler, cgroup'lar) |
| Ekstralar | — | Daha hızlı CPU/network performansı, network dosya sistemi desteği |

---

# Dosya Sistemi ve Data Storage

- **In-memory dosya sistemi**: yazılabilir, container'ın tahsis edilmiş belleğini kullanır, instance durduğunda **persist etmez** — cache, atılabilir istek-başı veri için iyidir.
- **Network dosya sistemi**: instance ömrünün ötesinde kalıcı olan standart dosya sistemi semantikleri için Filestore ya da self-managed seçenekler kullan — **second generation** execution environment gerektirir. Cloud Storage, **Cloud Storage FUSE** ile mount edilebilir.
- **Dosya sistemi gerekmiyorsa**: Firestore, Cloud SQL, Spanner, Cloud Storage, Memorystore, BigQuery'ye bağlanmak için cloud data storage client kütüphaneleri kullan.

---

# Local Geliştirme

**Cloud Code** — tam geliştirme döngüsü için IDE plugin'leri (VS Code, IntelliJ, Cloud Shell): örnek şablonlar, local bir **Cloud Run emulator** (CPU/bellek, env variable'lar, Cloud SQL bağlantıları yapılandır), tek tıkla **Deploy to Cloud Run** (local'de ya da Cloud Build ile build et, push et, deploy et, canlı URL gösterilir).

| Local test aracı | Nasıl |
| --- | --- |
| Cloud Code | IDE extension'ı + Cloud Run emulator |
| gcloud CLI | Local dev ortamı; varsa bir Dockerfile'dan build eder, yoksa Google Cloud's buildpacks'ten; kaynak değişikliklerinde otomatik yeniden build eder; `http://localhost:8080/`'da test et |
| Docker | Image URL'i ve dinleme portuyla `docker run`; `http://localhost:port/`'da test et |

---

# Container'ları Build Etmek ve Deploy Etmek

| Araç | Rolü |
| --- | --- |
| Docker | Bir Dockerfile ile local'de build et (`docker build`), bir repo'ya push et (`docker push`) |
| Cloud Build | Google Cloud'da bir Dockerfile ya da Google Cloud's buildpacks ile build et (`gcloud builds submit`, buildpacks için `--pack` ekle) |
| Cloud Run | `gcloud run deploy --source`, kaynaktan build eder (varsa Dockerfile'dan, yoksa buildpacks'ten), image'ı yükler, ve deploy eder — ya da `pack` komutuyla buildpacks ile local'de build et |

**Deployment, Artifact Registry (ya da Docker Hub) erişimi gerektirir** — Google, Artifact Registry'yi önerir; diğer registry'ler bir Artifact Registry remote repository gerektirir. Desteklenmeyen registry image'larını önce `docker push` ile push et.

**Artifact Registry**: universal package manager (Docker, NPM, Maven, PyPI repository'leri); Google Cloud için önerilen registry; Cloud Build ile entegre olur. Container image'larını barındırmak için bir **Docker repository** oluştur — her image benzersiz bir URL alır (örn. `us-central1-docker.pkg.dev/${PROJECT_ID}/my-repo/my-image`).

**Pull davranışı**: Cloud Run image'ı Artifact Registry'den pull eder ve hızlı, güvenilir başlangıç için **kendi internal storage'ına kopyalar** — büyük image'lar küçükler kadar hızlı yüklenir, ve image'ı yanlışlıkla Artifact Registry'den silmek çalışan servisi bozmaz.

**Bir servis oluşturmak/güncellemek**: Owner, Editor, ya da hem Cloud Run Admin hem Service Account User role'lerini (ya da eşdeğer bir custom role) gerektirir. İlk deploy, servisi **ve** ilk revision'ını oluşturur — servis başına bir container image vardır.

---

# Revision'ları Yönetmek

**Güncelleme akışı (5 adım):** kodu değiştir → image'ı build edip paketle → Artifact Registry'ye push et → servise yeniden deploy et → Cloud Run'ın deploy etmesini bekle.

**Service configuration (8 bileşen)** — bunlardan **herhangi birini** değiştirmek, image değişikliği olmasa bile yeni bir revision oluşturur:

1. Container image URL
2. Container entrypoint ve argümanları
3. Secret'lar ve environment variable'lar
4. Request timeout
5. Concurrency
6. CPU/bellek limitleri
7. Scaling boundaries (min/max instance)
8. Google Cloud configuration (service account, connector'lar)

Sonraki revision'lar, açıkça değiştirilmedikçe bu ayarları otomatik olarak miras alır.

**Revision'lar immutable'dır** — her deploy, service resource'un yeni, immutable bir kopyasını oluşturur (container image + configuration); sadece yeni revision'lar eklenebilir.

**Yeni bir revision'da trafik davranışı:**
1. Cloud Run, yeni revision'ı mevcut revision'ın kapasitesine kadar scale up eder, instance'larının başlamayı bitirmesini bekler — bu sırada eski revision serve etmeye devam eder.
2. Hazır olduğunda, trafik yeni revision'a yönlendirilir; her iki revision da bağımsız autoscale eder; eski olan idle olur ve sonunda sıfıra ölçeklenir.
3. Kademeli bir rollout için, `--no-traffic` ile (başlangıçta %0) deploy et, sonra yüzdeyi kademeli olarak artır.

| Özellik | Amaç |
| --- | --- |
| **Pinning** | Deployment'ı trafik migration'ından ayırmak için bir revision'ın trafik yüzdesini 100'e ayarla — rollback ya da migration öncesi test için kullanışlı |
| **Tagging** | Bir revision'a, trafik göndermeden önce test etmek için trafiksiz bir test URL'i ver (`https://TAG---service-xyz.a.run.app`) |
| **Splitting** | Her revision'a yapılandırılabilir bir yüzde istek yönlendir (console/gcloud/YAML/Terraform); trafik değişiklikleri anlık değildir — devam eden istekler tamamlanır, geçiş sırasında ya bir revision'a ya diğerine gidebilir |
| **Session affinity** | Aynı client'ın isteklerini aynı revision'ın container instance'ına yönlendirme çabası (varsayılan olarak kapalı) |

---

# Google Cloud Servisleriyle Entegrasyon

**Client kütüphaneleri**: servisin service account'u aracılığıyla şeffaf bir şekilde kimlik doğrular. **Built-in** hesap, geniş **Project Editor** role'üne sahiptir — erişimi bir **per-service identity** (minimal izinli service account) ile kısıtla, örn. yalnızca okuma amaçlı Firestore erişimi için Firestore User.

**Memorystore (Redis)**: **Serverless VPC Access** ile bağlan — Redis instance'ının yetkili VPC network'ünü belirle, servisle aynı region'da bir connector oluştur, connector'ı o network'e attach et, sonra `--vpc-connector` ve `REDISHOST`/`REDISPORT` env variable'larıyla deploy et.

**Cloud Run Integrations**: bir servisi Memorystore'a bağlamayı (ya da özel bir domain'i eşlemeyi) otomatikleştiren basitleştirilmiş bir console/CLI akışı — Redis cache'ini, yeni bir revision'ı, ve networking/env variable configuration'ını otomatik oluşturur.

**Pub/Sub tetikleme**: bir push subscription, mesajları servis endpoint'ine HTTP istekleri olarak teslim eder (endpoint private kalabilir, IAM ile korunur). Servis **600 saniye** içinde acknowledge etmelidir yoksa Pub/Sub yeniden teslim eder. Kurulum: bir topic oluştur → mesajı çıkarıp 200/204 (başarı, acked) ya da 400/500 (hata, retry edilir) döndürecek kod ekle → **Cloud Run Invoker**'a sahip bir service account oluştur → o hesaba bağlı, servis URL'ine işaret eden bir push subscription oluştur.

**Cloud SQL**:

| Yol | Mekanizma |
| --- | --- |
| Public IP (varsayılan) | Service account'un Cloud SQL Client ya da Cloud SQL Admin'e ihtiyacı var; `--add-cloudsql-instances=<connection-name>` ile deploy/güncelle; Cloud Run, şifrelemeyle **Cloud SQL Auth proxy** (network socket'ları ya da dile özgü connector'lar) üzerinden bağlanır |
| Private IP | Tüm egress'i bir **Serverless VPC Access** connector'ı üzerinden yönlendir |

Best practice'ler: credential'ları **Secret Manager**'da sakla, env variable ya da mount edilmiş bir volume olarak geçir; otomatik yeniden bağlanmak ve bağlantıları sınırlamak için bir **connection pool** kullan (Cloud Run servisleri, bir Cloud SQL veritabanına servis başına **100 bağlantıyla** sınırlıdır).

---

# Modül Özeti

Bu modül, "kod yazmak" ile "güvenilir bir production servisi işletmek" arasındaki köprüyü kurar: bir uygulamanın Cloud Run'a uyup uymadığı, karşılaması gereken container runtime contract'ı, execution environment'lar ve local test seçenekleri, image'ların Artifact Registry'den Cloud Run'ın internal storage'ına nasıl taşındığı, revision'ların nasıl versiyonlandığı ve aralarındaki trafiğin nasıl kontrol edildiği (pinning, tagging, splitting), ve bir servisin Pub/Sub, Memorystore, ve Cloud SQL'e nasıl güvenle bağlandığı.

---

# Anahtar Noktalar

- Cloud Run uygunluğu, beş kriterin tümünü aynı anda karşılamayı gerektirir.
- Job'lar bir portu dinlememelidir (bir exit code ile çıkarlar); service'ler request timeout içinde yanıt vermelidir.
- TLS'i asla kendin implement etme — Cloud Run onu şeffaf bir şekilde terminate eder.
- First generation, service'in (değiştirilebilir) varsayılanıdır; second generation, job'ın (değiştirilemez) varsayılanıdır ve network dosya sistemlerini destekleyen tek seçenektir.
- In-memory dosya sistemi ephemeral'dır; kalıcılık için Filestore/network FS ya da client kütüphaneleri kullan.
- Cloud Run, her başlangıçta image'ları doğrudan Artifact Registry'den değil, kendi internal storage'ından serve eder.
- Herhangi bir configuration değişikliği — sadece yeni bir image değil — yeni, immutable bir revision oluşturur.
- Yeni bir revision'a trafik kasıtlıdır: önce scale up eder, sonra (isteğe bağlı olarak `--no-traffic` ile kademeli) migrate eder, devam eden istekler asla düşürülmez.
- Pinning, tagging, ve splitting, sırasıyla rollback, yayın öncesi test, ve kademeli rollout için üç ayrı trafik kontrol aracıdır.
- Built-in service account (Editor role'ü) aşırı geniştir — bunun yerine minimal izinli bir per-service identity kullan.
- Memorystore ve private-IP Cloud SQL, ikisi de Serverless VPC Access gerektirir; public-IP Cloud SQL, Cloud SQL Auth proxy kullanır.
- Pub/Sub push endpoint'lerinin public olması gerekmez — IAM ve Cloud Run Invoker verilmiş bir service account ile koru.
- Cloud SQL bağlantılarını bir connection pool ile sınırla (servis başına 100 sınırı) ve credential'ları Secret Manager'da tut.
