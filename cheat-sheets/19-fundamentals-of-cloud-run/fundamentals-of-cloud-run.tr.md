# Modül 19 – Fundamentals of Cloud Run

> Kaynak materyal, "Developing Containerized Applications on Google Cloud" kursuna (modül 17-18) geçmiş zamanla atıfta bulunuyor — bu, bu modülün, özellikle Cloud Run'a odaklanan yeni, üçüncü bir kursun parçası olduğunu gösteriyor. Bu modül, daha önceki genel bakışın ötesine geçiyor: kaynak modeli, tam container yaşam döngüsü, autoscaling'in iç işleyişi, ve access control.

---

# Genel Bakış

```text
Kaynak modeli → Container yaşam döngüsü → Autoscaling → Access control
```

**Service vs. job:** kod, sürekli olarak bir **service** (web isteklerine/event'lere yanıt verir) ya da bir **job** (iş yapar, sonra quit eder) olarak çalışır. İkisi de aynı ortamda, aynı Google Cloud entegrasyonlarıyla çalışır.

**HTTPS:** Cloud Run, benzersiz bir `*.run.app` subdomain'inde bir TLS sertifikası ve HTTPS endpoint'i provizyonlar (özel domain'ler desteklenir); istekleri handle eder/decrypt eder/uygulamana HTTP üzerinden forward eder. WebSockets, HTTP/2, gRPC destekler. Tek sorumluluğun: bir TCP portunu dinlemek ve HTTP'yi handle etmek.

**Container çalıştırma:** herhangi bir dil çalışır, tek şart bir container image'a paketlenmiş **64-bit Linux binary'sine** compile edilebilmesi.

**Job'lar detaylı:** bir job = bir **job execution** sırasında paralel çalışan bir ya da daha fazla bağımsız **task**; her task bir container instance çalıştırır. Execution'ın başarılı olması için tüm task'ların başarılı olması gerekir (timeout/retry yapılandırılabilir). **Array job'lar**, birden fazla özdeş instance'ı paralel çalıştırır (örn. Cloud Storage'daki birçok dosyayı aynı anda işlemek).

**Tetikleme yöntemleri:**

| Yöntem | Kullanım alanı |
| --- | --- |
| HTTPS | Özel REST API, private mikroservis, HTTP middleware/reverse proxy, paketlenmiş web uygulaması |
| gRPC | Dahili mikroservis iletişimi; yüksek veri yükleri (protocol buffer ile REST'ten 7 kata kadar hızlı); basit servis tanımları; streaming |
| WebSockets | Ekstra configuration gerekmez |
| Pub/Sub (push) | Cloud Storage yüklemesinde veriyi dönüştürmek, export edilen operations-suite log'larını işlemek, custom event publishing |
| Cloud Scheduler | Cron benzeri zamanlanmış servis tetiklemeleri — yedeklemeler, tekrarlayan yönetim görevleri, fatura oluşturma |
| Cloud Tasks | Asenkron task işlemeyi güvenle zamanlamak |
| Eventarc | Cloud Audit Logs üzerinden Cloud Storage/BigQuery/diğer Google Cloud event'lerinden tetiklemek |

---

# Kaynak Modeli

| Kaynak | Detay |
| --- | --- |
| **Service** | Ana Cloud Run kaynağı. Regional'dır; instance'lar region içindeki herhangi bir zone'da başlayabilir (redundancy için zone'lara dağıtılır). Bir proje, region'lar genelinde birçok servis çalıştırabilir. Her biri benzersiz bir endpoint açığa çıkarır ve autoscale eder. |
| **Revision** | Bir container image'ın bir servise her deploy edilişinde oluşturulur. Belirli bir container image + environment config'ten (env variable'lar, memory limit'leri, concurrency) oluşur. **Immutable**'dır — hiç değiştirilmez, sadece yenisiyle değiştirilir. İstekler otomatik olarak en son sağlıklı revision'a yönlendirilir. |
| **Container instance** | Bir revision'a gelen istekleri handle eder; gereken sayıya otomatik ölçeklenir. Aynı anda birden fazla isteği handle edebilir (concurrency ayarına bakın). |
| **Job / Task / Execution** | Bir job bir region'da bulunur; bir job execution tüm task'larını başlatır; her task bir container instance çalıştırır; execution yalnızca tüm task'lar başarılıysa başarılıdır. |

**Region'lar ve zone'lar:** bir region, belirli bir coğrafi konumdur (≥3 zone, her biri tek bir hata alanı); Cloud Run, HA için container'ları bir region içindeki zone'lara dağıtır.

---

# Container Yaşam Döngüsü

```text
Starting → Serving requests ⇄ Idle → Shutting down → Stopped
```

| State | Ne olur |
| --- | --- |
| **Starting** | Cloud Run, container image'ı materialize eder ve uygulamayı başlatır. Dört adım: (1) image'ı container'ın root dosya sistemine materialize et, (2) entrypoint programını çalıştır, (3) uygulama TCP bağlantılarını kabul edene kadar 8080 portunu (yapılandırılabilir; YAML ile HTTP/TCP/gRPC startup & liveness probe'ları ayarlanabilir) sürekli probe et, (4) hazır olduğunda istekleri forward et — portu yalnızca gerçekten hazır olunca aç. |
| **Serving requests** | Container web isteklerini handle ediyor. |
| **Idle** | 100 ms boyunca istek gelmezse girilir. İstekleri serve etmez, **hiçbir ücret yoktur**, CPU neredeyse sıfıra throttle edilir (uygulama çok yavaş çalışır), her an kapatılabilir (graceful-shutdown hook'u vardır). Throttle edilirken arka plan görevleri güvenilmezdir — bunun yerine Cloud Tasks kullan. Üçüncü taraflara network çağrıları büyük olasılıkla başarısız olur. Throttling'i önlemek için CPU'yu **her zaman tahsisli** ayarlayabilirsin (arka plan/asenkron iş için kullanışlı), ama o zaman instance'ın tüm lifecycle'ı için ücretlendirilirsin. |
| **Idle → Serving requests** | Anlıktır — CPU unthrottle edilir ve tam erişim anında geri verilir, hiç gecikme yok. Cloud Run, sıçramaları handle etmek/cold start'ları minimize etmek için bazı instance'ları 15 dakikaya kadar idle tutabilir; **minimum instances**, belirli sayıda instance'ı kalıcı olarak warm tutar. |
| **Shutting down** | Uygulama SIGTERM'i handle ediyorsa, kaldırılmadan önce temizlik yapması için (TCP/DB bağlantılarını ve dosya tanımlayıcılarını kapat, buffer'ları flush et, debug log'u yaz) 10 saniyesi olur. SIGTERM handle edilmiyorsa, container anında durdurulur. |
| **Stopped** | Son state. Normal koşullarda Cloud Run, istekleri serve eden bir container'ı asla durdurmaz — tek istisnalar: uygulama çöker, ya da memory limit'ini aşar (varsayılan 512 MiB, 32 GiB'a kadar yapılandırılabilir). Devam eden istekler başarısız olur; yeniler bir yedek container'ı bekleyebilir. |

**Image nereden gelir:** **deploy**'da, Cloud Run image'ı Artifact Registry'den pull edip kendi internal storage'ına kopyalar (büyük image'ların küçükler kadar hızlı yüklenmesi için optimize edilmiştir). Her container **başlangıcında**, Artifact Registry'den değil, internal storage'dan pull eder — bu, servisleri Artifact Registry kesintilerinden ya da yanlışlıkla image silmekten izole eder.

---

# Autoscaling

- **Internal bir Application Load Balancer**, istekleri bir revision'ın container instance havuzuna dağıtır; tümü meşgulse instance ekler, talep azaldıkça kaldırır (kapatır).
- Instance sayısı şunlara göre şekillenir: istek oranı, **CPU utilization** (~%60 hedef), **maksimum concurrency**, ve **min/maks instance ayarları**. Varsayılan kota sınırı: **1.000 instance** (artış talep edilebilir).
- **Scale to zero:** gelen istek yoksa, son instance bile kapatılır — boştayken hiç ücret yoktur. Bir sonraki istek için yeni bir instance istek üzerine başlar.
- **Cold start / request queuing:** scale-to-zero sonrası ilk istekler, ilk instance başlarken kuyruğa alınır.
- **Minimum instances:** cold-start gecikmesinden kaçınmak için belirli sayıda instance'ı warm/idle tutar — bu idle instance'lar **maliyete yol açar.** `gcloud run services update my-service --min-instances 3`
- **Maximum instances:** maliyet kontrolü ve downstream uyumluluğu için (örn. sınırlı eşzamanlı bağlantı kapasitesine sahip bir DB) toplam instance sayısını sınırlar. Varsayılan: **100**. Çok düşük ayarlamak, Cloud Run'ın tüm trafiği serve etme yeteneğini sınırlar. `gcloud run services update my-service --max-instances 3`
- **Maximum concurrency:** instance başına maksimum eşzamanlı istek. Varsayılan **80**, maksimum **1.000**. Her istek container'ın CPU/belleğinin çoğunu kullanıyorsa, uygulama eşzamanlı istekleri handle edemiyorsa, ya da paylaşılamayan global state'e dayanıyorsa 1'e düşür. Daha yüksek concurrency, container başına bellek ihtiyacını artırır ve sıçramalar sırasında downstream servisleri zorlayabilir. `gcloud run services update my-service --concurrency 1`

---

# Access Control

**Google Cloud, bir API koleksiyonudur** — console, gcloud CLI, Terraform, ve client kütüphanelerinin hepsi bu API'leri çağırır. Bir container image deploy etmek (`gcloud run deploy`), `run.googleapis.com`'a yapılan **bir API çağrısının kendisidir.**

**IAM**, her API çağrısını yetkilendirir, çağıranın kimliğini doğrular ve izinleri kontrol eder — bir revision deploy ediyor olsan da, uygulaman Pub/Sub'a publish ediyor olsa da **aynı mekanizma** çalışır.

| IAM kavramı | Tanım |
| --- | --- |
| **Policy** | Her zaman bir kaynağa bağlıdır; bir policy binding'ler listesidir |
| **Policy binding** | Bir member'ı (kimlik) tek bir role'e bağlar; bir member'ın birden fazla binding'i (birden fazla role'ü) olabilir |
| **Role** | Bir izinler kümesi (örn. Pub/Sub Publisher → `pubsub.topics.publish`) |

**Varsayılan erişim:** yalnızca proje Owner/Editor kimlikleri Cloud Run servislerini ve job'larını oluşturabilir/güncelleyebilir/silebilir/çağırabilir; Owner ve **Cloud Run Admin** (`roles/run.admin`), proje üzerindeki ya da bireysel servis/job'lar üzerindeki IAM policy'lerini değiştirebilir.

**Bir servisi public yapmak:** `allUsers` member'ına **Cloud Run Invoker** role'ünü (`roles/run.invoker`) ver — `gcloud run services add-iam-policy-binding my-service --member="allUsers" --role="roles/run.invoker"` ile, `--allow-unauthenticated` deploy flag'iyle, console, YAML, ya da Terraform ile.

**Erişimi kontrol etmek:**

| Kapsam | Komut |
| --- | --- |
| Bireysel servis/job | `gcloud run services add-iam-policy-binding` / `gcloud run jobs remove-iam-policy-binding` |
| Projedeki tüm servis/job'lar | `gcloud projects add-iam-policy-binding` (project-level IAM) |

**Network ingress ayarları** (IAM'den bağımsız, birlikte katmanlanabilir):

| Ayar | İzin verilenler |
| --- | --- |
| **All** (varsayılan, en az kısıtlayıcı) | Doğrudan internetten dahil, tüm istekler |
| **Internal** (en kısıtlayıcı) | Yalnızca internal HTTP(S) load balancer, servisi içeren bir VPC Service Controls perimeter'ının izin verdiği kaynaklar, aynı proje/perimeter'daki VPC network'leri, ve aynı proje/perimeter'daki Cloud Tasks/Eventarc/Pub/Sub/Workflows — internet istekleri servise hiç ulaşamaz |
| **Internal and Cloud Load Balancing** | Internal'ın izin verdiği her şey, artı external HTTP(S) load balancer (yine de doğrudan internetten değil) |

**Serverless VPC Access:** bir servisi/job'ı, internal DNS/IP kullanarak doğrudan bir VPC network'üne bağlar (VM'lere, Memorystore'a vb. internal IP ile erişmek için) — trafik hiç internete çıkmaz. Kurulum: API'yi etkinleştir → bir connector oluştur (region, servis/job'ın region'ıyla eşleşmeli; ayrılmış, kullanılmayan bir `/28` subnet ya da CIDR kullanır) → bir VPC network'e ve region'a bağla → servisi/job'ı onu kullanacak şekilde yapılandır (`--vpc-connector`); firewall kurallarıyla daha da kısıtla. Internal servisler tüm egress'i connector üzerinden yönlendirmelidir.

---

# Modül Özeti

Daha önceki Cloud Run genel bakışının üzerine, bu modül perde arkasında gerçekte ne olduğunu açıklıyor: kaynak modeli (service → revision → container instance; job → task → execution), starting'den stopped'a tam container yaşam döngüsü, autoscaling'in iç işleyişi (load balancer, scale to zero, cold start'lar, min/maks instance, concurrency), ve hem kimlik (IAM) hem network (ingress, VPC) katmanında access control.

---

# Anahtar Noktalar

- Bir service, sabit, kalıcı bir kaynaktır; bir revision, her deploy'da oluşturulan immutable bir anlık görüntüdür.
- Bir job'ın execution'ı, yalnızca paralel task'larının **her biri** başarılıysa başarılıdır.
- Container image'ları, her container başlangıcında internal storage'dan pull edilir — sadece deploy anında dokunulan Artifact Registry'den değil.
- Idle container'lar ücretsizdir ama CPU'ları neredeyse sıfıra throttle edilir; oradaki arka plan işi güvenilmezdir — Cloud Tasks kullan.
- SIGTERM, temizlik için 10 saniye verir; bir handler olmadan, container anında durur.
- Normal koşullarda Cloud Run, bir isteğin ortasında bir container'ı asla durdurmaz — yalnızca bir crash ya da memory limit'inin (varsayılan 512 MiB) aşılması durdurur.
- Scale to zero para tasarrufu sağlar ama cold start riski taşır; minimum instances cold start'lardan kaçınır ama boşta geçen süre için faturalandırır.
- Varsayılan maksimum instance 100'dür (senin config'in); platform kota sınırı 1.000'dir (talep edilebilir) — farklı şeylerdir.
- Varsayılan concurrency 80, maksimum 1.000'dir — CPU/bellek ağırlıklı istekler ya da paylaşılamayan global state için 1'e düşür.
- Her Cloud Run aksiyonu bir API çağrısıdır; IAM bunu, kaynaklara bağlı policy'ler (member + role binding'leri) aracılığıyla yetkilendirir.
- Bir servisi public yapmak, `allUsers`'a `roles/run.invoker` vermek demektir.
- IAM ve network ingress ayarları, birleştirilebilen bağımsız katmanlardır.
- Serverless VPC Access, Cloud Run trafiğini internal DNS/IP üzerinden internal VPC kaynaklarına yönlendirir, hiçbir zaman public internete dokunmaz.
