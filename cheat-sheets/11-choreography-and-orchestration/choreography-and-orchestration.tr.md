# Modül 11 – Choreography and Orchestration

> Bu, "Service Orchestration and Choreography on Google Cloud" kursunun 3. Modülü'dür (1. Modül: [Microservices'e Giriş](../09-introduction-to-microservices/introduction-to-microservices.tr.md), 2. Modül: [Event-Driven Applications](../10-event-driven-applications/event-driven-applications.tr.md)). Bu, kursta somut Google Cloud ürünlerinden ilk kez bahseden modüldür: Pub/Sub, Eventarc, Workflows, Cloud Tasks ve Cloud Scheduler — birlikte, Google Cloud'un "Application Integration" araç kutusunu oluştururlar.

---

# 🎯 Genel Bakış

Microservices arasındaki iletişimi koordine etmek, bir microservices mimarisinin en zor kısımlarından biridir — bir zamanlar bir monolith'in içinde yaşayan karmaşıklığın bir kısmı, servislerin birbiriyle nasıl konuştuğuna kayar.

İki temel koordinasyon deseni vardır: **service choreography** ve **service orchestration**.

```text
Choreography (dans)                  Orchestration (orkestra)

Service A ──event──┐                          ┌──→ Service A
Service B ──event──┼─→ (şef yok)               Orchestrator ─┼──→ Service B
Service C ──event──┘                          └──→ Service C
```

---

# Service Choreography vs. Service Orchestration

| | Service Choreography | Service Orchestration |
| --- | --- | --- |
| Analoji | Koreografi yapılmış bir dans — her dansçı kendi bölümünü bilir ve bağımsız olarak icra eder | Bir orkestra — bir şef (conductor), her müzisyeni aktif olarak senkronize eder |
| Kontrol | Dağıtık (distributed) — her servis event'lere kendi başına tepki verir | Merkezi (centralized) — tek bir orchestrator tüm etkileşimleri kontrol eder |
| Coupling | Gevşek bağlı; servisler ayrı ayrı yaratılır, değiştirilir, ölçeklenir | Yine gevşek bağlı; servislerin birbirini bilmesine gerek yoktur |
| Source of truth | Yok — business logic servisler arasında dağılmıştır | Merkezi — orchestrator/workflow süreci tanımlar |
| Ana gücü | Takımlar/organizasyonlar arası decentralized control; GCP servis event'lerinden faydalanmak kolay | Sürecin üst düzey (high-level) görünümü; troubleshooting ve tracking daha kolay |
| Ana zayıflığı | Merkezi bir source of truth yok — genel akışı anlamak daha zor | Tek arıza noktası (single point of failure) — orchestrator çökerse hiçbir şey çalışmaz |

> Her iki desen de gevşek bağlı, bağımsız deploy edilebilir servisler üretir. Asıl fark, **süreç mantığının nerede yaşadığıdır**: servisler arasında dağılmış (choreography) ya da tek bir yerde merkezileşmiş (orchestration).

---

# Pub/Sub

Pub/Sub, bağımsız servisler ya da uygulamalar arasında mesaj göndermek ve almak için kullanılan, tamamen yönetilen (fully managed), gerçek zamanlı bir messaging servisidir.

```text
Publisher → Topic → [Subscriber 1'in kuyruğu, Subscriber 2'nin kuyruğu, ...] → Subscriber'lar
```

- Bir **publisher**, bir **topic**'e bir mesaj gönderir. Mesaj Pub/Sub içinde saklanır ve o topic'in her subscriber'ının kendi kuyruğuna teslim edilir.
- Bir **pull subscription**'da, subscriber yeni mesajlar için ara sıra poll eder.
- Bir **push subscription**'da, Pub/Sub mesajı otomatik olarak yapılandırılmış bir endpoint'e gönderir.
- Bir subscriber, mesajı kuyruğundan kaldırmak için onu **acknowledge** eder (onaylar). Mesaj yalnızca işlendikten **sonra** kaldırıldığı için, Pub/Sub **at-least-once delivery** garanti eder.

**Örnek — image resizing:** Bir Cloud Storage bucket'ı, yeni bir dosya geldiğinde bir "image uploads" topic'ine publish edecek şekilde yapılandırılır. İki subscriber tepki verir: bir Resizing Service (resmi yeniden boyutlandırır, başka bir bucket'a yazar, Firestore'u günceller) ve bir Upload Confirm uygulaması (başarılı upload'ı kaydetmek için Firestore'u günceller). Her Cloud Run servisi basit kalır — işlemeyi başlatan şey Pub/Sub'dır.

---

# Eventarc

Eventarc, event-driven uygulamalar inşa etmek için Google Cloud'un tamamen yönetilen eventing sistemidir.

