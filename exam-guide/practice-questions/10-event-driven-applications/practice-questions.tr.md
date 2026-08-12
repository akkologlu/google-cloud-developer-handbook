# Modül 10 — Event-Driven Applications: Pratik Sorular

Bu soru seti şunları kapsıyor: bir event'in ne olduğu, microservices arasındaki point-to-point iletişimin neden sorun haline geldiği, bir event intermediary'nin producer'ları consumer'lardan nasıl decouple ettiği, ve event-driven architecture'ın üç faydası: merkezi auditing/access control, decoupling ve asenkron işlem yoluyla resilience (push vs. polling dahil). Bu modül, "Service Orchestration and Choreography on Google Cloud" kursunun 2. Modülü'dür ve doğrudan 1. Modül'ün ([Microservices'e Giriş](../09-introduction-to-microservices/practice-questions.tr.md)) üzerine inşa edilir.

Sorular, insanların gerçekten takıldığı ayrımlara ağırlık veriyor: bir şeyi gerçekten "event" yapan nedir, consume edilmemiş bir event'in neden otomatik olarak bir bug olmadığı, bir event intermediary'nin bir Enterprise Service Bus (ESB)'den nasıl farklı olduğu, ve asenkron işlemin başarısızlık davranışını neden değiştirdiği.

Önce tüm soruları cevaplamayı dene, ardından cevaplarını aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle karşılaştır.

---

## Sorular

**1.** Bir geliştirici, bir alandaki bir yazım hatasını "düzeltmek" için yeni bir event yaymak yerine, event log'undaki mevcut bir event kaydını düzenlemeyi öneriyor. Modül bu konuda ne diyor ve neden?

A. Event'ler genelde immutable (değişmez) gerçekler olarak ele alınır — değiştirilmemesi ya da silinmemesi gereken bir olayın tarihsel kaydıdır. Düzeltme, eskisini düzenlemek yerine yeni bir event olarak ifade edilmelidir.
B. Bu sorun değil — değişiklik dokümante edildiği sürece event'ler her zaman yerinde düzenlenebilir.
C. Event'ler düzenlenebilir, ama yalnızca orijinal producer servis tarafından.
D. Event'ler düzenlenemez çünkü event intermediary, network katmanında write-once storage'ı zorunlu kılar.

**2.** Bir servis, sepete her ürün eklendiğinde bir event yayıyor, ama şu anda hiçbir servis o event'i consume etmiyor. Bir mühendis bunun kesinlikle bir bug olması gerektiğini savunuyor — "kimsenin dinlemediği bir event neden yayınlanır ki?" Bu gerçekten bir bug mu?

A. Evet — her event'in üretildiği anda en az bir aktif consumer'ı olmalıdır, aksi halde producer onu yaymamalıdır.
B. Evet, ama yalnızca event'leri saklamak pahalı olduğu için, consume edilmeyen event'ler kaynak israf eder.
C. Hayır — bir event, hiç consume edilmese bile üretilebilir; birçok producer, event'lerini bir şeyin consume edip etmediğini bilmez, bilmesine de gerek yoktur.
D. Hayır, ama yalnızca event intermediary sıfır consumer'lı event'leri sessizce düşürecek şekilde yapılandırılmışsa.

**3.** Bir event'in üretilmesinden üç ay sonra, yeni eklenen bir analytics servisi onu ilk kez işlemek istiyor, ve ayrı, mevcut bir servis de handler'ındaki bir bug'ı düzelttikten sonra onu yeniden işlemek istiyor. Modülde tanımlanan mimari bunu destekliyor mu?

A. Hayır — bir event yalnızca bir kez, onu ilk okuyan servis tarafından consume edilebilir.
B. Evet — bir event süresiz olarak persist edilebilir ve gerektiği kadar çok kez, birden fazla servis tarafından paralel olarak da dahil, consume edilebilir.
C. Hayır — event'ler üretildikten kısa süre sonra süresi dolar ve temizlenir, bu yüzden eski event'leri yeniden işlemek mümkün değildir.
D. Evet, ama yalnızca orijinal consumer bir event'i yeniden okuyabilir; yeni consumer'lar kendilerinden önce üretilen event'leri okuyamaz.

