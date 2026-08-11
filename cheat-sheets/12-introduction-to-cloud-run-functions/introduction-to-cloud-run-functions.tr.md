# Modül 12 – Introduction to Cloud Run Functions

> Bu, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 1. Modülü'dür. Bu, modül 09–11'de işlenen "Service Orchestration and Choreography on Google Cloud" kursundan bağımsız, yeni bir kurstur.

---

# 🎯 Genel Bakış

Cloud Run functions, Google Cloud'un tamamen yönetilen (fully managed), serverless functions-as-a-service (FaaS) platformudur: tek amaçlı (single-purpose) function kodu yazarsın, deploy edersin, ve Cloud Run functions provisioning'i, scaling'i ve çalıştırmayı senin yerine halleder — yönetecek bir server yoktur.

```text
Function kodunu geliştir → Deploy et (console / gcloud / Cloud Build / Cloud Code) → Bir trigger kur (HTTP ya da event)
```

İki versiyonu vardır:

| | Cloud Run Functions (2nd gen) | Cloud Run Functions (1st gen) |
| --- | --- | --- |
| Eski adı | Cloud Functions (2nd generation) | Cloud Functions (1st generation) |
| Nasıl deploy edilir | Arka planda bir Cloud Run service olarak | Orijinal functions runtime'ı |
| Nasıl tetiklenir | Eventarc ve Pub/Sub | Daha sınırlı bir event trigger kümesi |
| Yapılandırılabilirlik | Tam — güncel, önerilen yol budur | Sınırlı |

