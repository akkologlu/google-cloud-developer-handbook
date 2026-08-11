# Introduction to Cloud Run Functions — Baştan Sona Öğretici

> Bu metin, **"Developing Applications with Cloud Run Functions on Google Cloud"** kursunun **Modül 1 — Introduction to Cloud Run Functions** dersinde anlatılan **her şeyi** kavratmak için yazıldı. Bu, bambaşka ve yepyeni bir kurs — önceki modüllerde (09-11) işlenen "Service Orchestration and Choreography" kursuyla **hiçbir doğrudan devamlılığı yok**; orada microservices'i birbirine bağlayan Pub/Sub, Eventarc ve Workflows gibi araçlar işlenmişti, burada ise bu araçların **çoğu zaman tetiklediği ya da tetiklenmesine aracılık ettiği** hesaplama (compute) katmanının kendisine, yani **Cloud Run functions**'a odaklanıyoruz. Kurs, Google Cloud'un tam yönetilen (fully managed), sunucusuz (serverless) fonksiyon-olarak-hizmet (Functions-as-a-Service, FaaS) platformunu; onu nasıl geliştirip test edeceğini, dağıtacağını (deploy), güvenli hale getireceğini ve veritabanları gibi kaynaklara bağlayacağını öğretiyor. Bu ilk modül, o yolculuğun temel taşlarını atıyor: Cloud Run functions nedir, neden var, hangi iki türü/nesli vardır, nasıl bir geliştirme akışı izler, hangi kullanım alanlarına hizmet eder ve nasıl inşa edilip dağıtılır.
>
> **Kapsam notu:** Bu doküman, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun sadece **Modül 1**'ini kapsıyor. Kursun ilerleyen modüllerinin transkriptleri eklendikçe, bu handbook'a `deep-dive/13-...`, `deep-dive/14-...` gibi ayrı numaralı modüller olarak eklenmeye devam edilecek.

---

## Bu ders tam olarak neyi öğretiyor ve neden önemli?

Bu modülün kendi açılışında söylediği gibi: dersin amacı, Cloud Run functions'ı **tanıtmak**, özelliklerini ve faydalarını tartışmak, kullanım alanlarını gözden geçirmek ve Google Cloud console üzerinde ilk fonksiyonunu oluşturacağın bir laboratuvar (lab) ile pekiştirmek. Bu doküman, transkriptte anlatılan **kavramsal** kısmın tamamını, iki doğal yarıya ayırarak işliyor:

1. **Cloud Run functions nedir ve nasıl çalışır** — platformun tanımı, iki nesli (Cloud Run functions ve Cloud Run functions 1st generation), geliştirme iş akışı, özellikleri ve faydaları, iki fonksiyon türü (HTTP functions ve event-driven functions), event-driven fonksiyonları implemente etmenin iki yolu (CloudEvent functions ve Background functions), platformun kapasite/ölçekleme/revizyon yetenekleri, kullanım alanları, ve son olarak dil runtime'ları, kaynak kodu yapısı, entry point kavramı ve region seçimi.
2. **Cloud Run functions nasıl inşa edilip dağıtılır (build & deploy)** — gerekli IAM rolleri, dağıtım araçları (console, Cloud Build, Cloud Code, gcloud CLI), kaynak kodunu nereden dağıtabileceğin (yerel makine, Cloud Storage, kaynak deposu), console'daki satır içi (inline) editör, ve arka planda kaynak kodunu çalışan bir container image'a dönüştüren **otomatik build pipeline**'ı (Cloud Build → Artifact Registry).

Bu iki yarıyı ayrı ayrı ürün broşürleri olarak değil, **tek bir sorunun iki yüzü** olarak düşün: birinci yarı "bu platform ne işe yarar ve nasıl davranır" sorusuna, ikinci yarı ise "bu platforma kodumu nasıl teslim ederim" sorusuna cevap veriyor. İkisini birlikte öğrendiğinde, sadece Cloud Run functions'ın **ne olduğunu** değil, onu **gerçekten kullanmaya başlayabilecek** kadar derinlemesine anlamış olacaksın.

---

# BÖLÜM 1 — Cloud Run Functions Nedir? İki Nesil, Tek Platform

## Cloud Run functions'ın tanımı

**Cloud Run functions**, Google Cloud üzerinde fonksiyon inşa etmen, dağıtman ve çalıştırman için **tam yönetilen (fully managed) sunucusuz bir execution ortamıdır.** Ölçeklenebilir bir **fonksiyon-olarak-hizmet (Functions-as-a-Service, FaaS) platformu**dur — herhangi bir altyapı sağlamana (provision) ya da tek tek sunucuları yönetmene **gerek yoktur.**

İhtiyaçlarına bağlı olarak, Cloud Run functions'dan diğer bulut servislerine bağlanabilirsin. Platform, gözlemlenebilirlik (observability) ve teşhis (diagnosis) için **Cloud Observability** ile tam olarak entegredir.

> **Analoji:** Cloud Run functions'ı, bir **elektrik prizine** benzet. Elektrik şirketiyle ilgilenmen gereken tek şey, prize bir şey takmak — santralin nasıl çalıştığı, kabloların nereden geçtiği, voltajın nasıl dengelendiği tamamen görünmezdir ve seni ilgilendirmez. Cloud Run functions'da da aynı mantık işler: sen sadece "şu event olduğunda şu kod çalışsın" dersin; hangi sunucunun bu kodu çalıştıracağı, o sunucunun nasıl ölçekleneceği, hangi işletim sisteminin yama alacağı tamamen Google'ın sorumluluğundadır.

## İki nesil: Cloud Run functions vs. Cloud Run functions (1st generation)

Cloud Run functions'ın **iki versiyonu** vardır, ve bu ayrımı net tutmak sınav açısından da pratik açıdan da kritik önemdedir:

- **Cloud Run functions** — eskiden **Cloud Functions (2nd generation)** olarak biliniyordu. Fonksiyonlarını **Cloud Run üzerinde servisler olarak dağıtır (deploy)** ve bu fonksiyonları **Eventarc** ve **Pub/Sub** kullanarak tetiklemene (trigger) olanak tanır.
- **Cloud Run functions (1st generation)** — eskiden **Cloud Functions (1st generation)** olarak biliniyordu. Fonksiyonların **orijinal versiyonudur**; **sınırlı event trigger'ları** ve **sınırlı yapılandırılabilirlik (configurability)** sunar.

> **Sınav tuzağı — "Cloud Functions" ismi hâlâ geçerli mi?** Kurs boyunca ürünün güncel adının **Cloud Run functions** olduğuna dikkat et; "Cloud Functions (2nd generation)" ve "Cloud Functions (1st generation)" artık bu iki neslin **eski (former) isimleridir.** Bir sınav sorusu "Cloud Functions 2nd generation nedir?" diye sorarsa, bunun bugünkü karşılığının **Cloud Run functions** olduğunu bilmen gerekir — bu iki isim **aynı ürünü** tarif eder, farklı ürünler değildir. Buna karşılık **Cloud Run functions (1st generation)**, gerçekten **daha kısıtlı, ayrı bir nesildir** — sadece isim değişikliği değil, gerçek bir yetenek farkı vardır (trigger çeşitliliği ve yapılandırılabilirlik açısından).

## Cloud Run functions'ın altındaki platform: Cloud Run