- Birçok GCP ürünü, event'leri Eventarc'a **doğrudan** gönderebilir; doğrudan desteği olmayanlar ise **Cloud Audit Logs** kayıtları üzerinden yakalanır — log parse etme kodu yazmana gerek kalmaz.
- Third-party sağlayıcılar, Eventarc API'si üzerinden event yaratabilir. Pub/Sub topic'leri de bir event source olarak kullanılabilir.
- Bir **event trigger**, belirli bir event türünü belirli bir source'tan belirli bir target'a yönlendiren, kural tabanlı (rule-based) bir filtredir.
- Eventarc, transport katmanı olarak **Pub/Sub'ı kullanır** (güvenilirlik ve observability için) ve altındaki topic'leri/subscription'ları otomatik olarak yönetir — uygulaman yalnızca Eventarc'ın gönderdiği HTTP isteklerini kabul etmesi gerekir. Pub/Sub'a hiç doğrudan dokunmazsın.
- Event'ler, kaynak ne olursa olsun standart CNCF **CloudEvents** formatında teslim edilir — Python, JavaScript, Java, Go, C#, Ruby ve PHP'de SDK'ları olan ortak bir metadata formatı, bu sayede event-handling kodun, event'in nereden geldiğine bağlı olarak değişmez.

| | Pub/Sub (doğrudan) | Eventarc |
| --- | --- | --- |
| Event source'ları | Publisher'ları kendin bağlarsın | Birçok yerleşik GCP + third-party source, özel ingestion kodu yok |
| Topic/subscription yönetimi | Sen yönetirsin | Senin için otomatik olarak yaratılır ve yönetilir |
| Event formatı | Publisher ne seçtiyse | Standardize edilmiş CloudEvents formatı, SDK'larla birlikte |
| Arayüz | Ingestion kodu yaz | Basit, kural tabanlı trigger: source, filter, destination |

> **Sınav tuzağı:** Pub/Sub ve Eventarc rakip değildir. Eventarc, Pub/Sub'ın **üzerine** kurulmuş bir abstraction layer'dır — kendi ingestion kodunu ve topic yönetimini elle yazmadan, standardize edilmiş bir event formatı ve birçok event source'una yerleşik destek istediğinde Eventarc'ı kullan.

---

# Workflows

Workflows, Google Cloud'un tamamen yönetilen orchestration platformudur — service-orchestration deseninin somut implementasyonu.

- GCP servislerini ve API çağrılarını **stateful, otomatikleştirilmiş süreçlere** orchestrate eden workflow'lar tasarlar ve deploy edersin.
- Bir workflow, uygulamanın akışı için **merkezi bir source of truth**'tur.
- Her execution loglanır ve observable'dır, bu da troubleshooting'i kolaylaştırır.
- Bir workflow, state tutabilir, retry yapabilir, poll edebilir ya da **bir yıla kadar** bekleyebilir — bu da gerçekten uzun süren business process'lerin oluşturulmasını mümkün kılar.
- Workflows, API'ların fırlattığı retry ve exception'ları yönetir, genel güvenilirliği artırır.

**Örnek — sipariş işleme:** Firestore inventory'sini kontrol et → mevcut ürünleri kilitle → bir "out of stock" flag'ine göre dallan (bu flag akışın ilerisinde de kullanılır) → bir onay mesajı hazırla (Cloud Run function) ya da tedarikçilerden daha fazla envanter iste (Cloud Run service) → Firestore'u güncelle → müşteriye e-posta gönder → herhangi bir şey stokta yoksa opsiyonel olarak Slack'e mesaj at. Her execution, tek bir transaction'ı izlemek için loglanır.

Workflows; HTTP tabanlı microservices'i dayanıklı (durable), stateful akışlara zincirlemek için, ve robust error handling'in önemli olduğu batch/set-of-items işleme için güçlü bir seçimdir.

---

# Choreography mi, Orchestration mi Seçmeli?

**Choreography**'de, bir sonraki adımı receiving servis kontrol eder. Service A bir event ürettiğinde, başka herhangi bir servisin bu event üzerinde harekete geçip geçmeyeceği hakkında **hiçbir fikri yoktur**; bu A'nın sorumluluğunda değildir. Bir consumer (Service B), event'in formatını anlamalıdır ve onu Service A'nın gönderdiğini bilebilir, ama ona sıkı bağlı (tightly coupled) değildir. Eventarc kullanarak, çoğu GCP ya da custom servis producer olarak davranabilir.

Sipariş işleme örneği choreography ile de inşa **edilebilirdi**: her servis "sıradaki adım için hazırım" event'i üretir, ve stokta var/yok kararı o adımın sahibi olan servis tarafından verilir. Ama kurumsal bir sipariş sistemi visibility, error handling ve retry'lara ihtiyaç duyar — ve bunları saf event-based bir tasarımda doğru yapmak zordur. Onu nasıl troubleshoot edersin? Süreç, envanteri kilitlemek ile envanteri güncellemek arasında bir yerde aborta olursa ne olur? Bir tedarikçi isteğinin gerçekten başarılı olduğunu nasıl garanti edersin?