**4.** Doğrudan, point-to-point çağrılar kullanan bir microservices sisteminde, bir takım Service A'nın Service B, C ve D'yi doğrudan nasıl çağıracağını bilmesi gerektiğini ve yeni bir downstream servis eklemenin Service A'nın kodunu değiştirmek anlamına geldiğini fark ediyor. Modül bunun hangi sorunu gösterdiğini söylüyor ve hangi mimari değişiklik bunu çözer?

A. Bu beklenen bir durumdur ve gerçek bir dezavantajı yoktur; point-to-point çağrılar her zaman herhangi bir alternatife tercih edilir.
B. Bu, otomatik olarak coupling'i azaltan daha fazla microservice eklenerek çözülür.
C. Bu, Service B, C ve D'nin tekrar tek bir monolith'e birleştirilmesiyle çözülür.
D. Bu, point-to-point iletişimin "örümcek ağı" coupling sorunudur; servisler arasına bir event intermediary eklemek, producer'ların event'lerini kimin consume ettiğini bilmesine gerek kalmamasını sağlar.

**5.** Bir servis event producer olarak davranıyor. Modüle göre, event'lerini consume edebilecek servisler hakkında ne bilmesi gerekir?

A. Producer'ın event'lerini consume eden servisler hakkında hiçbir şey bilmesine gerek yoktur — yalnızca event'in formatını bilmesi gerekir.
B. Önceden her consumer'ın network adresini ve API contract'ını bilmesi gerekir.
C. İstekleri doğru şekilde dağıtabilmek için kaç consumer olduğunu bilmesi gerekir.
D. Teslimatı doğru sıralamak için hangi consumer'ın event'i ilk işleyeceğini bilmesi gerekir.

**6.** Genç bir mühendis, bir event intermediary'nin "aslında farklı isimli bir ESB" olduğunu söylüyor, çünkü ikisi de servisler arasında durur ve mesaj teslimatını yönetir. Bu doğru bir karşılaştırma mı?

A. Evet — modül bunları aynı mimari rol için tamamen birbirinin yerine geçen terimler olarak ele alır.
B. Hayır — bir event intermediary mesajların senkron olmasını gerektirir, ESB ise yalnızca asenkron mesajlaşmayı destekler.
C. Hayır — ESB, her entegrasyon değişikliğinin kendisinden ve onu yöneten takımdan geçmesi gerektiği için bir darboğaza dönüşen merkezi bir routing/transformation katmanıydı; event intermediary ise özellikle producer'ları consumer'lardan, bu tür merkezi bir değişiklik darboğazını yeniden yaratmadan ayırmak için var olur.
D. Evet, ama yalnızca Google Cloud implementasyonlarında; diğer bulutlarda yapısal olarak farklıdırlar.

**7.** Bir compliance görevlisi, yaklaşan bir denetim için dağıtık bir uygulamada yapılan her state değişikliğinin zamanlanmış, sıralı bir kaydına ihtiyaç duyuyor. Event-driven architecture'ın hangi özelliği doğrudan bu ihtiyacı destekler?

A. Push-based messaging, çünkü event'leri polling'den daha hızlı teslim eder.
B. Immutable event'lerin bir logu, uygulamanın state'indeki her değişikliğin zamanlanmış, sıralı bir kaydını verir ve auditing için kullanılabilir.
C. Consume edildikten sonra event'lerin silinebilmesi, audit log'unu küçük tutar.
D. Senkron request/response çağrılarının kullanımı, sıralamayı event'lerden daha iyi garanti eder.

**8.** Bir güvenlik takımı, her microservice'te erişim kontrollerini yeniden implemente etmek yerine, dağıtık, event-based bir uygulama için authentication ve authorization'ı tek bir yerden zorunlu kılmak istiyor. Modül bunu neyin mümkün kıldığını söylüyor?

A. Hiçbir şey — access control her consumer servisinde bağımsız olarak implemente edilmelidir.
B. Yalnızca event producer access control'ü zorunlu kılabilir; consumer'ların event'leri kimin okuduğunu kısıtlamasının bir yolu yoktur.
C. Event-driven sistemlerde access control gereksizdir çünkü event'ler immutable'dır.
D. Merkezi bir event service, event'lerin kendisinden geçtiği noktada authentication ve authorization'ı zorunlu kılabilir, bu da event-based servislere ve verilere erişimi tek bir yerden kontrol etmeni sağlar.

