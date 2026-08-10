# Modül 11 — Choreography and Orchestration: Pratik Sorular

Bu soru seti şunları kapsıyor: service choreography vs. service orchestration, Pub/Sub, Eventarc ve CloudEvents formatı, Workflows, pratikte choreography ile orchestration arasında seçim yapmak, Cloud Tasks vs. Pub/Sub, ve Cloud Scheduler. Bu modül, "Service Orchestration and Choreography on Google Cloud" kursunun 3. Modülü'dür ve somut Google Cloud ürünlerinden bahseden ilk modüldür; 1. Modül'ün ([Microservices'e Giriş](../09-introduction-to-microservices/practice-questions.tr.md)) ve 2. Modül'ün ([Event-Driven Applications](../10-event-driven-applications/practice-questions.tr.md)) üzerine inşa edilir.

Sorular, insanların gerçekten takıldığı ayrımlara ağırlık veriyor: orchestration'ın neden kesin olarak "daha güvenli" bir varsayılan olmadığı, Eventarc'ın neden bir Pub/Sub rakibi olmadığı, Cloud Tasks ve Pub/Sub'ın neden birbirinin yerine geçemeyeceği, ve Cloud Scheduler'daki UTC/daylight-saving tuzağı.

Önce tüm soruları cevaplamayı dene, ardından cevaplarını aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle karşılaştır.

---

## Sorular

**1.** Bir takım, her microservice'in event'leri dinlediği ve bir sonraki adımda ne yapacağına bağımsız olarak karar verdiği bir sipariş işleme akışı implemente ediyor — genel sürecin tanımına sahip tek bir servis yok. Bu hangi koordinasyon deseni ve tanımlayıcı özelliği nedir?

A. Service orchestration; tanımlayıcı özellik, her etkileşimi kontrol eden merkezi bir orchestrator'dır.
B. Service choreography; tanımlayıcı özellik, her servisin bağımsız çalışması ve genel süreç için merkezi bir source of truth olmamasıdır.
C. Service choreography; tanımlayıcı özellik, tüm servislerin source of truth olarak tek bir veritabanını paylaşmasıdır.
D. Service orchestration; tanımlayıcı özellik, servislerin birbirinin internal implementasyonuna sıkı bağlı (tightly coupled) olmasıdır.

**2.** Merkezi, Workflows tabanlı bir süreç bir iş sürecinin her adımını koordine ediyor, ve takım bu merkezi süreç çökerse koordine edilen işlerin hiçbirinin ilerleyemeyeceğini fark ediyor. Bu hangi kavramı gösteriyor, ve bu orchestration'ı geçerli bir desen olmaktan çıkarır mı?

A. Bu, bir single point of failure'ı gösteriyor; bu, orchestration'ın doğasında olan bir trade-off'tur, diskalifiye edici bir kusur değil — orchestration, merkezi olarak yönetilen karmaşık süreçler için hâlâ güçlü bir desendir.
B. Bu, decentralized control'ü gösteriyor, yani takım yanlışlıkla orchestration yerine choreography implemente etmiş.
C. Bu, spesifik olarak Workflows'a özgü bir bug'dır; diğer orchestration araçlarında bu özellik yoktur.
D. Bu, orchestration'ı tamamen diskalifiye eder — modül, production sistemleri için merkezi bir orchestrator'ın asla kullanılmamasını önerir.

**3.** Bir Pub/Sub subscriber bir mesajı işler ama onu acknowledge edemeden çöker. Bu mesaja ne olur, ve bu hangi teslimat garantisini gösterir?

A. Mesaj kalıcı olarak kaybolur, çünkü Pub/Sub acknowledgment durumunu takip etmez.
B. Mesaj, subscriber'ın kuyruğunda kalır (çünkü hiç acknowledge edilmedi) ve yeniden teslim edilecektir — "at-least-once delivery"nin anlamı tam olarak budur.
C. Mesaj, yeniden işlemeyi önlemek için otomatik olarak farklı bir topic'e teslim edilir.
D. Pub/Sub exactly-once delivery garanti eder, bu yüzden mesaj yeniden teslim edilmek yerine düşürülür.

**4.** Bir subscriber, kendisi kontrol etmek yerine yeni mesajların geldiği anda Pub/Sub'ın onları yapılandırılmış bir HTTP endpoint'ine otomatik olarak göndermesini istiyor. Bu hangi subscription türünü tanımlar?

A. Bir pull subscription.
B. Bir push subscription.
C. Bir polling subscription.
D. Bir choreographed subscription.

**5.** Bir GCP servisinin, event'leri Eventarc'a gönderme konusunda yerleşik, doğrudan desteği yok. Eventarc genelde bu servisten yine de event'leri nasıl yakalar, ve bu geliştiriciyi neyi yapmaktan kurtarır?

A. Eventarc, o servisten hiç event yakalayamaz; doğrudan destek zorunludur.
B. Eventarc, o servisten gelen Cloud Audit Logs kayıtlarını event üretmek için kullanabilir, bu da geliştiriciyi audit log'ları parse etme ya da event'leri poll etme kodu yazmaktan kurtarır.
C. Geliştirici, servis dikkate değer bir şey yaptığında her seferinde manuel olarak özel bir Pub/Sub mesajı publish etmelidir.
D. Eventarc, herhangi bir event yakalanmadan önce servisin Eventarc SDK'sı kullanılarak yeniden yazılmasını gerektirir.

**6.** Bir geliştirici, Eventarc kullanmanın, tıpkı Pub/Sub'ı doğrudan kullanmak gibi, uygulamasının Pub/Sub topic'lerini ve subscription'larını doğrudan yönetmesi gerektiği anlamına geldiğini iddia ediyor. Bu doğru mu?

A. Evet — Eventarc, ham Pub/Sub üzerinde uygulamanın yönetmesi gereken şey açısından gerçek bir fark olmayan, ince bir isimlendirme kuralından ibarettir.
B. Hayır — Eventarc, güvenilirlik ve observability için transport katmanı olarak Pub/Sub'ı kullanır, ama altındaki topic'leri ve subscription'ları otomatik olarak yaratır ve yönetir; uygulama yalnızca Eventarc'ın gönderdiği HTTP isteklerini kabul etmesi gerekir.
C. Hayır — Eventarc, Pub/Sub'ın yerini tamamen alır ve onu hiçbir zaman internal olarak kullanmaz.
D. Evet, ama yalnızca third-party event source'ları için; GCP-native source'lar Pub/Sub'ı tamamen atlar.

**7.** İki takım, farklı publisher'lar kullanarak event-driven servisler inşa ediyor ve her publisher tarihsel olarak kendi özel event formatını kullanmış. Eventarc'ın CloudEvents formatını kullanması burada neyi çözer, ve takımların yazdığı kod açısından bu neden önemli?

A. Hiçbir şeyi çözmez — CloudEvents, uygulama koduna hiçbir etkisi olmayan salt bir dokümantasyon kuralıdır.
B. Event verisini tanımlamak için ortak bir metadata formatı sağlar, bu sayede her iki takım da SDK'ları (Python, JavaScript, Java, Go, C#, Ruby, PHP ve daha fazlasında mevcut) ve event'in kaynağından ya da türünden bağımsız olarak aynı event-handling mantığını kullanabilir, her publisher'ın formatı için özel parsing kodu yazmak yerine.
C. Her iki publisher'ı da tam olarak aynı programlama dilini kullanmaya zorlar.
D. Event trigger'lara olan ihtiyacı ortadan kaldırır, çünkü CloudEvents mesajları hiçbir yapılandırılmış kural olmadan otomatik olarak route eder.

**8.** Bir workflow'un envanteri kontrol etmesi, stok durumuna göre dallanması, o dala bağlı olarak farklı servisleri çağırması, bir veritabanını güncellemesi ve son olarak müşteriyi e-postayla bilgilendirmesi gerekiyor — yol boyunca yavaş bir dış adımı beklemesi de mümkün. Bu tam olarak hangi Google Cloud ürünü için tasarlanmıştır, ve sürecin içindeki uzun gecikmeleri pratik hale getiren yetenek nedir?

A. Cloud Tasks, çünkü at-least-once delivery garanti eder.
B. Pub/Sub, çünkü publisher'ların subscriber'larının kim olduğunu bilmesine gerek yoktur.
C. Workflows, çünkü bir workflow state tutabilir, retry yapabilir, poll edebilir ya da bir yıla kadar bekleyebilir, bu da uzun süren, stateful iş süreçlerini pratik hale getirir.
D. Cloud Scheduler, çünkü job'ları tekrarlayan bir cron programına göre tetikleyebilir.

**9.** Bir takım aynı sipariş işleme mantığını iki kez implemente ediyor: bir kere choreography ile (her servis "sıradaki adım için hazırım" event'i üretiyor, stokta var/yok kararları o adımın sahibi olan servis tarafından yerel olarak veriliyor) ve bir kere Workflows tabanlı orchestration ile. Modül, choreography versiyonunun karşılaştığı ve orchestration versiyonunun kaçındığı spesifik zorluğu ne olarak söylüyor?

A. Choreography versiyonu Firestore'u kullanamaz, orchestration versiyonu kullanabilir.
B. Choreography versiyonu, event'ler her zaman network latency'si eklediği için runtime'da doğası gereği daha yavaştır, orchestrated çağrılar ise her zaman in-process'tir.
C. Choreography versiyonu, visibility, error handling ve retry'ları doğru yapmayı zorlaştırır — örneğin sistemi troubleshoot etmek ya da envanteri kilitlemek ile güncellemek arasında aborta olan bir süreci ele almak — Workflows ise mantığın tek bir yerde tanımlandığı, yerleşik retry ve error handling'e sahip her execution'ı ayrı ayrı takip eder.
D. Choreography versiyonu tüm servislerin tek bir takıma ait olmasını gerektirir, orchestration versiyonu gerektirmez.

**10.** Bir organizasyonun Order, Fulfillment ve Marketing servisleri, şirketin farklı bölümlerindeki farklı takımlar tarafından sahiplenilip bağımsız olarak yönetiliyor. Bir mimar, bu üç servis arasındaki tüm koordinasyonu, tek bir takıma ait, paylaşılan bir Workflows orchestrator'ının altına koymayı öneriyor. Modül bunun muhtemelen hangi soruna çarpacağı konusunda ne uyarıyor?

A. Hiçbiri — Workflows, herhangi bir sayıda bağımsız yönetilen takım arasında hiçbir koordinasyon maliyeti olmadan her zaman sorunsuz ölçeklenir.
B. Koordine edilen servisler farklı takımlar ya da organizasyonlar tarafından inşa edilip yönetildiğinde, merkezi bir orchestration sürecinin paylaşılan yönetimi zor olabilir — bu tam olarak choreography'nin decentralized control'ünün daha uygun olma eğiliminde olduğu senaryodur.
C. Workflows, IAM kısıtlamaları nedeniyle teknik olarak farklı bir takıma ait servisleri çağıramaz ve bunun etrafından dolaşmanın bir yolu yoktur.
D. Bu, modülün tartıştığı gerçek bir trade-off değildir; orchestration ve choreography bu senaryoda fonksiyonel olarak özdeş olarak sunulur.

**11.** Order, Fulfillment ve Marketing servislerinin her birinin kendi içinde Workflows kullanılarak implemente edildiği, Eventarc'ın ise bu servisler arasında event trigger'ları taşıdığı ve bir Cloud Storage bucket'ına düşen yeni dosyalara tepki verdiği bir tasarımda, hangi desen nerede kullanılıyor?

A. Her yerde saf orchestration — Eventarc, Workflows ile birlikte çalışan sadece başka bir orchestrator'dır.
B. Her yerde saf choreography — Workflows yalnızca loglama için kullanılıyor, herhangi bir süreci kontrol etmiyor.
C. Her servisin kendi süreci içinde orchestration (Workflows üzerinden) ile servisler arasında choreography'nin (Eventarc üzerinden) birleşimi — iki desenin pratikte birleştirilmesinin yaygın bir yolu.
D. İkiden fazla servis dahil olduğunda hiçbir desen geçerli olmaz.

**12.** Bir backend'in belirli, yavaş bir işlemi (örn. büyük bir rapor üretmek) bilinen bir worker servisine offload etmesi gerekiyor, bu dispatch'in tam olarak ne zaman olacağını kontrol etmek istiyor, ve worker başarısız olursa yapılandırılabilir bir gecikmeyle otomatik retry'lara ihtiyacı var. Hangi servis buna en uygun, ve en benzer alternatife göre temel ayırt edici özelliği nedir?

A. Pub/Sub, çünkü ilgilenen her subscriber'a at-least-once delivery garanti eder.
B. Cloud Tasks, çünkü explicit invocation kullanır — creator, task'ın execution'ı ve destination'ı üzerinde tam kontrolü elinde tutar — Pub/Sub'ın implicit invocation'ının aksine, orada publisher'ın hangi subscriber'ların mesajı alacağı üzerinde hiçbir kontrolü yoktur.
C. Eventarc, çünkü Cloud Audit Log tabanlı event'leri otomatik olarak tetikleyebilir.
D. Cloud Scheduler, çünkü cron job'ları her zaman yapılandırılabilir gecikmeyle retry'ları garanti eder.

**13.** Bir publisher, şu anda hangi servislerin (varsa) subscribe olduğunu bilmeden ya da umursamadan bir Pub/Sub topic'ine bir mesaj gönderiyor. Bu hangi invocation modelini tanımlar, ve Cloud Tasks'tan nasıl farklıdır?

A. Explicit invocation — Cloud Tasks'ın kullandığı ile aynı model.
B. Implicit invocation — publish etmek, hangi subscriber'lar varsa onların örtük olarak çalışmasına neden olur, hangi subscribe eden servislerin mesajı alacağı üzerinde hiçbir kontrol yoktur; Cloud Tasks ise bunun yerine explicit invocation kullanır, burada creator, belirli, bilinen bir endpoint'e bağlı bir kuyruğa bir task yerleştirir.
C. Scheduled invocation — davranış olarak bir Cloud Scheduler cron job'uyla özdeştir.
D. Choreographed invocation — Pub/Sub'ın gerçek teslimat modeliyle ilgisi olmayan bir kavram.

**14.** Bir takım, bir Cloud Scheduler job'unu varsayılanda bırakmak yerine `America/New_York` time zone'unda `15 0 * * *` cron string'i ile yapılandırıyor. Yılda iki kez, job ya çalışmıyor ya da aynı gün iki kez çalışıyor. Buna ne sebep oluyor, ve modül bunu önlemek için ne öneriyor?

A. Bu, önerilen bir çözümü olmayan bir Cloud Scheduler bug'ıdır.
B. `America/New_York` time zone'u daylight saving time uygular, bu da saatler ileri alındığında bir job'ın atlanmasına ya da saatler geri alındığında iki kez çalışmasına neden olabilir; modül, bunu önlemek için, aynı zamanda varsayılan olan UTC time zone'unu kullanmayı önerir.
C. Cron string'inin kendisi geçersizdir, bu da time zone'larla ilgisizdir.
D. Bu yalnızca push subscription'larda olur, Cloud Scheduler job'larında değil.

**15.** Cloud Scheduler tarafından kullanılan Unix cron formatında, bir job'ın hour alanı `*/2` olarak ayarlanmış. Bu ne anlama gelir, ve job'ı yalnızca Pazartesi günleri çalışacak şekilde kısıtlamak için hangi alanı değiştirirsin?

A. `*/2`, "her 2 saatte bir" anlamına gelir; Pazartesi'ye kısıtlamak, day-of-week alanının (beşinci alan, Pazartesi'nin 0=Pazar...6=Cumartesi şemasında 1 olarak temsil edildiği alan) değiştirilmesini gerektirir.
B. `*/2`, "yalnızca saat 2'de" anlamına gelir; Pazartesi'ye kısıtlamak month alanının değiştirilmesini gerektirir.
C. `*/2`, "ayda iki kez" anlamına gelir; Pazartesi'ye kısıtlamak minute alanının değiştirilmesini gerektirir.
D. `*/2` geçersiz bir syntax'tır; Cloud Scheduler adım (step) değerlerini desteklemez.

**16.** Bir çözüm mimarı, Pub/Sub, Eventarc, Workflows, Cloud Tasks ve Cloud Scheduler'ı Google Cloud'un "Application Integration" araç kutusu olarak özetliyor ve tam donanımlı bir microservices uygulamasının, yalnızca birini seçmek yerine makul olarak bunlardan birkaçını birlikte kullanabileceğini söylüyor. Bu, modülden doğru bir çıkarım mı?

A. Hayır — modül bu beş servisi birbirini dışlayan alternatifler olarak sunar; aynı uygulamada birden fazlasını kullanmak caydırılır.
B. Evet — modül, bu beş servisi tam donanımlı bir microservices uygulamasının birlikte kullanmaktan fayda görebileceği bir araç kutusu olarak açıkça çerçeveler (örn. bir servis içinde orchestration için Workflows, servisler arasında choreography için Eventarc, offload edilmiş async iş için Cloud Tasks, tekrarlayan job'lar için Cloud Scheduler, bunun bir kısmının altında Pub/Sub).
C. Hayır — yalnızca Pub/Sub ve Eventarc birlikte kullanılmak üzere tasarlanmıştır; Workflows, Cloud Tasks ve Cloud Scheduler yeni uygulamalar için tasarlanmamış legacy servislerdir.
D. Evet, ama yalnızca uygulama Pub/Sub'ı tamamen atlıyorsa, çünkü Eventarc onun yerini tamamen almak için tasarlanmıştır.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
Genel süreci sahiplenen tek bir servis olmadan dağıtık karar verme, tam olarak service choreography'dir, orchestration'ın merkezi kontrolcüsüyle tezat oluşturur. Bir veritabanını paylaşmak (C) choreography'yi tanımlayan şey değildir, ve tight coupling (D), modülün her iki desenin de gevşek bağlı servisler ürettiği noktasıyla çelişir.

**2. Doğru cevap: A.**
Modül bu trade-off'u açıkça adlandırır: choreography'deki tamamen dağıtık servislerin aksine, orchestration bir single point of failure'a sahiptir — orchestrator çalışamıyorsa, orchestrated süreçler çalışamaz. Bu, orchestration'dan vazgeçmek için bir neden değil, doğasında olan bir trade-off olarak sunulur; orchestration, merkezi olarak yönetilen karmaşık süreçler için hâlâ güçlü bir desendir.

**3. Doğru cevap: B.**
Bir mesaj, ancak acknowledge edildikten sonra bir subscriber'ın kuyruğundan kaldırılır; çökme öncesinde acknowledge edilmediği için kuyrukta kalır ve yeniden teslim edilecektir. Bu "silme başarılı işlemeyi takip eder" davranışı, tam olarak at-least-once delivery'yi garanti eden şeydir — bir mesaj birden fazla kez teslim edilebilir, ama sıfır kez teslim edilemez.

**4. Doğru cevap: B.**
Bir push subscription, Pub/Sub'ın mesajı yapılandırılmış bir endpoint'e otomatik olarak göndermesiyle tanımlanır; buna karşılık bir pull subscription'da subscriber yeni mesajlar için ara sıra kendisi poll eder.

**5. Doğru cevap: B.**
Doğrudan event-source desteği olmayan servisler için Eventarc, event üretmek üzere Cloud Audit Logs kayıtlarını sorunsuzca kullanabilir — geliştiricinin audit log'ları parse etme ya da event'leri poll etme kodu yazmasına gerek kalmaz; bu yetenek tam olarak bu manuel işi ortadan kaldırır.

**6. Doğru cevap: B.**
Eventarc, transport katmanı olarak gerçekten Pub/Sub'ı kullanır, ama ilgili topic'leri ve subscription'ları otomatik olarak yaratır ve yönetir — uygulamanın kendisinin yalnızca Eventarc'ın gönderdiği HTTP isteklerini kabul etmesi gerekir, Pub/Sub'a hiç doğrudan dokunmaz. Bu, Pub/Sub'ı tek başına kullanmaktan temel fark budur.

**7. Doğru cevap: B.**
CloudEvents, kaynaktan bağımsız olarak event verisini tanımlamak için ortak bir metadata formatı sağlar, ve SDK'ları (birçok popüler dil için mevcut) geliştiricilerin farklı event türleri ve kaynakları arasında aynı event-handling mantığını yeniden kullanmasına izin verir, her publisher için özel parsing yazmak yerine — bu, senaryoda tanımlanan "herkesin kendi formatı vardı" sorununu tam olarak çözer.

**8. Doğru cevap: C.**
Workflows, modülün stateful, otomatikleştirilmiş, potansiyel olarak uzun süren iş süreçleri için verdiği cevaptır; bir workflow'un state tutabilme, retry yapabilme, poll edebilme ya da bir yıla kadar bekleyebilme yeteneği, uzun gecikmeleri ve çok adımlı dallanan süreçleri özellikle pratik hale getiren şeydir — farklı sorunları çözen Cloud Tasks, Pub/Sub ya da Cloud Scheduler'ın aksine.

**9. Doğru cevap: C.**
Modül, choreography'li bir sipariş akışının visibility, error handling ve retry'ları doğru yapmayı zorlaştırdığını açıkça belirtir — event-based bir sistemi troubleshoot etmek ve akış ortasında aborta olan bir süreci ele almak dahil — Workflows tabanlı orchestration ise süreç mantığının tek bir yerde tanımlandığı her execution'ı ayrı ayrı takip eder, yerleşik retry ve error handling ile.

**10. Doğru cevap: B.**
Modül açıkça, altındaki servisler farklı takımlar ya da organizasyonlar tarafından inşa edilip yönetildiğinde, merkezi bir orchestration sürecinin paylaşılan yönetiminin zor olabileceği konusunda uyarır — bu decentralized-ownership senaryosu, tam olarak choreography'nin decentralized control'ünün daha uygun olma eğiliminde olduğu, orchestration ile etrafından dolaşılacak bir kusur olmadığı senaryodur.

**11. Doğru cevap: C.**
Bu, modülün kendi birleştirilmiş örneğidir: Order, Fulfillment ve Marketing'in her biri Workflows kullanılarak implemente edilir (her servisin içinde orchestration), Eventarc ise servisler arasında event trigger'ları taşır ve dosya upload'larına tepki verir (servisler arasındaki sınırlarda choreography) — iki desen aynı anda farklı kapsamlarda çalışır.

**12. Doğru cevap: B.**
Cloud Tasks, explicit invocation ile tanımlanır: task creator, execution ve destination üzerinde tam kontrolü elinde tutar, task'ı belirli bir endpoint'e bağlı bir kuyruğa yerleştirir ve isteğe bağlı olarak dispatch'i erteler — bu, Pub/Sub'ın implicit invocation'ının tam tersidir, orada publisher'ın bir mesaj üzerinde hangi subscriber'ların harekete geçeceği üzerinde hiçbir kontrolü yoktur. Retry denemeleri ve gecikme de bir Cloud Tasks kuyruğunda doğrudan yapılandırılabilir.

**13. Doğru cevap: B.**
Bu, tanımı gereği implicit invocation'dır: publisher'ın eylemi, o subscriber'ların kim olduğunu kontrol etmeden ya da mutlaka bilmeden, mevcut olan hangi subscriber varsa onun örtük olarak çalışmasına neden olur. Cloud Tasks ise bunun yerine explicit invocation kullanır, creator'a tam olarak hangi endpoint'in task'ı alacağı üzerinde doğrudan kontrol verir.

**14. Doğru cevap: B.**
Daylight saving time uygulayan time zone'lar, saatler ileri alındığında zamanlanmış bir job'ın atlanmasına ya da saatler geri alındığında iki kez çalışmasına neden olabilir, çünkü belirtilen yerel saat geçiş gününde ya hiç yoktur ya da iki kez oluşur. Modül, bu tür bir bug'dan kaçınmak için özellikle UTC'yi (aynı zamanda varsayılan) kullanmayı önerir.

**15. Doğru cevap: A.**
`*/N` olarak yazılan bir cron alanı, "bir aralığı takiben her N birimde bir" anlamına gelir — hour alanı için, `*/2` her iki saatte bir anlamına gelir. Execution'ı yalnızca haftanın bir gününe kısıtlamak, month ya da minute alanları üzerinden değil, beşinci (day-of-week) alan üzerinden yapılır.

**16. Doğru cevap: B.**
Modül, açıkça Pub/Sub, Eventarc, Workflows, Cloud Tasks ve Cloud Scheduler'ı birlikte bir "Application Integration" araç kutusu olarak adlandırarak kapanır ve tam donanımlı bir microservices uygulamasının bunlardan herhangi birinden ya da hepsinden fayda görebileceğini belirtir — bunları birbirini dışlayan seçenekler olarak sunmaz, ve Eventarc, Pub/Sub'ın yerini almak yerine açıkça onun üzerine kuruludur.