Cloud Run functions'ın "Cloud Run üzerinde servisler olarak dağıtıldığı" ifadesi boşuna değil — bunun ne anlama geldiğini anlamak için önce **Cloud Run**'ın kendisini tanımak gerekir.

**Cloud Run**, container'ları doğrudan Google'ın ölçeklenebilir altyapısı üzerinde çalıştırmanı sağlayan, **yönetilen bir compute platformudur.** Eğer bir kod parçasından **container image** inşa edebiliyorsan, o kodu — hangi programlama dilinde yazılmış olursa olsun — Cloud Run üzerinde dağıtabilirsin.

Uygulamanı Go, Node.js, Python, Java, .NET Core ya da Ruby ile geliştiriyorsan, **kaynak tabanlı dağıtım (source-based deployment)** seçeneğini kullanabilirsin — bu seçenek, container'ı **senin yerine** inşa eder. Cloud Run, Google Cloud'daki diğer servislerle iyi çalışır; böylece tam donanımlı uygulamalar inşa ederken Cloud Run servisini işletmeye (operate), yapılandırmaya (configure) ve ölçeklendirmeye (scale) fazla zaman harcamazsın.

> **Bu neden önemli?** Cloud Run functions'ı, Cloud Run'ın üzerine kurulu **daha dar kapsamlı, fonksiyona-özel bir katman** olarak düşünebilirsin. Cloud Run sana "herhangi bir container'ı çalıştır" derken, Cloud Run functions sana "tek bir işlevi yerine getiren, event'lerle tetiklenen küçük bir kod parçasını çalıştır" der — ve bu küçük kod parçası, arka planda yine Cloud Run'ın ölçeklenebilir container altyapısı üzerinde koşar. Bu ilişki, ilerleyen bölümlerde göreceğin "fonksiyonunu Cloud Run'a taşıyabilirsin" (portability) özelliğinin de temelini oluşturur.

## Bir cloud function nedir?

**Bir cloud function**, senin yazdığın, **tek bir işlevi (single piece of functionality)** yerine getiren basit bir koddur. Cloud infrastructure ve diğer servisler tarafından üretilen bir **event ile tetiklenir.**

Cloud function'lar, **tam yönetilen sunucusuz bir ortamda** çalışır. Herhangi desteklenen bir dilde cloud function geliştirebilirsin.

Cloud Run functions, bulut servislerini birbirine bağlamak ve genişletmek için kod yazmanı sağlayan **bağlayıcı bir mantık katmanı (connective layer of logic)** sunar. Örneğin bir cloud function, Cloud Storage'a bir dosya yüklendiğinde ya da bir Pub/Sub topic'ine gelen bir mesaja **tepki verebilir.**

> **Analoji:** Bir cloud function'ı, bir binadaki **elektrikli anahtarlar (switch)** gibi düşün. Her anahtar, tek bir işi yapar — ışığı açar/kapatır, asansörü çağırır, alarmı tetikler. Anahtarın kendisi karmaşık bir makine değildir; sadece "şu olay olduğunda şu tepkiyi ver" der. Cloud function da böyledir: karmaşık, çok amaçlı bir uygulama değil, **tek bir tetikleyiciye tek bir tepki veren**, küçük ve odaklı bir kod parçasıdır.

## Cloud Run functions kullanmanın üç adımı

Cloud Run functions'ı kurup kullanmak için izlediğin akış üç adımdan oluşur:

1. **Geliştir (develop).** Fonksiyon kodunu ya da mantığını, desteklenen programlama dillerinden birinde geliştirirsin.
2. **Dağıt (deploy).** Fonksiyonu Google Cloud console ya da gcloud CLI kullanarak dağıtabilirsin.
3. **Trigger kur.** Fonksiyonun HTTP isteklerine ya da desteklenen cloud event'lerine tepki olarak çalışması için bir **trigger** kurarsın.

Bu üç adım — geliştir, dağıt, trigger kur — bu dokümanın geri kalanının **iskeletidir**: BÖLÜM 3-4 "trigger kur" adımının iki farklı biçimini (HTTP vs. event-driven), BÖLÜM 7 "geliştir" adımının dil/dosya-yapısı detaylarını, BÖLÜM 8-9 ise "dağıt" adımının araçlarını ve mekaniğini derinlemesine işliyor.

**Cloud Run functions vs. Cloud Run functions (1st generation) — özet tablo:**

| Özellik | Cloud Run functions | Cloud Run functions (1st generation) |
| --- | --- | --- |
| Eski adı | Cloud Functions (2nd generation) | Cloud Functions (1st generation) |
| Dağıtım biçimi | Cloud Run üzerinde servis olarak dağıtılır | Kendi orijinal execution modeli |
| Trigger'lar | Eventarc ve Pub/Sub ile tetiklenir | Sınırlı event trigger'ları |
| Yapılandırılabilirlik | Daha geniş yapılandırma seçenekleri | Sınırlı yapılandırılabilirlik |
| CloudEvent functions desteği | Tüm dil runtime'larında | Sadece .NET, Ruby, PHP'de |
| Background functions desteği | Yok (bu nesle özgü değil) | Node.js, Python, Go, Java'da |

---

# BÖLÜM 2 — Geliştirme İş Akışı, Özellikler ve Faydalar

## Özellikler (features)

Cloud Run functions'ın sunduğu özellikler, geliştirici deneyimini ve entegrasyon yeteneklerini tanımlar:

- **Yerel geliştirme ve test (local development and testing).** Fonksiyonlarını, Cloud Run functions'a dağıtmadan önce yerel bir geliştirme ortamında geliştirebilir ve test edebilirsin.
- **Sorunsuz kimlik doğrulama (seamless authentication).** Cloud function'larını, **service account'lar** kullanarak diğer Google Cloud servisleriyle sorunsuzca kimlik doğrulayabilirsin.
- **HTTP ve event-driven.** Cloud Run functions, hem cloud event'lerine hem de HTTP isteklerine tepki olarak çalışır.
- **Veritabanı entegrasyonu.** **Cloud SQL, Bigtable, Spanner** ve **Firestore** veritabanlarıyla entegrasyon sağlar.
- **Taşınabilirlik (portability).** Bir fonksiyon, desteklenen dillerden biri için **herhangi bir standart runtime ortamında** çalışabilir.

## Faydalar (benefits)

- **Genişletme ve zenginleştirme.** Mevcut bulut servislerini **programlama mantığıyla** zenginleştirmene ve genişletmene olanak tanır.
- **Sunucusuz (serverless).** Yazılım ve altyapı tamamen Google tarafından yönetilir. Sunucuları yönetmen, framework'leri güncellemen ya da işletim sistemlerini yamalaman **gerekmez.**
- **Otomatik ölçekleme (autoscaling).** Kaynaklar, event'lere tepki olarak **otomatik olarak sağlanır.** Bir fonksiyon, günde birkaç çağrıdan milyonlarca çağrıya kadar, **hiçbir ek çalışma gerektirmeden** ölçeklenebilir.
- **Gözlemlenebilir (observable).** Cloud Run functions, gözlemlenebilirlik ve hata teşhisi için **Google Cloud Observability**'nin monitoring ve logging araçlarıyla entegredir.
- **Fiyatlandırma.** **Kullandığın kadar öde (pay-as-you-go)** modeli kullanılır. Maliyet; fonksiyon çağrılarının sayısına, fonksiyonun ne kadar süre çalıştığına (compute time) ve giden (outbound) ağ trafiği için veri transfer ücretlerine dayanır.