**9.** Bir takım, mevcut order-processing servislerinde hiçbir kod değişikliği yapmadan, "order placed" event'lerini consume eden yepyeni bir fraud-detection servisi eklemek istiyor. Modülün event-driven architecture tanımı bunu destekliyor mu, neden?

A. Evet — çünkü producer'lar ve consumer'lar decoupled'dır ve yalnızca bir event'in formatı üzerinde anlaşmaları gerekir, mevcut hiçbir servisi değiştirmeden yeni bir event consumer eklenebilir.
B. Hayır — herhangi bir yeni consumer, producer'ın alıcı listesine yeni servisi eklemek için güncellenmesini gerektirir.
C. Hayır — yeni bir consumer eklemek her zaman event intermediary'nin ve her producer'ın yeniden deploy edilmesini gerektirir.
D. Evet, ama yalnızca yeni consumer producer ile aynı platformda deploy edilirse.

**10.** Tamamen senkron request/response çağrıları üzerine kurulu bir sistemde, Service X, Service Y'yi çağırıyor, o da Service Z'yi çağırıyor. Service Z sağlıksız hale geliyor. Modül bunun sonucunda ne olduğunu söylüyor, ve event-driven bir tasarım bu sonucu nasıl değiştirirdi?

A. Hiçbir şey değişmez — senkron ve event-driven mimariler aynı şekilde başarısız olur.
B. Senkron zincir aslında daha resilient'tır çünkü başarısızlıklar anında tespit edilir.
C. Senkron zincirde, bir servisin sağlığı çağırdığı her servisin sağlığından etkilenir, bu yüzden Z'nin başarısızlığı Y üzerinden X'e kadar cascade edebilir; event-driven bir tasarımda ise sağlıksız servise yönelik event'ler, o servis geri geldiğinde replay edilebilir ya da yeniden teslim edilebilir, tüm zinciri anında başarısız kılmak yerine.
D. Event-driven architecture, Service Z'nin başından beri sağlıksız hale gelmesini engeller.

**11.** Bir kesintiden sonra, bir consumer servisi tekrar online oluyor ve devre dışıyken kaçırdığı event'leri yakalaması gerekiyor. Event-driven architecture'ın hangi yeteneği bu recovery'i doğrudan mümkün kılar?

A. Bu mümkün değildir — bir consumer devredeyken üretilen tüm event'ler kalıcı olarak kaybolur.
B. Sağlıksız bir servise gönderilen event'ler, o servis geri geldiğinde replay edilebilir ya da yeniden teslim edilebilir, çünkü event'ler tek bir teslimat denemesinden sonra atılmak yerine persist edilir.
C. Producer, consumer'ın geri geldiğini fark ettiğinde her event'i doğrudan API çağrısıyla manuel olarak yeniden göndermelidir.
D. Recovery yalnızca consumer event yerine senkron request/response çağrıları kullanıyorsa mümkündür.

**12.** Bir takım, event consumer'ını her 500 milisaniyede bir event kaynağını çağırıp "benim için yeni bir şey var mı?" diye sorarak implemente ediyor. Bu desenin adı nedir, dezavantajı nedir, ve modül bunun yerine neyi önerir?

A. Bu push-based messaging'dir ve implemente etmesi basit olduğu için önerilen yaklaşımdır.
B. Buna replay denir ve dezavantajı, yalnızca kesinti sonrası consumer recovery için kullanılabilmesi, normal operasyon için değil.
C. Bu event intermediary desenidir ve alternatiflere kıyasla gerçek bir dezavantajı yoktur.
D. Bu polling'dir; genelde network I/O'yu artırır ve yeni işin işlenmesinden önce gereksiz gecikme ekler. Consumer'ların consume edilecek bir şey olduğunda otomatik olarak bilgilendirildiği push-based messaging tercih edilir.

**13.** Push-based messaging'de, bir consumer genelde işlenecek yeni bir event olduğunu nasıl öğrenir?

A. Consumer, sormak zorunda kalmak yerine, consume edilecek bir event olduğunda intermediary tarafından otomatik olarak bilgilendirilir.
B. Consumer, yeni veri bulana kadar event intermediary'yi sabit bir programa göre tekrar tekrar sorgular.
C. Consumer'ın bilmesinin bir yolu yoktur; bir insan operatörün işlemeyi manuel olarak tetiklemesi gerekir.
D. Producer, event intermediary'yi tamamen atlayarak consumer'ı doğrudan ve senkron olarak çağırır.

