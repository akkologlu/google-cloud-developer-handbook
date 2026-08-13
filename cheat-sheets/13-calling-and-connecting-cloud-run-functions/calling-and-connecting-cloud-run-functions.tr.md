# Modül 13 – Calling and Connecting Cloud Run Functions

> Bu, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 2. Modülü'dür, Modül 1'i (Modül 12: Introduction to Cloud Run Functions) takip eder.

---

# Genel Bakış

Bu modül, Cloud Run functions'ı çağırmak ve mimarinizin geri kalanına bağlamak için üç ilişkili mekanizmayı ele alır:

```text
Trigger'lar (bir function nasıl invoke edilir) → Workflows (function'lar/servisler nasıl zincirlenir) → Serverless VPC Access (bir function bir VPC network'e nasıl erişir)
```

---

# Function Trigger'ları

Bir Cloud Run function'ını bir senaryoya tepki olarak çalışacak şekilde ayarlamak için deployment anında bir **trigger** belirlersiniz.

| Trigger kategorisi | Neye tepki verir | Neye karşılık gelir |
| --- | --- | --- |
| HTTP trigger | HTTP(S) istekleri | HTTP functions |
| Event trigger | Google Cloud projenizdeki event'ler | Event-driven functions |

- Aynı trigger source ayarlarıyla birden fazla function deploy ederek, aynı event'in hepsini tetiklemesini sağlayabilirsiniz.
- Ancak tek bir function, aynı anda **birden fazla trigger'a bağlanamaz**.
- Event-driven function'lar, filtrelerle (service name, method name, event type ve diğer kriterler) oluşturulan **Eventarc event trigger'larını** kullanır — Google Cloud console'da ya da gcloud CLI ile yapılandırılabilir.

> **Sınav tuzağı:** "Bir event birden fazla function'a fan-out olabilir" ile "bir function'ın yalnızca tek bir trigger'ı olabilir" birbiriyle çelişmez — fan-out, aynı trigger source'u paylaşan *birden fazla function* deploy ederek gerçekleşir, tek bir function'a birden fazla trigger bağlayarak değil.

**Tüm event-driven Cloud Run functions, event delivery için Eventarc kullanır.** Eventarc, Cloud Audit Logs, harici SaaS event kaynakları ve Pub/Sub'a publish edilen custom kaynaklar dahil 90'dan fazla Google Cloud kaynağını destekler — bu, Cloud Run functions'ın, Pub/Sub'ı event bus olarak kullanan herhangi bir Google servisiyle (örn. Cloud Logging, Cloud Scheduler, Gmail Push Notifications) entegre olabilmesinin de sebebidir.

---

# Trigger Türleri

| Trigger türü | Ne zaman tetiklenir | Gereksinimler |
| --- | --- | --- |
| HTTP trigger | Function'a atanmış URL'ye bir HTTP(S) isteği gelir; GET, POST, PUT, DELETE, OPTIONS destekler | HTTP function olarak deploy edilmek dışında bir şey gerekmez |
| Pub/Sub trigger | Belirtilen bir Pub/Sub topic'ine bir mesaj publish edilir | Function event-driven olmalı; bir Eventarc trigger olarak implemente edilir |
| Cloud Storage trigger | Belirtilen bir bucket'taki bir obje üzerinde seçilen bir event türü gerçekleşir | Function event-driven olmalı; bir Eventarc trigger olarak implemente edilir |
| Firestore trigger | Belirtilen bir document path'indeki bir document üzerinde seçilen bir event türü (create, update, delete, write) gerçekleşir | Firestore, function ile aynı Google Cloud projesinde olmalı; yalnızca document seviyesinde geçerlidir, hiçbir zaman bir field ya da collection seviyesinde değil |
| Firebase trigger'ları | Google Analytics for Firebase (yalnızca 1st gen), Firebase Realtime Database, Firebase Authentication (yalnızca 1st gen), Firebase Remote Config'ten gelen event'ler | Firebase servisi, function ile aynı Google Cloud projesinde olmalı |

**Event payload formatı, function'ın implementasyon stiline bağlıdır:**