> **Analoji — otomatik ölçeklemenin gerçek anlamı:** Bir fonksiyonun "günde birkaç çağrıdan milyonlarca çağrıya" ölçeklenmesini, bir kafenin sabahın erken saatlerinde tek bir barista ile açılıp, öğle yemeği saatinde aniden **onlarca baristanın** aynı anda kahve yapmaya başlamasına benzet — ama burada hiç kimsenin sabahtan telefon açıp "bugün kalabalık olacak, ekstra personel ayarlayın" demesine gerek yoktur. Sistem, kapıdan giren müşteri sayısına (event hacmine) göre **kendiliğinden** büyür ve küçülür.

> **Sınav tuzağı — "serverless" sunucu olmadığı anlamına mı gelir?** Hayır. "Serverless" (sunucusuz), fiziksel ya da sanal sunucuların **var olmadığı** anlamına gelmez — sunucular elbette vardır. "Serverless" demek, **sen** bu sunucuları **görmezsin, yönetmezsin, yamalamazsın, ölçeklendirmezsin** demektir. Ders bunu açıkça "yazılım ve altyapı tamamen Google tarafından yönetilir" diyerek belirtiyor — sorumluluk Google'a geçmiştir, sunucular ortadan kalkmamıştır.

---

# BÖLÜM 3 — İki Fonksiyon Türü: HTTP Functions vs. Event-Driven Functions

Cloud Run functions'ın **iki türü** vardır: **HTTP functions**, HTTP isteklerini işler; **event-driven functions** ise cloud ortamından gelen event'leri işler. Bu ayrım, bir fonksiyonun **nasıl tetikleneceğine** dair temel bir tasarım kararıdır.

## HTTP functions

Bir fonksiyonu bir HTTP(S) isteğiyle çağırmak istediğinde, **HTTP trigger'lı HTTP functions** kullanılır. Bir fonksiyon için bir HTTP trigger belirttiğinde, fonksiyona **istek alabileceği bir URL atanır.** Bir HTTP trigger, fonksiyonun HTTP(S) isteklerine tepki olarak çalışmasını sağlar.

HTTP functions kullanmak için iyi bir kullanım senaryosu, **diğer servislerden gelen HTTP isteklerini işleyen bir webhook ya da API** implemente etmektir.

**Varsayılan olarak, HTTP functions'a gelen istekler kimlik doğrulama (authentication) gerektirir.** Fonksiyon dağıtımı sırasında, kimliği doğrulanmamış (unauthenticated) isteklere izin vermeyi **seçebilirsin.** (Cloud Run functions'ı güvenli hale getirmeyi, kursun ilerleyen bir modülünde daha detaylı ele alınıyor.)

Bir HTTP function implemente etmek için, bir **HTTP handler fonksiyonu** yazar ve bunu programlama dilin için olan **Functions Framework** ile kaydedersin (register). Handler, request ve response argümanlarını kabul eder, isteği **istek metoduna (request method)** göre işler ve geri bir HTTP yanıtı gönderir.

> **Sınav tuzağı — arka plan işlerini unutmak:** Eğer fonksiyon herhangi bir arka plan görevi (thread, process, Promise object gibi) oluşturuyorsa, bu görevler **yanıt gönderilmeden önce sonlandırılmalıdır (terminated).** Bu ayrıntı kolayca gözden kaçar: bir geliştirici, yanıtı gönderdikten sonra arka planda "biraz daha iş" yapmaya devam edeceğini varsayabilir, ama serverless bir ortamda **yanıt gönderildikten sonra fonksiyonun execution ortamı her an durdurulabilir** — bu yüzden tüm işin, yanıt gönderilmeden **önce** bitmiş olması gerekir.

## Event-driven functions

Cloud ortamında gerçekleşen bir event'e tepki olarak bir fonksiyonun **otomatik olarak** çağrılmasını istediğinde, **event-driven functions** kullanılır.