Orchestration bu soruları doğrudan cevaplar: her execution ayrı ayrı takip edilir, sipariş mantığı tek bir yerde yaşar, ve retry/error handling yerleşik gelir.

| Soru | Hangisini destekler |
| --- | --- |
| Karmaşık bir süreci merkezi olarak, güçlü visibility, retry ve error handling ile yönetmen mi gerekiyor? | Orchestration |
| Servisler, her biri kendi parçasını bağımsız yöneten farklı takımlara ya da organizasyonlara mı yayılıyor? | Choreography |
| Öncelikle GCP servisleri tarafından zaten üretilen event'lere mi tepki vermek istiyorsun? | Choreography |
| Troubleshooting için tek, izlenebilir bir execution kaydına mı ihtiyacın var? | Orchestration |

> **Sınav tuzağı:** Orchestration, troubleshoot etmesi daha kolay diye otomatik olarak "daha iyi" değildir — paylaşılan, merkezi bir orchestrator, paylaşılan, merkezi bir kontrol gerektirir; bu da bağımsız yönetilen takımlar/organizasyonlar arasında çöker. Choreography'nin decentralization'ı, çevresinden dolaşılması gereken bir kısıtlama değil, tam olarak buradaki avantajdır.

Pratikte ikisi bir arada kullanılır: örneğin, Order, Fulfillment ve Marketing servislerinin her biri kendi içinde **Workflows** ile implemente edilebilir (orchestration), **Eventarc** ise bu servisler **arasında** event trigger'ları taşır ve bir Cloud Storage bucket'ına yeni dosyalar düştüğünde tepki verir (sınırlar arasında choreography).

---

# Cloud Tasks vs. Pub/Sub

Cloud Tasks, çok sayıda dağıtık task'ın execution'ını, dispatch'ini ve teslimatını yönetir — her task bir HTTP servisine dispatch edilir.

- Queue'lar, yapılandırılabilir bir maksimum dispatch rate'i, maksimum eşzamanlı dispatch sayısını, maksimum retry denemesini ve retry'lar arası gecikmeyi destekler.
- Bir task, belirli bir gelecek zamanda dispatch edilmek üzere zamanlanabilir.
- Teslimat **at-least-once**'tur, duplicate task'lar elenir.
- Authenticate edilmiş çağrılar, yaratan uygulamanın service account'ına bağlı bir token'ı otomatik olarak ekleyebilir.

Cloud Tasks ve Pub/Sub kavramsal olarak benzerdir (ikisi de message passing / async integration yapar) ama farklı sorunları çözer:

| | Cloud Tasks | Pub/Sub |
| --- | --- | --- |
| Invocation | **Explicit (açık)** — creator, execution ve destination üzerinde tam kontrolü elinde tutar | **Implicit (örtük)** — bir mesaj publish etmek, mevcut olan hangi subscriber varsa onu tetikler |
| Destination | Creator'ın seçtiği belirli, bilinen bir endpoint | Publisher için bilinmiyor — o anki herhangi bir subscriber |
| En uygun olduğu durum | Belirli, bilinen bir servisi asenkron olarak çalıştırmak, isteğe bağlı olarak kendi programına göre (örn. yavaş bir background işi ana request yolundan offload etmek) | Receiving servislerin diğer servislerden gelen event'lere tepki verdiği event-based mimariler |

> **Sınav tuzağı:** Cloud Tasks ve Pub/Sub'ı, ikisi de asenkron messaging yapıyor diye birbirinin yerine geçebilir sanma. Belirleyici soru şudur: *gönderen, bunu tam olarak hangi servisin ve ne zaman ele alacağını kontrol etmek zorunda mı* (Cloud Tasks), *yoksa gönderen sadece bir şeyin olduğunu duyurmak mı istiyor, kimin dinlediğiyle ilgilenmeden* (Pub/Sub)?

---

# Cloud Scheduler

Cloud Scheduler, tek bir dashboard'dan yönetilen, tamamen yönetilen, kurumsal düzeyde bir cron job scheduler'dır.

- Job'lar standart **Unix cron formatını** kullanır: 5 boşlukla ayrılmış alan — minute, hour, day-of-month, month, day-of-week (0 = Pazar … 6 = Cumartesi).
- Bir alan; tek bir sayı, tire ile bir aralık (kapsayıcı/inclusive), tüm aralık için `*`, bir adım için `*/N`, ya da virgülle ayrılmış bir liste olabilir.
- Job'lar bir Pub/Sub topic'ini, bir App Engine uygulamasını, ya da herkese açık bir HTTP endpoint'ini hedef alabilir.
- Cloud Tasks gibi, Cloud Scheduler da authenticate edilmiş HTTP isteklerine bir service account token'ı ekleyebilir.
- Execution garantilidir, başarısız execution'lar retry edilir.