| Implementasyon stili | Pub/Sub event verisi nasıl gelir | Cloud Storage event verisi nasıl gelir |
| --- | --- | --- |
| CloudEvent function | CloudEvents formatında | CloudEvents formatında |
| Background function | `PubsubMessage` formatında | `StorageObjectData` formatında |

> **Sınav tuzağı:** Bir Firestore trigger, field ya da collection bazında değil, document bazında tetiklenir. "Belirli bir field için trigger oluştur" Firestore trigger'larının ifade edebileceği bir şey değildir — en ince granülerlik document'tır.

---

# Workflows

**Workflows**, servisleri tanımladığınız bir sırayla çalıştıran, tamamen yönetilen (fully managed), serverless bir orchestration platformudur — **service orchestration** pattern'inin somut implementasyonudur ve merkezi orchestrator olarak görev yapar.

| Özellik | Detay |
| --- | --- |
| Neyi bağlar | Cloud Run functions ile inşa edilmiş HTTP servisleri, harici API'ler ve Cloud Run gibi diğer Cloud servisleri |
| State | Bir yıla kadar state tutabilir, retry yapabilir, poll edebilir ya da bekleyebilir — uzun süren business process'leri mümkün kılar |
| Observability | Her execution loglanır ve gözlemlenebilir, application flow için merkezi bir source of truth sağlar |
| Tanım formatı | YAML ya da JSON — bir dizi step |

**Build süreci:**

1. Gerekli API'leri etkinleştirin (Cloud Run functions, Cloud Run, Workflows ve kullanılan diğer servisler); gerekli service account'ları oluşturun.
2. Function'ları **HTTP function** olarak, HTTP trigger'larla yazın ve deploy edin, böylece her biri Workflows'un çağırabileceği bir URL endpoint'i alır.
3. Function'ları tek tek test edin — curl ya da başka bir HTTP client ile, ve best practice olarak deployment'tan önce local olarak.
4. Function'ları bağlayan workflow'u, Workflows syntax'ı (YAML/JSON) kullanarak oluşturun.
5. Workflow'u deploy edin ve execute edin.

**Bir workflow tanımının içinde:**

- Step'ler, Cloud Run functions'ı bir HTTP isteği aracılığıyla invoke eder (örn. bir `GET` step'i, ardından bir `POST` step'i), function'ın URL'i argüman olarak sağlanır.
- Bir step'in sonucu bir sonraki step'e input olarak geçirilebilir — function output'larını zincirleme.
- Bir workflow ayrıca harici bir **REST API**'yi de çağırabilir (önceki sonuçları query parametresi olarak geçirerek) ve bir **Cloud Run service**'e bağlanabilir — Cloud Run service'in sonucu, genel workflow'un sonucu haline gelebilir.

> **Sınav tuzağı:** Bir workflow'dan invoke edilen function'lar **HTTP function** olmalıdır — Workflows onları HTTP(S) üzerinden invoke eder, bu yüzden HTTP trigger'ı olmayan, yalnızca event-driven bir function bir workflow step'ine aynı şekilde bağlanamaz.

---

# Serverless VPC Access

Bir **Virtual Private Cloud (VPC) network**, Google'ın production network'ü içinde uygulanan, fiziksel bir network'ün sanal versiyonudur — global bir wide area network ile birbirine bağlanmış regional subnet'lerden oluşan global bir kaynaktır.

**Serverless VPC Access**, Cloud Run functions'ı doğrudan VPC network'ünüze bağlamanıza izin verir, böylece function yalnızca **internal IP address**'i olan kaynaklara erişebilir:

- Compute Engine VM instance'ları
- Memorystore
- Internal IP'si olan diğer kaynaklar

Trafik, **internal DNS ve internal IP address**'ler üzerinden akar — hiçbir zaman genel internete açık hale gelmez.

**Kurulum adımları:**