Event-driven functions'ı, seçtiğin dil runtime'ına ve kullandığın ürünün Cloud Run functions mi yoksa Cloud Run functions (1st generation) mi olduğuna bağlı olarak, **CloudEvent functions** ya da **Background functions** olarak implemente edersin (bu ikisinin farkını BÖLÜM 4'te detaylıca işliyoruz).

Event-driven functions, Google Cloud projendeki event'lere tepki veren **event trigger'lar** kullanır. Event trigger'lar, fonksiyonun Cloud Run functions'dan mı yoksa Cloud Run functions (1st generation)'dan mı olduğuna bağlı olarak, birçok Google Cloud servisi tarafından desteklenir.

Event trigger'ları şu kaynaklar için kullanabilirsin:

- **Pub/Sub**
- **Cloud Storage**
- **Firestore**
- **Firebase**
- **Eventarc**

Ayrıca, Pub/Sub'ı bir event bus olarak destekleyen **herhangi bir başka Google Cloud servisiyle** de Cloud Run functions'ı entegre edebilirsin. (Trigger'lar, kursun ilerleyen bir modülünde daha detaylı ele alınıyor.)

Event-driven functions, yeni bir Pub/Sub mesajının alınması ya da Cloud Storage'da bir dosyanın silinmesi gibi bir event'e tepki olarak çalıştırılır.

**HTTP functions vs. Event-driven functions — özet tablo:**

| Özellik | HTTP functions | Event-driven functions |
| --- | --- | --- |
| Nasıl tetiklenir? | HTTP(S) isteğiyle, atanmış bir URL üzerinden | Cloud ortamındaki bir event ile, otomatik olarak |
| Tipik kullanım senaryosu | Webhook, API | Dosya yükleme/silme, yeni mesaj, veritabanı değişikliği |
| Kimlik doğrulama | Varsayılan olarak zorunlu (istersen kapatılabilir) | Event trigger'ın kendi yapılandırmasına bağlı |
| Implementasyon | HTTP handler, Functions Framework ile kayıtlı | CloudEvent function ya da Background function |
| Trigger kaynakları | HTTP(S) istekleri | Pub/Sub, Cloud Storage, Firestore, Firebase, Eventarc |

---

# BÖLÜM 4 — Event-Driven Fonksiyonları Implemente Etmek: CloudEvent Functions vs. Background Functions

Event-driven bir fonksiyonu, ya bir **CloudEvent function** ya da bir **Background function** olarak implemente edersin. Bu iki yaklaşım, aynı işi (event'e tepki vermek) yapar ama **farklı nesillerde, farklı dillerde** desteklenir — bu yüzden ikisini karıştırmamak önemlidir.

## CloudEvent functions

CloudEvent functions:

- **CloudEvents endüstri standardı spesifikasyonuna** dayanır — event verisini tanımlamak için kullanılan bir standarttır.
- **Functions Framework** ile kayıt edilir — kullanıcı fonksiyonlarını kalıcı (persistent) bir HTTP uygulamasının içine saran, açık kaynaklı bir kütüphanedir.
- **Cloud Run functions** tarafından, **tüm dil runtime'larıyla** kullanılır.
- **Cloud Run functions (1st generation)** tarafından ise sadece **.NET, Ruby ve PHP** ile kullanılır.

## Background functions

Background functions:

- **Daha eski stildeki** event-driven fonksiyonlardır; event verisini, **event'in türüne göre (based on the type of event)** alırlar.
- **Cloud Run functions (1st generation)** tarafından, **Node.js, Python, Go ve Java** runtime'larıyla kullanılır.

> **Analoji:** CloudEvent functions'ı **evrensel bir posta zarfı standardına** benzet — zarfın boyutu, üzerindeki alanlar (gönderen, alıcı, tarih) her zaman **aynı biçimde** düzenlenmiştir, hangi postane ya da ülkeden geldiği fark etmez. Background functions ise, her postanenin **kendi özel zarf biçimini** kullandığı, daha eski ve standardize edilmemiş bir sisteme benzer — zarfı açmadan önce hangi postaneden geldiğini bilmen, ona göre farklı davranman gerekir.

> **Sınav tuzağı — bu ikisini nesil ve dil eksenlerinde karıştırmak:** Bu tablo, sınavda en çok karıştırılan noktalardan biridir çünkü **iki farklı eksen** aynı anda değişiyor: nesil (Cloud Run functions vs. 1st generation) ve dil runtime'ı. Akılda tutman gereken kural şu: **Cloud Run functions, event-driven fonksiyonlar için sadece CloudEvent functions kullanır — hangi dil olursa olsun.** Buna karşılık **Cloud Run functions (1st generation)**, dile göre ikiye ayrılır: Node.js/Python/Go/Java'da **Background functions**, .NET/Ruby/PHP'de ise **CloudEvent functions** kullanır. Bir sınav sorusu "Java ile yazılmış, Cloud Run functions (1st generation)'da çalışan bir event-driven fonksiyon nasıl implemente edilir?" diye sorarsa, cevap **Background function**'dır — CloudEvent function değil.

**CloudEvent functions vs. Background functions — özet tablo:**

| Özellik | CloudEvent functions | Background functions |
| --- | --- | --- |
| Dayandığı standart | CloudEvents endüstri standardı | Event türüne göre özel veri biçimi |
| Kayıt mekanizması | Functions Framework | (Nesle özgü eski mekanizma) |
| Cloud Run functions'da desteklenen diller | Tüm dil runtime'ları | Desteklenmez |
| Cloud Run functions (1st generation)'da desteklenen diller | .NET, Ruby, PHP | Node.js, Python, Go, Java |
| Karakter | Yeni, standardize, ileriye dönük | Eski stil (older style) |

---

# BÖLÜM 5 — Platformun Motor Gücü: Kapasite, Ölçekleme, Revizyonlar ve Eventarc

Cloud Run functions, altyapı seviyesinde bir dizi somut yetenek sunar. Bunları tek tek tanımak, hem sınav hem de gerçek mimari kararları için önemlidir.

## Kapasite ve eşzamanlılık (concurrency)

- Fonksiyon instance'ları, **32 GB'a kadar RAM** ve **4 vCPU'ya kadar** kaynağa sahip olabilir.
- Tek bir fonksiyon instance'ı, **1000'e kadar eşzamanlı isteği (concurrent requests)** işleyebilir.

Bu eşzamanlılık yeteneği, **cold start'ları azaltır** ve ölçeklenirken genel gecikmeyi (latency) iyileştirir.

> **Neden bu gerçekten önemli?** Eğer tek bir instance sadece bir isteği aynı anda işleyebilseydi, trafik arttığında Cloud Run functions her yeni istek için **yeni bir instance başlatmak (cold start)** zorunda kalırdı — ve her cold start, ekstra gecikme demektir. Tek bir instance'ın 1000 eşzamanlı isteği işleyebilmesi, **zaten ısınmış (warm)** bir instance'ın çok daha fazla trafiği karşılayabileceği, dolayısıyla yeni instance başlatma ihtiyacının daha az sıklıkla ortaya çıkacağı anlamına gelir.

## Zaman aşımı (timeout) limitleri

- **HTTP functions** için çalışma süresi limiti **60 dakikaya kadar.**
- **Event-driven functions** için çalışma süresi limiti **10 dakikaya kadar.**

Bu limitler, daha uzun süren istek iş yüklerini (request workloads) işleyebilmen için tanımlanmıştır.

> **Sınav tuzağı — iki timeout değerini karıştırmak:** HTTP functions ve event-driven functions'ın timeout limitleri **farklıdır** — HTTP functions çok daha uzun (60 dakika), event-driven functions ise çok daha kısadır (10 dakika). Bir sınav sorusu "bir event-driven fonksiyon en fazla ne kadar çalışabilir?" diye sorarsa, cevap **10 dakika**dır, 60 dakika değil — 60 dakika sadece HTTP functions için geçerlidir.

## Revizyonlar, trafik bölme ve rollback

Fonksiyonunu her dağıttığında (deploy), **yeni bir revizyon (revision)** oluşturulur. Bu sayede:

- **Trafiği farklı revizyonlar arasında bölebilirsin (traffic splitting).**
- Fonksiyonunu **önceki bir revizyona geri alabilirsin (rollback).**

## Taşınabilirlik: Cloud Run'a ya da Kubernetes'e

Cloud Run'ın ölçeklenebilir container platformundan yararlanarak, fonksiyonunu **Cloud Run'a** ya da **Kubernetes'e** taşıyabilirsin.

> **BÖLÜM 1'deki hangi fikri somutlaştırıyor?** BÖLÜM 1'de Cloud Run functions'ın Cloud Run'ın üzerine kurulu, fonksiyona-özel bir katman olduğunu görmüştük. Bu taşınabilirlik özelliği, o ilişkinin **pratik sonucudur**: fonksiyonun zaten bir container image olarak paketlendiği için, ihtiyaç duyduğunda onu doğrudan Cloud Run'a (daha genel bir container platformuna) ya da Kubernetes'e (daha da fazla kontrol isteyen senaryolar için) taşıyabilirsin — kilitlenip kalmazsın (no lock-in).

## Eventarc ile 90'dan fazla event kaynağı

**Eventarc** ile, event-driven fonksiyonlar **90'dan fazla Google, harici SaaS event kaynağından** ve Pub/Sub'a publish ederek oluşturulan **özel (custom) kaynaklardan** tetiklenebilir.

Endüstri standardı **CloudEvents** formatı ile, event işleme kodunu **basitleştirebilirsin.**

## Google Cloud console'daki yönetim yetenekleri

Google Cloud console'da:

- Fonksiyonunun **trigger'larını yapılandırabilirsin.**
- **Dağıtımları (deployments) izleyebilirsin.**
- **Fonksiyonlarını test edebilirsin.**
- **Fonksiyon metriklerini görüntüleyebilirsin.**

Cloud Run functions ile Cloud Run functions (1st generation) arasındaki karşılaştırma için ilgili dokümantasyona bakabilirsin.

**Platform kapasitesi — özet tablo:**

| Yetenek | Değer / açıklama |
| --- | --- |
| Maksimum RAM | 32 GB'a kadar |
| Maksimum vCPU | 4 vCPU'ya kadar |
| Eşzamanlı istek (tek instance) | 1000'e kadar |
| HTTP functions timeout | 60 dakikaya kadar |
| Event-driven functions timeout | 10 dakikaya kadar |
| Revizyonlar | Her deploy'da yeni revizyon; trafik bölme ve rollback mümkün |
| Taşınabilirlik | Cloud Run'a ya da Kubernetes'e taşınabilir |
| Event kaynak sayısı (Eventarc ile) | 90'dan fazla Google/SaaS/özel kaynak |
| Event formatı | Endüstri standardı CloudEvents |

---

# BÖLÜM 6 — Kullanım Alanları (Use Cases)

Cloud Run functions'ın tipik kullanım alanları, platformun "event'lere tepki ver, tek bir işi yap" doğasını doğrudan yansıtır:

- **Veri işleme (data processing).** Cloud Storage'daki dosyalar ve nesneler (objects) üzerindeki event'lere tepki vererek, veriyi doğrulayabilir (validate) ve dönüştürebilirsin (transform); ayrıca ileri işleme (downstream processing) için görselleri ve videoları işleyebilirsin.
- **Webhook'lar ve API'ler.** HTTP functions ile, harici sistemlerden gelen HTTP isteklerini işleyerek webhook ve API çağrılarını destekleyebilirsin.
- **Mobil cihaz backend'i.** Firebase tarafından tetiklenen event'lere tepki veren bir mobil cihaz backend'i geliştirebilirsin. Cloud Run functions ile, **Firebase ve Firestore** tarafından tetiklenen event'lere tepki veren mobil cihaz backend servisleri geliştirebilirsin.
- **IoT (nesnelerin interneti).** Cihazlardan Pub/Sub'a akan (streamed) veriyi işleyip saklayarak IoT uygulamaları geliştirebilirsin.

> **Bu dört kullanım alanını nasıl okumalısın?** Dikkat edersen, bu dört kullanım alanının her biri, BÖLÜM 3'te öğrendiğin **iki fonksiyon türünden birine** doğrudan bağlanıyor: veri işleme ve mobil/IoT senaryoları **event-driven functions**'ı (Cloud Storage, Firebase/Firestore, Pub/Sub trigger'ları) kullanırken, webhook/API senaryosu **HTTP functions**'ı kullanır. Bu, kullanım alanlarının rastgele bir liste olmadığını, platformun temel iki yeteneğinin (HTTP ve event-driven) doğal birer uzantısı olduğunu gösterir.

---

# BÖLÜM 7 — Dil Runtime'ları, Kaynak Kod Yapısı, Entry Point ve Region Seçimi

## Desteklenen dil runtime'ları

Cloud Run functions'ı, desteklenen dillerden herhangi birinde geliştirebilirsin. Her dil runtime'ının **belirli desteklenen versiyonları** vardır; versiyon numaraları için Cloud Run functions dokümantasyonuna başvurman gerekir (bu doküman, sürekli güncellenen versiyon numaralarını içermez).

Seçtiğin dil runtime'ı ve yazmak istediğin fonksiyon türü, kodunun **yapısını (structure)** ve fonksiyon implementasyonunu belirler. Cloud Run functions'ın fonksiyon tanımını bulabilmesi için, her dil runtime'ının kaynak kodunu yapılandırmak için **belirli gereksinimleri** vardır.

## Dil bazında dizin yapıları (directory structures)

- **Node.js:** Varsayılan olarak, Cloud Run functions kaynak kodunu, fonksiyon dizininin kökündeki (root) **`index.js`** adlı dosyadan yükler. Farklı bir ana kaynak dosyası belirtmek için, `package.json` dosyandaki **`main`** alanını kullanabilirsin. `package.json` dosyası ayrıca, Node.js için **Functions Framework**'ü bir bağımlılık (dependency) olarak, fonksiyonunun ihtiyaç duyduğu ek kütüphane bağımlılıklarıyla birlikte içerir.
- **Python:** Cloud Run functions kaynak kodunu, fonksiyon dizininin kökündeki **`main.py`** adlı dosyadan yükler. `requirements.txt` dosyası, Python için Functions Framework'ü bir bağımlılık olarak, fonksiyonunun ihtiyaç duyduğu ek kütüphane bağımlılıklarıyla birlikte içerir.
- **Go:** Fonksiyonun, projenin kökünde bir **Go package** içinde olması gerekir. `go.mod` dosyası, Go için Functions Framework'ü bir bağımlılık olarak, fonksiyonunun ihtiyaç duyduğu ek kütüphane bağımlılıklarıyla birlikte içerir.

Kaynak kodu dizin yapısını tercih ettiğin dil için oluşturma konusunda daha fazla ayrıntı için Cloud Run functions dokümantasyonuna başvurabilirsin.

> **Analoji:** Bu dosya adlandırma kuralları (convention), bir binadaki **"ana giriş kapısı her zaman zemin kattadır"** kuralına benzer. Bir kurye (Cloud Run functions), binanın (fonksiyon dizininin) içinde neyin nerede olduğunu bilmez — ama "ana giriş her zaman şu isimde, şu konumdadır" kuralını bildiği için, her binada aynı yeri arayarak doğru kapıyı bulabilir. `index.js`, `main.py` ve kök dizindeki Go package'ı, tam olarak bu "her zaman burada ara" kuralının dil bazındaki karşılıklarıdır.

## Entry point kavramı

**Fonksiyon entry point'i**, Cloud Run function çağrıldığında (invoke edildiğinde) çalıştırılan koddur. Kaynak kodun, bir **entry point tanımlamalıdır** ve bu tanım, ana dosyanda ya da kök package'ında yer almalıdır.

Programlama diline bağlı olarak, entry point bir **fonksiyon** ya da bir **sınıf (class)** olabilir. Bu entry point'i, fonksiyonunu **dağıttığında (deploy)** belirtirsin.

> **Bu neden ayrı bir kavram olarak öğretiliyor?** Dosya konumu (index.js, main.py, Go package) Cloud Run functions'a "kaynak kodun nerede" sorusunu cevaplarken, entry point "bu dosyanın/paketin içindeki **hangi fonksiyon ya da sınıf** çalıştırılacak" sorusunu cevaplar. Bir dosyada birden fazla fonksiyon tanımlı olabilir — entry point, bunlardan **hangisinin** gerçek giriş noktası olduğunu açıkça belirtir.

## Region seçimi

Cloud Run functions'ı çalıştıran altyapı, **belirli bir bölgede (region)** konumlanır ve Google tarafından, o bölge içindeki **tüm zone'larda yedekli (redundantly) olarak** kullanılabilir şekilde yönetilir.

Cloud Run functions'ını çalıştırmak için bir region seçerken, birincil değerlendirme kriterlerin **gecikme (latency)** ve **kullanılabilirlik (availability)** olmalıdır. Genellikle, Cloud Run function'ının kullanıcılarına **en yakın** region'ı seçebilirsin — ama uygulamanın kullandığı **diğer Google Cloud ürün ve servislerinin konumunu** da göz önünde bulundurmalısın.

Servisleri **birden fazla lokasyonda** kullanmak, uygulamanın gecikmesini ve **fiyatlandırmasını** etkileyebilir.

> **Sınav tuzağı — region seçiminde sadece kullanıcı yakınlığına bakmak:** "Region'ı sadece kullanıcılarıma en yakın yere göre seçmeliyim" düşüncesi **eksiktir.** Ders açıkça, uygulamanın kullandığı **diğer Google Cloud servislerinin konumunun da** dikkate alınması gerektiğini vurguluyor — çünkü servisler birden fazla lokasyona yayılırsa, hem **gecikme** hem de **fiyatlandırma** olumsuz etkilenebilir (bölgeler arası veri transferi ek maliyet ve gecikme getirir). Ayrıca unutma: **Cloud Run functions ve Cloud Run functions (1st generation)'ın bölgesel kullanılabilirliği (regional availability) farklıdır** — seçtiğin nesil, hangi bölgelerde dağıtım yapabileceğini de etkiler.

---

# BÖLÜM 8 — Build & Deploy: IAM Gereksinimleri ve Dağıtım Araçları

Şimdi kaynak koddan çalışan bir Cloud Run function'a giden yolu — **build ve deploy sürecini** — inceleyelim.

## Dağıtım süreci ne yapar?

Dağıtım süreci (deployment process), fonksiyonunun **kaynak kodunu** ve **yapılandırma ayarlarını (configuration settings)** alır ve çalıştırılabilir bir **image** inşa eder (build). Cloud Run functions, fonksiyonuna gelen istekleri işleyebilmek için bu image'ı **otomatik olarak yönetir.**

## Gerekli IAM rolleri

Cloud Run functions dağıtımı yapan bir kullanıcının:

1. **Cloud Functions Developer** IAM rolüne (ya da aynı izinleri içeren bir role) sahip olması gerekir.
2. Ayrıca, **Cloud Run functions runtime service account**'u üzerinde **Service Account User** IAM rolüne atanmış olması gerekir.

> **Neden iki ayrı rol gerekiyor?** Bu ayrım, IAM'in genel bir prensibini yansıtıyor: "bir şeyi yapabilme izni" ile "bir kimlik olarak hareket edebilme izni" birbirinden farklıdır. **Cloud Functions Developer**, sana "fonksiyon dağıtabilirsin" der; ama dağıttığın fonksiyon çalışırken **bir service account kimliğiyle** çalışacaktır — ve o service account'u **kullanabilmek (impersonate/act as)** için ayrıca **Service Account User** rolüne ihtiyacın vardır. Bu, birinin sana "bu arabayı kullanabilirsin" demesi ile "bu arabanın anahtarını sana veriyorum" demesi arasındaki farka benzer — ikisi de gereklidir, biri diğerinin yerine geçmez.

## Dağıtım araçları

Fonksiyonunu şu yollardan biriyle dağıtabilirsin:

- **Google Cloud console**
- **Cloud Build** kullanarak
- **Cloud Code** kullanarak
- **gcloud CLI** ile

## gcloud CLI ile dağıtım ve temel flag'ler

Cloud Run functions'ı dağıtmak için, **`--gen2`** flag'ini belirtirsin. Diğer temel flag'ler:

- **`--region`**: Fonksiyonunun dağıtılacağı region'ı belirtir.
- **`--runtime`**: Fonksiyonunun dil runtime'ını belirtir.
- **`--source`**: Fonksiyon kaynak kodunun konumunu belirtir. Kaynak kodu **yerel makinende**, **Cloud Storage'da** ya da **bir kaynak deposunda (source repository)** bulunabilir (bu üç seçeneği BÖLÜM 9'da detaylıca işliyoruz).
- **`--entry-point`**: Kaynak kodundaki, cloud function'ının **entry point'i** olarak kullanılacak fonksiyon ya da sınıf adını belirtir. Fonksiyonun çalıştığında çalıştırılan kod, bu fonksiyon ya da sınıf kodudur.
- **Trigger flag'leri**: Fonksiyonun trigger türünü ve ilgili yapılandırmasını belirtmek için kullanılır. Ayrıntılar için `gcloud functions deploy` komut dokümantasyonuna başvurabilirsin.

## Cloud Code

**Cloud Code**, Cloud Run functions'ın geliştirilmesini ve dağıtılmasını basitleştiren bir araçtır. Fonksiyonları **doğrudan IDE'n içinde** oluşturmana, dağıtmana ve çağırmana (invoke) olanak tanır — bu da süreci daha verimli hale getirir.

> **Analoji:** Dört dağıtım aracını (console, Cloud Build, Cloud Code, gcloud CLI), aynı hedefe (Cloud Run functions'a kod teslim etmek) ulaşan **dört farklı yol** gibi düşün: console, tarayıcıdan tıklayarak giden görsel bir yoldur; gcloud CLI, terminalden komut yazarak giden, otomasyona en uygun yoldur; Cloud Code, hiç IDE'den çıkmadan giden, geliştirici deneyimine en uygun yoldur; Cloud Build ise, bu yolların **arkasında** çalışan, kaynak kodu gerçekten bir container image'a dönüştüren **motor**dur (bu motoru BÖLÜM 9'un sonunda detaylıca göreceğiz).

---

# BÖLÜM 9 — Kaynak Kodu Dağıtım Seçenekleri ve Otomatik Build Pipeline

## Üç kaynak konumu seçeneği

`--source` flag'i ile fonksiyon kaynak kodunun konumunu belirtebilirsin. Üç seçenek vardır:

### 1. Yerel makine (local machine)

`--source` flag'inin değeri, fonksiyon kaynak kodunun **kök dizinine (root directory)** giden, yerel dosya sistemi üzerindeki bir yoldur (path).

İsteğe bağlı olarak, kaynak kodunu dağıtımın bir parçası olarak yükleyeceğin (upload) bir Cloud Storage bucket'ı belirtmek için **`--stage-bucket`** flag'ini kullanabilirsin. Gereksiz dosyaları **`.gcloudignore`** dosyasıyla hariç tutabilirsin.

### 2. Cloud Storage

`--source` flag'inin değeri, fonksiyon kaynak kodunu **zip dosyası olarak paketlenmiş** şekilde içeren bucket'a giden Cloud Storage yoludur. Fonksiyon kaynak dosyaların, **zip dosyasının kökünde** yer almalıdır.

- **Cloud Run functions (1st generation)**'da, dağıtımı gerçekleştiren hesabın bucket'tan okuma izni olması gerekir.
- **Cloud Run functions**'da ise, **Cloud Run functions service agent**'ının bucket'tan okuma izni olması gerekir.

### 3. Kaynak deposu (source repository)

`--source` flag'inin değeri, fonksiyon kaynak kodunun konumuna giden bir **source repository referansıdır.**

Kaynak deponda bir revizyondan (revision) fonksiyon kaynak kodunu dağıtmak için, source repository yoluna **`revisions/revision_name`** belirtebilirsin. Kaynak kodun, depo kökü dışındaki bir konumunu belirtmek için, source repository yoluna **`paths/source_directory_path`** ekleyebilirsin.

**Cloud Run functions service agent**'ının, depo üzerinde **Source Repository Reader (`roles/source.reader`)** IAM rolüne sahip olması gerekir.

Cloud Source Repositories'den dağıtım yapmak, aynı zamanda **GitHub** ya da **Bitbucket** deposunda barındırılan kodu dağıtmanı da mümkün kılar.

> **Sınav tuzağı — Cloud Storage'da izin sorumluluğunu karıştırmak:** Bu, iki nesil arasındaki ince ama önemli bir farktır. **Cloud Run functions (1st generation)**'da, bucket'tan okuma izni **dağıtımı yapan hesaba (deployer)** aittir. **Cloud Run functions**'da ise bu izin, dağıtımı yapan hesaba değil, **Cloud Run functions service agent**'ına aittir. Bir sınav sorusu "Cloud Run functions ile Cloud Storage'dan dağıtım yaparken kimin okuma izni olmalı?" diye sorarsa, cevap **Cloud Run functions service agent**'dır — dağıtımı yapan kullanıcı değil.

## Console'daki satır içi (inline) editör

Google Cloud console'da yer alan **satır içi editör** kullanarak, bir fonksiyonu **doğrudan console üzerinden** yazıp dağıtabilirsin. Editör:

- **Sol pane**: kaynak dosyaları görüntülemek ve seçmek için.
- **Sağ pane**: bir kaynak dosyasını düzenlemek için.

**Kaynak kodu dağıtım seçenekleri — özet tablo:**

| Seçenek | `--source` değeri | İzin gereksinimi (Cloud Run functions) |
| --- | --- | --- |
| Yerel makine | Yerel dosya sistemindeki kök dizin yolu | (Opsiyonel `--stage-bucket` ile Cloud Storage'a yüklenir) |
| Cloud Storage | Zip dosyasını içeren bucket'a giden yol | Cloud Run functions service agent'ın bucket'tan okuma izni |
| Kaynak deposu | Source repository referansı (`revisions/...`, `paths/...`) | Service agent'ın `roles/source.reader` rolü; GitHub/Bitbucket da desteklenir |
| Console inline editör | (Doğrudan console'da yazılır, flag gerekmez) | — |

## Otomatik build pipeline: kaynak koddan çalışan container'a

Fonksiyonunun kaynak kodunu Cloud Run functions'a dağıttığında, bu kaynak kodu bir **Cloud Storage bucket'ında saklanır.**

**Cloud Build**, build'leri Google Cloud altyapısı üzerinde çalıştıran bir servistir. Cloud Build, fonksiyon kaynak kodunu **otomatik olarak bir container image'a inşa eder (build)** ve bu image'ı **Artifact Registry**'ye **push eder.** Cloud Run functions, fonksiyonunu çalıştırmak için container'ı çalıştırması gerektiğinde, bu image'a erişir.

Image'ın inşa edilme süreci **tamamen otomatiktir** ve senden **hiçbir doğrudan girdi (input) gerektirmez.**

Bu sürecin çalışması için, projende **Cloud Build API**'sinin etkinleştirilmiş (enabled) olması gerekir. Build sürecinde kullanılan tüm kaynaklar **kendi kullanıcı projende (user project)** çalışır ve tüm build loglarına **Cloud Logging** üzerinden erişebilirsin.

**Artifact Registry**, fonksiyon kaynak kodundan inşa edilen image'ları saklamak için Cloud Run functions tarafından kullanılır. Artifact Registry, **container image'ları ve dil paketleri (language packages)** dahil olmak üzere, yazılım artifact'lerini **özel (private) depolarda** saklamak ve yönetmek için kullanılan bir servistir. Artifact Registry, build'lerinden gelen paketleri saklamak için **Cloud Build ile entegre çalışır.**

> **Analoji:** Bu pipeline'ı bir **matbaa** sürecine benzet. Sen (geliştirici), yayınlanacak metni (kaynak kodu) matbaaya (Cloud Storage bucket'ına) teslim edersin. Matbaanın baskı makinesi (Cloud Build), bu metni otomatik olarak fiziksel bir kitaba (container image) dönüştürür — sen bu dönüşüm sürecine hiç müdahale etmezsin, sadece sonucu beklersin. Basılan kitaplar, dağıtılmaya hazır şekilde bir depoda (Artifact Registry) raflara dizilir. Kitapçı (Cloud Run functions), bir müşteri (event ya da HTTP isteği) geldiğinde, depodaki doğru kitabı raftan alıp müşteriye sunar — kitabı her seferinde yeniden basmaz, zaten basılmış olanı kullanır.

> **Bu pipeline'ı BÖLÜM 8'deki dört dağıtım aracıyla nasıl ilişkilendirmelisin?** Console, Cloud Code ve gcloud CLI, kaynak kodunu **Cloud Storage'a ulaştırmanın** üç farklı ön kapısıdır (front door) — hangisini kullanırsan kullan, sonuç aynıdır: kaynak kod bir bucket'a gider. Cloud Build ise, bu noktadan sonra **her zaman aynı şekilde**, senin hangi aracı kullandığından bağımsız olarak devreye girer ve kaynak kodu bir container image'a dönüştürür. Yani "hangi araçla dağıtırım" sorusu ile "kaynak kodum nasıl çalışan bir container'a dönüşür" sorusu, **birbirinden ayrı iki katmandır** — biri senin ön yüzün, diğeri arka planda çalışan sabit motor.

> **Sınav tuzağı — build sürecine manuel müdahale gerektiğini düşünmek:** Ders özellikle "image'ın inşa edilme süreci tamamen otomatiktir ve senden hiçbir doğrudan girdi gerektirmez" diyor. Bir sınav sorusu "Cloud Run functions için container image'ı kim/ne inşa eder ve geliştiricinin bu sürece müdahale etmesi gerekir mi?" diye sorarsa, cevap **Cloud Build, tamamen otomatik olarak, geliştiricinin doğrudan müdahalesi olmadan** inşa eder — geliştiricinin tek sorumluluğu doğru kaynak kodu ve yapılandırmayı sağlamaktır; Dockerfile yazmak ya da build adımlarını elle tetiklemek **gerekmez.**

---

# Toplu Özet (Hızlı Tekrar)

**Modülün ana fikri:** Cloud Run functions, Google Cloud'un tam yönetilen, sunucusuz FaaS platformudur; Cloud Run'ın ölçeklenebilir container altyapısı üzerine kuruludur ve iki nesli vardır (Cloud Run functions / eski adıyla Cloud Functions 2nd gen, ve Cloud Run functions 1st generation / eski adıyla Cloud Functions 1st gen). Bu ilk modül, platformun ne olduğunu, nasıl çalıştığını ve nasıl dağıtıldığını uçtan uca tanıtıyor.

**Cloud Run functions nedir (BÖLÜM 1):** Fully managed serverless FaaS platformu; Cloud Run üzerine kurulu. Cloud Run functions, Eventarc/Pub/Sub ile tetiklenir ve gen1'e göre daha geniş trigger/yapılandırma desteği sunar; Cloud Run functions (1st generation) ise orijinal, daha kısıtlı nesildir. Bir cloud function, tek bir işlevi yerine getiren ve event'lerle tetiklenen basit koddur. İş akışı üç adımdır: geliştir → dağıt → trigger kur.

**Özellikler ve faydalar (BÖLÜM 2):** Özellikler — yerel geliştirme/test, service account ile kimlik doğrulama, HTTP+event-driven destek, Cloud SQL/Bigtable/Spanner/Firestore entegrasyonu, taşınabilirlik. Faydalar — serverless (Google altyapıyı yönetir), otomatik ölçekleme (birkaç çağrıdan milyonlara, ek çalışma gerekmeden), gözlemlenebilirlik (Cloud Observability entegrasyonu), pay-as-you-go fiyatlandırma (çağrı sayısı + compute time + outbound veri transferi).

**İki fonksiyon türü (BÖLÜM 3):** **HTTP functions**, atanmış bir URL üzerinden HTTP(S) isteğiyle tetiklenir; varsayılan olarak kimlik doğrulama gerektirir; Functions Framework'e kayıtlı bir handler ile implemente edilir; arka plan görevleri yanıt gönderilmeden önce sonlandırılmalıdır. **Event-driven functions**, Pub/Sub, Cloud Storage, Firestore, Firebase, Eventarc gibi kaynaklardan gelen event'lerle otomatik tetiklenir; CloudEvent function ya da Background function olarak implemente edilir.

**CloudEvent functions vs. Background functions (BÖLÜM 4):** CloudEvent functions, CloudEvents standardına dayanır, Functions Framework ile kayıtlıdır, Cloud Run functions'da tüm dillerde, gen1'de sadece .NET/Ruby/PHP'de kullanılır. Background functions, daha eski stildir, sadece gen1'de Node.js/Python/Go/Java'da kullanılır.

**Platform kapasitesi (BÖLÜM 5):** 32 GB RAM / 4 vCPU'ya kadar instance; tek instance'ta 1000'e kadar eşzamanlı istek (cold start'ları azaltır); HTTP functions için 60 dakika, event-driven functions için 10 dakika timeout; her deploy'da yeni revizyon (traffic splitting, rollback mümkün); Cloud Run/Kubernetes'e taşınabilirlik; Eventarc ile 90'dan fazla event kaynağı; standart CloudEvents formatı; console'da trigger yapılandırma/deployment izleme/test/metrik görüntüleme.

**Kullanım alanları (BÖLÜM 6):** Veri işleme (Cloud Storage event'leri ile doğrulama/dönüştürme/görsel-video işleme), webhook'lar ve API'ler (HTTP functions), mobil cihaz backend'i (Firebase/Firestore event'leri), IoT (Pub/Sub'a akan cihaz verisi).

**Dil runtime'ları, entry point, region (BÖLÜM 7):** Node.js `index.js` (ya da `package.json`'daki `main` alanı), Python `main.py`, Go kök dizindeki package; her birinin Functions Framework bağımlılığı ilgili bağımlılık dosyasında (package.json, requirements.txt, go.mod) tanımlıdır. Entry point, çağrıldığında çalışacak fonksiyon/sınıftır, deploy sırasında belirtilir. Region seçiminde birincil kriterler gecikme ve kullanılabilirlik; diğer GCP servislerinin konumu da göz önünde bulundurulmalı; iki neslin bölgesel kullanılabilirliği farklıdır.

**IAM ve dağıtım araçları (BÖLÜM 8):** Dağıtım yapan kullanıcı, Cloud Functions Developer rolüne ve runtime service account'u üzerinde Service Account User rolüne sahip olmalıdır. Dağıtım console, Cloud Build, Cloud Code ya da gcloud CLI ile yapılabilir. gcloud flag'leri: `--gen2`, `--region`, `--runtime`, `--source`, `--entry-point`, trigger flag'leri.

**Kaynak dağıtım seçenekleri ve build pipeline (BÖLÜM 9):** Kaynak kod yerel makineden (`--stage-bucket`, `.gcloudignore`), Cloud Storage'dan (zip dosyası, kök dizinde; izin sorumluluğu gen1'de deployer'da, Cloud Run functions'da service agent'ta) ya da bir kaynak deposundan (`revisions/`, `paths/`, `roles/source.reader`, GitHub/Bitbucket) dağıtılabilir; console'da satır içi editör de mevcuttur. Dağıtılan kaynak kod bir Cloud Storage bucket'ında saklanır; Cloud Build bunu otomatik olarak bir container image'a inşa eder ve Artifact Registry'ye push eder; Cloud Run functions bu image'ı kullanarak fonksiyonu çalıştırır. Süreç tamamen otomatiktir; Cloud Build API etkin olmalıdır; build logları Cloud Logging'de görülebilir.

---

# En Kritik Ayrımlar / Hızlı Tekrar (Cebinde Taşı)

- **İsim eşleşmesi:** Cloud Run functions = eski adıyla Cloud Functions (2nd generation) — Cloud Run üzerinde servis olarak dağıtılır, Eventarc/Pub/Sub ile tetiklenir. Cloud Run functions (1st generation) = eski adıyla Cloud Functions (1st generation) — orijinal, sınırlı trigger/yapılandırma.
- **HTTP functions varsayılan davranışı:** Gelen istekler **varsayılan olarak kimlik doğrulama gerektirir**; kimliği doğrulanmamış isteklere izin vermek istersen, bunu **dağıtım sırasında** açıkça seçmelisin.
- **CloudEvent functions vs. Background functions — iki eksenli kural:** Cloud Run functions → her zaman CloudEvent functions (tüm diller). Cloud Run functions (1st generation) → dile göre değişir: Node.js/Python/Go/Java'da Background functions, .NET/Ruby/PHP'de CloudEvent functions.
- **İki farklı timeout değeri:** HTTP functions = 60 dakikaya kadar. Event-driven functions = 10 dakikaya kadar. Bunları birbirine karıştırma.
- **1000 eşzamanlı istek/instance:** Cold start'ları azaltmanın ve ölçeklenirken gecikmeyi iyileştirmenin temel mekanizması.
- **Entry point ≠ dosya konumu:** Dosya konumu (index.js/main.py/Go package) "kaynak kod nerede" sorusunu, entry point ise "hangi fonksiyon/sınıf çalıştırılacak" sorusunu cevaplar — ikisi ayrı kavramlardır ve entry point deploy sırasında ayrıca belirtilir.
- **Region seçiminde tek kriter kullanıcı yakınlığı değildir:** Gecikme ve kullanılabilirliğe ek olarak, uygulamanın kullandığı diğer GCP servislerinin konumu da dikkate alınmalıdır — aksi halde hem gecikme hem fiyatlandırma olumsuz etkilenir.
- **İki ayrı IAM rolü gerekir:** Cloud Functions Developer (dağıtma izni) + Service Account User (runtime service account'u kullanma izni). Biri diğerinin yerine geçmez.
- **Cloud Storage'dan dağıtımda izin sorumluluğu nesle göre değişir:** Cloud Run functions (1st generation)'da deployer'ın, Cloud Run functions'da ise **Cloud Run functions service agent**'ının bucket'tan okuma izni olmalıdır.
- **Build süreci tamamen otomatiktir:** Kaynak kod → Cloud Storage → Cloud Build (otomatik container image inşası) → Artifact Registry (image saklama) → Cloud Run functions (image'ı çalıştırır). Geliştiricinin bu sürece manuel müdahalesi gerekmez; sadece Cloud Build API'sinin projede etkin olması gerekir.
- **Dört dağıtım aracı, tek build motoru:** Console, Cloud Code, gcloud CLI ve (dolaylı olarak) Cloud Build tetikleyicileri, kaynak kodu Cloud Storage'a ulaştırmanın farklı yollarıdır — ama hangisini kullanırsan kullan, arkadaki Cloud Build → Artifact Registry pipeline'ı **her zaman aynıdır.**

---

> **Kapanış:** Bu modül, Cloud Run functions'ı sıfırdan tanıttı: ne olduğunu (tam yönetilen serverless FaaS platformu, Cloud Run üzerine kurulu, iki nesli olan), nasıl tetiklendiğini (HTTP functions ve event-driven functions, CloudEvent functions ile Background functions arasındaki nesil/dil ayrımı), ne kadar güçlü olduğunu (32 GB RAM'e, 1000 eşzamanlı isteğe, 90'dan fazla event kaynağına kadar), nerede kullanılacağını (veri işleme, webhook/API, mobil/Firebase backend, IoT) ve son olarak kaynak kodunun nasıl gerçek, çalışan bir container'a dönüştüğünü (yerel makine/Cloud Storage/kaynak deposu → Cloud Build → Artifact Registry → Cloud Run functions) öğretti. Bu doküman, kursun şu anda mevcut olan tek modülünü kapsıyor; kursun ilerleyen modüllerinin transkriptleri eklendikçe, bu handbook'a yeni numaralı modüller olarak eklenmeye devam edecek.