```text
Cron alanları:  minute   hour   day-of-month   month   day-of-week
Örnek:             15      0          *           *          *      → her gün 00:15
Örnek:              *      */2        *           *          *      → her 2 saatte bir
```

> **Sınav tuzağı:** Cloud Scheduler'ın varsayılan time zone'u UTC'dir, ve modül açıkça UTC'de kalmayı önerir — daylight saving time uygulayan time zone'lar, bir job'ın **atlanmasına** (saatler ileri alındığında) ya da **iki kez çalışmasına** (saatler geri alındığında) neden olabilir.

---

# Doğru Application Integration Servisini Seçmek

| İhtiyaç | Servis |
| --- | --- |
| Kendi kurduğun ve yönettiğin publish/subscribe messaging | Pub/Sub |
| Ingestion kodu yazmadan, standart bir formatta, birçok GCP servisinden/third-party'den gelen event'lere tepki vermek | Eventarc |
| Merkezi olarak kontrol edilen, stateful, retry'lı ve tam observability'li, uzun süren bir süreç | Workflows |
| Belirli, bilinen bir servisi asenkron olarak çalıştırmak, isteğe bağlı olarak gelecekte bir zamana erteleyerek | Cloud Tasks |
| Tekrarlayan bir programa (cron) göre bir job çalıştırmak | Cloud Scheduler |

---

# Modül Özeti

Microservices, koordinasyon karmaşıklığını tekil servislerin dışına, servislerin birbiriyle nasıl iletişim kurduğuna iter. İki desen bunu ele alır: servislerin merkezi bir source of truth olmadan birbirlerinin event'lerine bağımsız olarak tepki verdiği **service choreography**, ve bir orchestrator'ın tüm süreci kontrol ettiği — visibility ve daha kolay troubleshooting karşılığında tek bir arıza noktası (single point of failure) taşıyan — **service orchestration**.

Google Cloud, ikisini de implemente etmek için bir "Application Integration" araç kutusu sağlar: publish/subscribe messaging için **Pub/Sub** (at-least-once delivery, pull ya da push subscription); birçok event source'unu otomatik olarak bağlayan ve CloudEvents formatını standardize eden, Pub/Sub üzerine kurulu bir abstraction olan **Eventarc**; orchestration'ın yönetilen implementasyonu olan, stateful, observable, uzun süren merkezi bir süreç sağlayan **Workflows**; Pub/Sub'ın implicit, subscriber-agnostic invocation'ının aksine, belirli, bilinen bir servisi açıkça (explicit) asenkron olarak çağırmak için **Cloud Tasks**; ve yönetilen bir cron scheduler olan **Cloud Scheduler** (varsayılan ve önerilen time zone: UTC, daylight-saving atlama/tekrar çalışma sorunlarından kaçınmak için). Pratikte, gerçek uygulamalar bir servisin içinde orchestration'ı (Workflows) servisler arasında choreography ile (Eventarc) birleştirir.

---

# Önemli Noktalar

- Choreography = dağıtık kontrol, merkezi source of truth yok, decentralized takımlar/organizasyonlar için güçlü. Orchestration = merkezi kontrol, single point of failure, karmaşık, merkezi olarak yönetilen süreçler için güçlü.
- Pub/Sub at-least-once delivery sağlar; bir mesaj, ancak acknowledge edildikten sonra bir subscriber'ın kuyruğundan kaldırılır.
- Eventarc, transport katmanı olarak Pub/Sub'ın üzerine kuruludur — rakip bir ürün değildir, birçok event source'u ve standardize edilmiş CloudEvents formatı ekleyen bir abstraction'dır.
- Workflows, state tutabilen, retry yapabilen, poll edebilen ya da bir yıla kadar bekleyebilen merkezi bir source of truth'tur, bu da uzun süren süreçleri pratik hale getirir.
- Cloud Tasks explicit invocation kullanır (tam olarak hangi servisi ve ne zaman kullanacağını sen seçersin); Pub/Sub implicit invocation kullanır (sen publish edersin, kim subscribe olduysa o tepki verir).
- Cloud Scheduler job'ları standart Unix cron syntax'ı kullanır; daylight-saving-time kaynaklı atlama/tekrar çalışma bug'larından kaçınmak için varsayılan olarak UTC kullan.
- Gerçek sistemler genelde her iki deseni de birleştirir — servis sınırı içinde orchestration (Workflows), servis sınırları arasında choreography (Eventarc).