| Adım | Detay |
| --- | --- |
| 1. API'yi etkinleştir | Serverless VPC Access API'sini etkinleştirin |
| 2. Bir connector oluştur | Bir Serverless VPC Access connector'u, serverless ortam ile VPC network arasındaki trafiği yönetir |
| 3. Connector'ı bağla | Belirli bir VPC network'e ve region'a bağlayın — connector'ın region'ı, function'ın deploy edildiği region ile **eşleşmelidir** |
| 4. Bir subnet ayır | Connector'ın münhasır kullanımı için bir subnet ya da CIDR range yapılandırılmalıdır |
| 5. Function'ı yapılandır | Connector'ı kullanması gereken her function, connector'ı kullanacak şekilde ayrı ayrı yapılandırılmalıdır (console ya da gcloud) |

Bir connector'ın erişimini **firewall rule**'larıyla daha da kısıtlayabilirsiniz, ve connector'lar bir **Shared VPC** network'ündeki kaynaklara da erişebilir.

> **Sınav tuzağı:** Bir Serverless VPC Access connector'ının region'ı, function'ın deployment region'ıyla eşleşmelidir — uyuşmayan region'lar, "function VPC kaynağıma erişemiyor" hatalarının yaygın bir sebebidir, bir firewall ya da IAM problemi değil.

---

# Modül Özeti

Cloud Run functions, trigger'lar aracılığıyla invoke edilir — doğrudan HTTP(S) çağrıları için HTTP trigger'lar, Pub/Sub, Cloud Storage, Firestore ve Firebase event'leri için (hepsi Eventarc aracılığıyla teslim edilen) event trigger'lar; bir function tam olarak bir trigger'a bağlanır, ancak birçok function aynı trigger source'unu paylaşabilir. Function'ları ve diğer servisleri stateful, gözlemlenebilir bir sürece zincirlemek için Workflows, bir YAML/JSON step tanımı aracılığıyla HTTP function'ları, harici REST API'leri ve Cloud Run service'lerini orchestrate eder, bir step'in sonucunu bir sonrakine geçirir. Yalnızca internal IP address'i olan kaynaklara — VM instance'ları, Memorystore ve daha fazlası — erişmek için bir function, Serverless VPC Access kullanır: bir subnet/CIDR range'e ayrılmış, function ile eşleşen bir VPC network ve region'a bağlanmış, ve function başına açıkça etkinleştirilmiş bir connector, tüm trafiği genel internetten uzak tutarak.

---

# Önemli Noktalar

- Trigger'lar iki kategoride gelir — HTTP (HTTP functions) ve event (event-driven functions) — ve bir function yalnızca bir trigger'a bağlanabilir, ancak birden fazla function aynı trigger source'unu paylaşabilir.
- Tüm event-driven function'lar, delivery için Eventarc kullanır; Eventarc, Pub/Sub'a publish edilen custom kaynaklar dahil 90'dan fazla kaynağı destekler.
- Pub/Sub, Cloud Storage, Firestore ve Firebase trigger'larının her biri event-driven bir function gerektirir; Firestore trigger'ları kesinlikle document seviyesinde uygulanır.
- CloudEvent functions, Pub/Sub/Cloud Storage verisini CloudEvents formatında alır; Background functions ise bunun yerine `PubsubMessage`/`StorageObjectData` formatlarını alır.
- Workflows, bir YAML/JSON step tanımı aracılığıyla HTTP function'ları, harici REST API'leri ve Cloud Run service'lerini zincirleyen, bir yıla kadar state tutan ve retry yapan, tamamen yönetilen bir orchestration platformudur.
- Bir workflow'dan invoke edilen function'lar, Workflows'un onları HTTP(S) üzerinden çağırabilmesi için HTTP function olarak deploy edilmelidir.
- Serverless VPC Access, function'ları VM instance'larına, Memorystore'a ve diğer internal-IP kaynaklarına internal DNS/IP üzerinden bağlar, genel internete hiçbir açıklık olmadan.
- Bir Serverless VPC Access connector'ının kendine ait, ayrılmış bir subnet/CIDR range'e ihtiyacı vardır, function'ın region'ıyla eşleşen bir region'a bağlanmalıdır ve function başına açıkça yapılandırılmalıdır; firewall rule'ları ve Shared VPC de desteklenir.