Cloud Run'ın kendisi, Google'ın altyapısında container çalıştırmak için yönetilen bir compute platformudur; Cloud Run functions bunun üzerine inşa edilir — bir function, tek bir işlevsellik parçasına hizmet eden basit koddur, cloud altyapısından ya da başka servislerden gelen bir event tarafından tetiklenir (örn. Cloud Storage'a bir dosya yüklemesi, gelen bir Pub/Sub mesajı).

---

# Özellikler ve Faydalar

**Özellikler:**

- Deploy etmeden önce local geliştirme ve test.
- Service account'lar aracılığıyla diğer Google Cloud servislerine sorunsuz kimlik doğrulama.
- HTTP ve event-driven execution.
- Cloud SQL, Bigtable, Spanner ve Firestore ile yerleşik entegrasyon.
- Portability — bir function, kendi dili için standart herhangi bir runtime ortamında çalışır.

**Faydalar:**

- Mevcut cloud servislerini programlama mantığıyla genişletir ve zenginleştirir.
- Serverless — yazılım ve altyapı tamamen Google tarafından yönetilir; yama yapılacak server ya da güncellenecek framework yoktur.
- Autoscaling — kaynaklar event'lere yanıt olarak otomatik olarak provision edilir, günde birkaç invocation'dan milyonlarcasına kadar, ekstra bir çaba gerektirmeden.
- Observable — diagnosis için Cloud Logging ve Cloud Monitoring ile entegredir.
- Invocation sayısına, compute time'a ve outbound network veri transferine dayalı pay-as-you-go fiyatlandırma.

**Altta yatan kapasiteler:**

| Kapasite | Limit / davranış |
| --- | --- |
| Instance başına bellek / CPU | Instance başına 32 GB'a kadar RAM, 4 vCPU'ya kadar |
| Concurrency | Instance başına 1000'e kadar eşzamanlı istek (cold start'ları azaltır, scaling latency'sini iyileştirir) |
| Timeout | HTTP function'lar için 60 dakikaya, event-driven function'lar için 10 dakikaya kadar |
| Revision'lar | Her deploy'da yeni bir revision yaratılır — traffic splitting ve rollback'i destekler |
| Portability | Cloud Run'ın container platformu üzerine kurulu olduğu için Cloud Run'a ya da Kubernetes'e taşınabilir |

> **Sınav tuzağı:** Cloud Run functions, kendine ait bir execution modeline sahip ayrı bir ürün değildir — altta gerçekten Cloud Run'dır (2nd gen), sadece üzerine function-odaklı bir deployment ve triggering modeli eklenmiştir. Revision/rollback/traffic-splitting davranışını miras almasının ve düz Cloud Run'a ya da Kubernetes'e migrate edilebilmesinin sebebi tam olarak budur.

---

# HTTP Functions vs. Event-Driven Functions

Cloud Run functions iki türde gelir:

| | HTTP functions | Event-driven functions |
| --- | --- | --- |
| Trigger | Bir HTTP(S) isteği | Cloud ortamından gelen bir event (örn. yeni bir Pub/Sub mesajı, Cloud Storage'dan silinen bir dosya) |
| URL atanır mı? | Evet — function, o URL'de istekleri alır | Hayır |
| Varsayılan kimlik doğrulama | Gereklidir (unauthenticated'a opt-in edilebilir) | N/A — event trigger source'u tarafından belirlenir |
| Tipik kullanım | Webhook'lar, başka servislerden gelen istekleri işleyen API'ler | Altyapı değişikliklerine otomatik olarak tepki vermek |
| Implementasyon | Diliniz için Functions Framework'e register edilmiş bir HTTP handler yazılır; istek işlenir, bir response gönderilir; herhangi bir background task (thread, promise) response gönderilmeden önce tamamlanmalıdır | Bir **CloudEvent function** ya da bir **Background function** olarak implemente edilir |

Event-driven function'lar **event trigger**'lar kullanır; Pub/Sub, Cloud Storage, Firestore, Firebase ve Eventarc kaynakları için desteklenir — ve Eventarc üzerinden, Pub/Sub'ı event bus olarak destekleyen herhangi bir Google Cloud servisi için.

| | CloudEvent functions | Background functions |
| --- | --- | --- |
| Temel | CloudEvents endüstri standardı spesifikasyonu | Daha eski tarz, event verisini event türüne göre alır |
| Kayıtlı olduğu yer | Functions Framework (kullanıcı function'larını kalıcı bir HTTP uygulamasında saran açık kaynaklı bir kütüphane) | — |
| Desteklendiği yer | Cloud Run functions, tüm dil runtime'ları; Cloud Run functions 1st gen'de .NET, Ruby, PHP ile | Yalnızca Cloud Run functions 1st gen, Node.js, Python, Go, Java ile |

> **Sınav tuzağı:** Background functions, bugün serbestçe seçebileceğin bir alternatif stil değildir — bunlar, belirli birkaç runtime'da Cloud Run functions 1st gen'e özgü, **daha eski** event-driven implementasyondur. CloudEvent functions (Functions Framework ve CloudEvents standardı üzerine kurulu) ise güncel, tam donanımlı Cloud Run functions'ın (2nd gen) tüm desteklenen dillerde kullandığı şeydir.

---

# Kullanım Senaryoları (Use Cases)

| Kullanım senaryosu | Nasıl çalışır |
| --- | --- |
| Data processing | Dosyalar/objeler üzerindeki Cloud Storage event'lerine tepki ver: doğrula, dönüştür, downstream kullanım için görsel/video işle |
| Webhook'lar ve API'ler | HTTP function'lar, dış sistemlerden gelen webhook/API çağrılarını işler |
| Mobile backend | Firebase ve Firestore tarafından tetiklenen event'lere tepki ver |
| IoT | Pub/Sub'a stream edilen device verisini işle ve sakla |

---

# Dil Runtime'ları ve Kaynak Kodu Yapısı

Her dil runtime'ının, Cloud Run functions'ın function'ının source kodunu nerede arayacağına dair kendi kuralları vardır:

| Dil | Varsayılan giriş dosyası | Dependency/config dosyası |
| --- | --- | --- |
| Node.js | Kökte `index.js` (`package.json`'daki `main` alanıyla override edilebilir) | `package.json` — Node.js için Functions Framework'ü dependency olarak içerir |
| Python | Kökte `main.py` | `requirements.txt` — Python için Functions Framework'ü içerir |
| Go | Proje kökünde bir Go package | `go.mod` — Go için Functions Framework'ü içerir |

**Function entry point**, function invoke edildiğinde çalıştırılan function'dır (ya da dile bağlı olarak class'tır); main dosyanızda ya da root package'ınızda tanımlanmış olmalı, ve deploy anında açıkça belirtilir.

**Region seçimi:** temel değerlendirmeler latency ve availability'dir. Genellikle kullanıcılarınıza en yakın region'ı seçebilirsiniz, ama uygulamanızın bağımlı olduğu diğer Google Cloud servislerinin konumunu da tartmanız gerekir — birden fazla lokasyona yayılmış servisler hem latency'yi hem fiyatlandırmayı etkiler. Cloud Run functions ile Cloud Run functions 1st gen'in farklı bölgesel kullanılabilirlikleri vardır.

---

# Build Etme ve Deploy Etme

**IAM gereksinimleri:** deploy etmek için bir kullanıcının **Cloud Functions Developer** IAM rolüne (ya da eşdeğerine), artı Cloud Run functions runtime service account'u üzerinde **Service Account User** IAM rolüne ihtiyacı vardır.

**Deployment yöntemleri:** Google Cloud console, Cloud Build, Cloud Code (function'ları doğrudan IDE'nizden yaratmanıza, deploy etmenize ve invoke etmenize izin verir), ya da gcloud CLI.

Önemli `gcloud functions deploy` flag'leri:

| Flag | Amaç |
| --- | --- |
| `--gen2` | Cloud Run functions'a (2nd gen) deploy et |
| `--region` | Deploy edilecek region |
| `--runtime` | Dil runtime'ı |
| `--source` | Source kodunun konumu |
| `--entry-point` | Entry point olan function/class adı |
| Trigger flag'leri | Trigger'ın türü ve yapılandırması |

**Source konumu seçenekleri (`--source`):**

| Kaynak | Detaylar |
| --- | --- |
| Local machine | Source kökü için bir local dosya sistemi yolu; deployment'ın bir parçası olarak Cloud Storage'a upload etmek için opsiyonel olarak `--stage-bucket` kullanılabilir; `.gcloudignore` ile dosyalar hariç tutulabilir |
| Cloud Storage | Source'u zip olarak paketlenmiş şekilde içeren bir bucket'a giden yol, source dosyaları zip'in kökünde; deploy eden hesap (1st gen) ya da Cloud Run functions service agent'ı (2nd gen) bucket'a okuma erişimine sahip olmalıdır |
| Source repository | Bir revision'a (`revisions/<name>`) ve opsiyonel olarak bir alt dizine (`paths/<source_directory_path>`) referans; service agent'ın **Source Repository Reader** (`roles/source.reader`) rolüne ihtiyacı vardır; ayrıca bağlı bir GitHub ya da Bitbucket repository'sinden deploy etmeyi de mümkün kılar |

Google Cloud console'un inline editor'ünden de doğrudan yazıp deploy edebilirsiniz (solda dosya listesi, sağda editor).

---

# Build Pipeline'ı

```text
Source kodunuz → Cloud Storage → Cloud Build → container image → Artifact Registry → Cloud Run functions çalıştırır
```

- Deploy edilen source, bir Cloud Storage bucket'ında saklanır.
- **Cloud Build**, source'u otomatik olarak bir container image'a build eder ve **Artifact Registry**'ye push eder — bu tamamen otomatiktir ve sizden doğrudan bir girdi gerektirmez.
- Cloud Run functions, function'ınızı çalıştırması gerektiğinde bu image'a erişir.
- Cloud Build API'sinin projenizde etkinleştirilmiş olması gerekir; build kaynakları kendi projenizde çalışır, ve tüm build logları Cloud Logging üzerinden erişilebilirdir.
- **Artifact Registry**, ortaya çıkan container image'ları (ve diğer yazılım artifact'lerini) private repository'lerde saklar ve yönetir; ürettiği paketleri saklamak için Cloud Build ile entegre olur.

---

# Modül Özeti

Cloud Run functions, Cloud Run'ın üzerine kurulu, Google Cloud'un tamamen yönetilen FaaS platformudur: tek amaçlı function'lar yazarsınız, bunları deploy edersiniz (console, gcloud, Cloud Build ya da Cloud Code), ve bir trigger eklersiniz — ya HTTP (bir URL ile, varsayılan olarak kimlik doğrulamalı) ya da event-driven (güncel 2nd-gen platformda tüm dillerde CloudEvent functions, ya da 1st gen'de birkaç runtime için daha eski Background functions). Function'lar sıfıra yakından milyonlarca invocation'a kadar otomatik olarak scale olur, Cloud Logging/Monitoring ile entegre olur, ve invocation sayısı, compute time ve egress üzerinden faturalandırılır. Deployment, giriş noktası ne olursa olsun (console, CLI ya da source repo) her zaman aynı pipeline'ı takip eder: source'unuz Cloud Storage'a düşer, Cloud Build onu bir container image'a dönüştürür, o image Artifact Registry'ye push edilir, ve Cloud Run functions oradan çalıştırır — bu, gerektiğinde bir function'ın doğrudan düz Cloud Run'a ya da Kubernetes'e migrate edilebilmesinin de sebebidir.

---

# Önemli Noktalar

- Cloud Run functions (2nd gen, eski adıyla Cloud Functions 2nd gen), function'ları Cloud Run service'leri olarak deploy eder, Eventarc/Pub/Sub üzerinden tetiklenir; Cloud Run functions 1st gen, daha eski, daha sınırlı versiyondur.
- HTTP function'lar bir URL alır ve varsayılan olarak auth gerektirir; event-driven function'lar Pub/Sub, Cloud Storage, Firestore, Firebase ya da Eventarc'tan (90+ kaynak) gelen trigger'lara tepki verir.
- CloudEvent functions (Functions Framework, CloudEvents standardı, 2nd gen'de tüm diller) güncel event-driven implementasyondur; Background functions yalnızca 1st gen'e özgü daha eski stildir.
- Source kodu konumu kuralları runtime'a özgüdür: `index.js` (Node.js), `main.py` (Python), package kökü (Go) — her biri Functions Framework'ü de içeri çeken kendi dependency manifest'iyle birlikte.
- Deploy etmek Cloud Functions Developer rolünü, artı runtime service account'u üzerinde Service Account User rolünü gerektirir; source bir local yoldan, bir Cloud Storage zip'inden ya da bir source repository'den (bağlı GitHub/Bitbucket dahil) gelebilir.
- Her deployment aynı build pipeline'ını takip eder: source → Cloud Storage → Cloud Build → container image → Artifact Registry → çalıştırma.