**14.** Bir takım, producer'ları ve consumer'ları bir event intermediary ile decouple etmenin, producer'ların ve consumer'ların hiçbir konuda anlaşmasına gerek olmadığı anlamına gelmesi gerektiğini savunuyor. Bu doğru mu?

A. Evet — tam decoupling, producer ile consumer arasında sıfır paylaşılan contract anlamına gelir.
B. Hayır — producer'lar ve consumer'lar aynı programlama dilini ve deployment platformunu paylaşmalıdır.
C. Hayır — bir producer ya da consumer'ın yalnızca belirli bir event'in formatını bilmesi gerekir; hiçbir taraf diğerinin kimliğini ya da implementasyonunu bilmek zorunda olmasa bile, bu paylaşılan format anlayışı hâlâ gereklidir.
D. Evet, ama yalnızca auditing ile ilgili event'ler için; fonksiyonel event'ler hâlâ doğrudan bir API contract'ı gerektirir.

**15.** Modülün genel argümanını özetlersek, event-driven architecture neden microservices architecture ile doğal olarak uyumludur (bir monolith'le değil, örneğin)?

A. Uyumlu değildir — event-driven architecture özellikle microservices'te caydırılır ve yalnızca bir monolith içinde faydalıdır.
B. Çünkü microservices zaten ayrı deploy edilebilir birimler olarak network üzerinden iletişim kurar, ve bunun yarattığı point-to-point coupling sorunu, tam olarak bir event intermediary'nin ortadan kaldırmak için tasarlandığı şeydir — decoupled, resilient, bağımsız olarak evrilebilen servisleri mümkün kılarak.
C. Çünkü event'ler, microservices'in kendi veritabanlarına sahip olma ihtiyacını ortadan kaldırır.
D. Çünkü event-driven architecture, tüm servislerin aynı programlama dilinde yazılmasını gerektirir, microservices de bunu zaten varsayar.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Modül, event'lerin genelde immutable gerçekler olarak ele alındığını açıkça belirtir — değiştirilmemesi ya da silinmemesi gereken bir olayın tarihsel kaydı. Bir "düzeltme", mevcut kaydı düzenlemek yerine yeni bir event olarak ifade edilmelidir; geçmişi yerinde değiştirmek, event'leri kullanışlı yapan audit-log özelliğini bozar.

**2. Doğru cevap: C.**
Modül, bir event'in hiç consume edilmese bile üretilebileceğini ve event üreten birçok uygulamanın bu event'lerin hiç consume edilip edilmediğini bilmediğini doğrudan belirtir. "Şu anda consumer yok" durumunu doğası gereği bir bug olarak ele almak, producer'ların ve consumer'ların kasıtlı olarak decouple edildiğini yanlış anlamaktır — bir consumer henüz var olmayabilir.

**3. Doğru cevap: B.**
Event'ler süresiz olarak persist edilebilir ve gerektiği kadar çok kez, birden fazla servis tarafından paralel olarak da dahil, consume edilebilir — bu tam olarak yeni eklenen bir consumer'ın geçmiş event'leri işlemesine ve mevcut bir consumer'ın bir bug düzeltmesinden sonra yeniden işlemesine izin veren şeydir.

**4. Doğru cevap: D.**
Bu, modülde tanımlanan point-to-point iletişimin "örümcek ağı"dır: her servis, her downstream servise nasıl ulaşacağını bilmelidir, bu da coupling yaratır. Bir event intermediary eklemek, bir producer'ın consumer'ları hakkında bir şey bilme ihtiyacını ortadan kaldırır ve bu doğrudan bağımlılığı kırar.

**5. Doğru cevap: A.**
Modül açıkça, bir producer'ın event'lerini consume eden servisler hakkında hiçbir şey bilmesine gerek olmadığını belirtir — producer ile consumer arasındaki tek paylaşılan gereksinim, event'in formatı üzerinde anlaşmaktır.

**6. Doğru cevap: C.**
Bir event intermediary ve bir ESB aynı şey değildir, ikisi de servisler "arasında" dursa bile. Modülün daha geniş anlatısı (SOA tartışmasından devralınan), ESB'yi, her entegrasyon değişikliğinin kendisinden ve onu yöneten takımdan geçmesi gerektiği için bir darboğaza dönüşen merkezi bir routing/transformation katmanı olarak çerçeveler. Bir event intermediary ise özellikle producer'ları consumer'lardan, bu tür merkezi bir darboğazı yeniden yaratmadan ayırmak için var olur.

**7. Doğru cevap: B.**
Modül, auditing'i doğrudan immutability özelliğine bağlar: immutable event'lerin bir logu, bir uygulamanın state'indeki her değişikliğin zamanlanmış, sıralı bir kaydını verir — bir denetimin tam olarak ihtiyacı olan şey budur. Consume edildikten sonra event'leri silmek (C) bu özelliği yok eder, senkron çağrıların (D) ise denetlenebilir bir geçmiş sağlamakla bir ilgisi yoktur.

**8. Doğru cevap: D.**
Modül, merkezi bir event service'in, event service'in kendisinde authentication ve authorization'ı zorunlu kılarak servislere ve veriye erişimi kontrol etmeye yardımcı olabileceğini belirtir — bu, dağıtık, event-based bir uygulama genelinde access control'ü her yerde yeniden implemente etmek yerine tek bir yerden zorunlu kılmanı sağlar.

**9. Doğru cevap: A.**
Producer'lar ve consumer'lar decoupled olduğu ve yalnızca bir event'in formatını paylaşmaları gerektiği için, yeni bir consumer (fraud-detection servisi), yalnızca ilgili event stream'ini okumaya başlayarak eklenebilir — mevcut order-processing producer'larında hiçbir değişiklik gerekmez. Bu, decoupling'in ekstra bir faydası olarak açıkça belirtilir.

**10. Doğru cevap: C.**
Senkron bir zincirde, bir servisin sağlığı, doğrudan ya da dolaylı olarak çağırdığı her şeyin sağlığına bağlıdır, bu yüzden Z'deki bir başarısızlık Y üzerinden X'e kadar cascade edebilir. Event-driven bir tasarımda ise event'ler bir yanıt beklemeden asenkron olarak üretilir ve sağlıksız bir servise yönelik event'ler, o servis geri geldiğinde replay edilebilir ya da yeniden teslim edilebilir — bu tam olarak modülün tanımladığı resilience faydasıdır.

**11. Doğru cevap: B.**
Event'ler persist edildiği için (yalnızca bir kez teslim edilip atılmadığı için), bir servis sağlıksızken ona gönderilen event'ler, o servis tekrar online olduğunda replay edilebilir ya da yeniden teslim edilebilir — bu, modülün geçici bir servis kesintisini atlatmak için adlandırdığı spesifik mekanizmadır.

**12. Doğru cevap: D.**
Sürekli "yeni bir şey var mı?" diye sormak polling modelidir, ve modül bunun genelde network I/O'yu artırdığını ve işlemede gereksiz gecikme yarattığını belirtir. Push-based messaging — consumer'ın otomatik olarak bilgilendirildiği yaklaşım — modülde tanımlanan tercih edilen alternatiftir.

**13. Doğru cevap: A.**
Push-based messaging, consumer'ların consume edilecek event'ler olduğunda otomatik olarak bilgilendirilmesi anlamına gelir ve event'ler consumer'lara verimli biçimde route edilir — consumer'ın tekrar tekrar sormasına gerek yoktur, bu da push'u polling'den ayıran tam olarak budur.

**14. Doğru cevap: C.**
Decoupling, producer'ların ve consumer'ların birbirinin kimliğini, implementasyonunu ya da konumunu bilme ihtiyacını ortadan kaldırır, ama tüm paylaşılan bağlamı ortadan kaldırmaz: modül, bir producer ya da consumer'ın yalnızca belirli bir event'in formatını bilmesi gerektiğini açıkça belirtir. Hâlâ üzerinde anlaşılması gereken tek şey bu paylaşılan formattır.

**15. Doğru cevap: B.**
Modül, event-driven architecture'ı, microservices'in kendisinin yarattığı bir soruna doğrudan bir cevap olarak çerçeveler — birçok bağımsız deploy edilen servis arasındaki point-to-point iletişim, yönetilemez bir örümcek ağına dönüşür. Bir event intermediary, tam olarak bu microservices'e özgü coupling sorununun çözümüdür, bu yüzden iki desen ilgisiz fikirler olarak değil, birlikte sunulur.
