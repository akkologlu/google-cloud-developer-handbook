# Modül 13 — Calling and Connecting Cloud Run Functions: Alıştırma Soruları

Bu set, iki trigger kategorisini (HTTP trigger'lar ve event trigger'lar) ve her bir spesifik trigger türünün — HTTP, Pub/Sub, Cloud Storage, Firestore ve Firebase — nasıl davrandığını, Eventarc'ın evrensel event-delivery mekanizması olarak rolünü, CloudEvent functions ile Background functions'ın event verisini nasıl farklı şekilde aldığını, Workflows'un Cloud Run functions'ı ve diğer servisleri nasıl orchestrate ettiğini (build süreci, workflow tanımları, sonuçları zincirleme, harici REST API'leri ve Cloud Run service'lerini çağırma), ve Serverless VPC Access'in function'ları yalnızca internal IP address'i olan kaynaklara nasıl bağladığını kapsar. Bu modül, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun, Modül 1'i (Modül 12) takip eden 2. Modülü'dür.

Sorular, insanların gerçekten takıldığı ayrımlara ağırlık verir: bir function'a tek bir trigger bağlamak ile bir event'i birden fazla function'a fan-out etmek arasındaki fark, bir workflow'dan çağrılan function'ların neden HTTP function olması gerektiği, Firestore trigger'larının yalnızca document seviyesindeki granülerliği, ve bir Serverless VPC Access connector'ının region'ının neden function'ın region'ıyla eşleşmesi gerektiği.

Önce tüm soruları yanıtlamayı deneyin, ardından yanıtlarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle karşılaştırın.

---

## Sorular

**1.** Bir function'ın, herhangi bir harici caller olmadan, bir Google Cloud projesi içinde belirli bir türde event gerçekleştiğinde çalışması gerekiyor. Hangi trigger kategorisi buna uyar, ve bu neyle ilişkilidir?

A. Bir event trigger, ki bu bir event-driven function'a karşılık gelir.
B. Bir event trigger, ki bu bir HTTP function'a karşılık gelir.
C. Bir HTTP trigger, ki bu bir event-driven function'a karşılık gelir.
D. Hiçbir kategori uygulanmaz; bir function'ı caller olmadan yalnızca Cloud Scheduler başlatabilir.

**2.** Bir takım, aynı Pub/Sub mesajı geldiğinde üç farklı function'ın hepsinin çalışmasını istiyor, ama bir geliştirici "bir function'ın yalnızca bir trigger'ı olabilir" dediği için bunun imkansız olduğunu iddia ediyor. Bu iddiayı değerlendirin.

A. İddia doğrudur, ve birden fazla function'ın tek bir event'e tepki vermesinin hiçbir yolu yoktur.
B. İddia doğrudur, ama yalnızca HTTP-triggered function'lar için, event-driven function'lar için değil.
C. İddia yanlıştır, çünkü tek bir function aynı anda birden fazla trigger'a bağlanabilir.
D. Tek bir function gerçekten de aynı anda yalnızca bir trigger'a bağlanabilir, ama bu fan-out'u engellemez — aynı trigger source ayarlarını paylaşan birden fazla function deploy edersiniz.

**3.** Bir geliştirici, kaynağın Pub/Sub, Cloud Storage ya da tamamen başka bir şey olmasından bağımsız olarak, event-driven bir Cloud Run function'ına event'leri gerçekte hangi servisin teslim ettiğini öğrenmek istiyor. Cevap nedir, ve yaklaşık kaç kaynağı destekler?

A. Tüm event-driven Cloud Run functions, event delivery için Eventarc kullanır, ki bu 90'dan fazla Google Cloud kaynağını destekler.
B. Her event kaynağının (Pub/Sub, Cloud Storage, Firestore) kendi ayrı, birbiriyle ilişkisiz delivery mekanizması vardır.
C. Kaynak ne olursa olsun tüm event'leri Cloud Tasks teslim eder.
D. Yalnızca Pub/Sub event teslim eder; Cloud Storage ve Firestore trigger'ları herhangi bir delivery mekanizmasından geçmez.

**4.** HTTP-triggered bir function'ın, kendisine doğrudan HTTP üzerinden tam CRUD tarzı işlemler yapan client'ları desteklemesi gerekiyor. Bir HTTP trigger hangi request method'larını destekler?

A. Yalnızca GET ve POST; PUT ve DELETE, HTTP trigger'lar tarafından desteklenmez.
B. Yalnızca POST, çünkü HTTP trigger'lar yalnızca webhook tarzı çağrılar için tasarlanmıştır.
C. GET, POST, PUT, DELETE ve OPTIONS.
D. HTTP trigger'lar herhangi bir spesifik method kısıtlaması ya da liste desteklemez — herhangi bir custom method adı kabul edilir.

**5.** Bir function'ın, belirli bir Pub/Sub topic'ine bir mesaj publish edildiğinde çalışması gerekiyor. Bir Pub/Sub trigger'ın çalışması için function'ın implementasyonu hakkında ne doğru olmalıdır, ve trigger arka planda nasıl implemente edilir?

A. Function herhangi bir şekilde implemente edilebilir; Pub/Sub trigger'lar event-driven bir implementasyon gerektirmez.
B. Function bir HTTP function olmalıdır, çünkü Pub/Sub trigger'lar HTTP trigger'ların özel bir durumudur.
C. Function topic'i manuel olarak poll etmelidir; Pub/Sub trigger'lar yalnızca bildirir, invoke etmez.
D. Function event-driven bir function olarak implemente edilmelidir, ve Cloud Run functions'ta Pub/Sub trigger'ları bir Eventarc trigger türü olarak implemente edilir.

**6.** Aynı Pub/Sub topic'i tarafından tetiklenen iki function var — biri CloudEvent function, diğeri Background function. Pub/Sub event verisi her birine nasıl ulaşır?

A. Her ikisi de veriyi aynı formatta alır; implementasyon stili payload'ı etkilemez.
B. CloudEvent function veriyi CloudEvents formatında alır; Background function ise `PubsubMessage` formatında alır.
C. CloudEvent function veriyi `PubsubMessage` formatında alır; Background function ise CloudEvents formatında alır.
D. Hiçbir function aslında Pub/Sub event verisini alamaz; yalnızca HTTP function'lar Pub/Sub mesajlarını işleyebilir.

**7.** Bir bucket üzerinde bir Cloud Storage trigger yapılandırılmış, ve onu işleyen function bir Background function olarak implemente edilmiş. Cloud Storage event verisi hangi formatta gelir?

A. Bir CloudEvent function ile aynı, CloudEvents formatında.
B. Herhangi bir yapılandırılmış format olmadan, ham bir dosya byte stream'i olarak.
C. `StorageObjectData` formatında.
D. Cloud Storage trigger'ları hiçbir koşulda Background functions ile uyumlu değildir.

**8.** Bir takım, bir document'in başka yerlerindeki değişikliklere tepki vermeden, Firestore document'i içindeki belirli bir field değiştiğinde bir function'ın tetiklenmesini istiyor. Bu bir Firestore trigger ile mümkün müdür?

A. Hayır — Firestore trigger'ları yalnızca document seviyesinde uygulanır; belirli bir document field'ına ya da collection'a özgü bir trigger oluşturmak mümkün değildir.
B. Evet — Firestore trigger'ları bir field-path filtresi kullanılarak tek bir field'a özgülenebilir.
C. Evet — Firestore trigger'ları, document seviyesinden daha ince granülerlikte olan tüm bir collection'a özgülenebilir.
D. Firestore trigger'ları event türü olarak "update"i desteklemez, yalnızca "create" ve "delete"i destekler.

**9.** Proje A'daki bir function'ın, Proje B'de bulunan bir Firestore veritabanındaki değişikliklere tepki vermesi bekleniyor. Bu yapılandırma açıklandığı gibi çalışır mı?

A. Evet, her iki proje de aynı organizasyona ait olduğu sürece.
B. Evet, cross-project Firestore trigger'ları hiçbir ek yapılandırma olmadan açıkça desteklenir.
C. Bu, event türüne bağlıdır; yalnızca "write" event'leri proje sınırlarını geçebilir.
D. Hayır — trigger'ın çalışması için Firestore'un function ile aynı Google Cloud projesinde olması gerekir.

**10.** Bir mobile backend takımı, Firebase Authentication event'leri tarafından tetiklenen bir function istiyor ve bunun hangi Cloud Run functions nesli kullanılırsa kullanılsın mevcut olup olmadığını bilmek istiyor. Modül ne diyor?

A. Firebase Authentication trigger'ları her iki neslde de aynı şekilde ve kısıtlama olmadan çalışır.
B. Firebase Authentication trigger'ları (Google Analytics for Firebase trigger'ları gibi) yalnızca Cloud Run functions (1st gen) için kullanılabilir.
C. Firebase Authentication trigger'ları yalnızca Cloud Run functions (2nd gen) için kullanılabilir, asla 1st gen için değil.
D. Firebase Authentication event'leri Cloud Run functions'ı hiç tetikleyemez; yalnızca Firestore ve Realtime Database tetikleyebilir.

**11.** Bir takım, aylarca sürebilecek bir süreç boyunca state tutan ve başarısız step'leri retry eden, çok adımlı bir business process'i orchestrate edecek merkezi bir platform istiyor. Modül neyi önerir, ve uzun süren süreçleri pratik hale getiren spesifik yetenek nedir?

A. Cloud Scheduler, çünkü birkaç dakikadan uzun süren herhangi bir süreç için tasarlanmıştır.
B. Workflows, bir yıla kadar state tutabilen, retry yapabilen, poll edebilen ya da bekleyebilen, tamamen yönetilen bir serverless orchestration platformu.
C. Pub/Sub, çünkü uzun bir retention süresi yapılandırılarak mesajlar süresiz olarak saklanabilir.
D. Eventarc, çünkü hiçbir orchestration katmanına ihtiyaç duymadan trigger'ları zincirleyebilir.

**12.** İki Cloud Run function'ını step olarak çağıracak bir workflow inşa ederken, bir geliştirici bu function'ların hangi türde olması gerektiğini bilmek istiyor. Ne gereklidir, ve neden?

A. Function'lar Background functions olmalıdır, çünkü bir workflow step'ine yalnızca event-driven function'lar gömülebilir.
B. Hiçbir gereksinim yoktur; herhangi bir function türü bir workflow step'i olarak birbirinin yerine kullanılabilir.
C. Function'lar HTTP function olmalıdır, çünkü Workflows her step'i function'ın URL endpoint'ine bir HTTP isteği aracılığıyla invoke eder.
D. Function'lar özellikle CloudEvent functions olmalıdır, çünkü Workflows yalnızca CloudEvents formatını anlar.

**13.** Bir workflow tanımı, önce bir ilk Cloud Run function'ını çağıran, ardından ikinci bir tanesini çağıran bir step içeriyor. Workflow tanımı hangi formatta yazılır, ve veri iki step arasında nasıl hareket eder?

A. Tanım YAML ya da JSON'da yazılır, ve ilk step tarafından üretilen sonuç ikinci step'e input olarak sağlanabilir.
B. Tanım yalnızca proprietary bir Workflows'a özgü binary formatta yazılabilir, ve step'ler veri paylaşamaz.
C. Tanım XML'de yazılır, ve her step ihtiyaç duyduğu her veriyi bağımsız olarak yeniden çekmelidir.
D. Tanım yalnızca YAML'de yazılır (asla JSON'da değil), ve sonuçlar hiçbir koşulda step'ler arasında geçirilemez.

**14.** Cloud Run functions'ı invoke etmenin ötesinde, bir workflow'un harici bir REST API'yi çağırması ve aynı sürece bir Cloud Run service'i de dahil etmesi gerekiyor. Modül bunun tek bir workflow tanımı içinde başarılabilir olduğunu açıklıyor mu?

A. Hayır — bir workflow tanımı yalnızca Cloud Run functions'ı çağırabilir, başka hiçbir şeyi çağıramaz.
B. Yalnızca harici REST API dahil edilebilir; Cloud Run service'lere bir workflow'dan referans verilemez.
C. Yalnızca Cloud Run service dahil edilebilir; harici REST API'ler Workflows'un kapsamı dışındadır.
D. Evet — bir workflow tanımı, harici bir REST API'yi çağırmak için yapılandırma (örn. önceki bir sonucu query parametresi olarak geçirerek) ve bir Cloud Run service'e bağlanan, sonucu workflow'un genel sonucu haline gelebilen yapılandırma içerebilir.

**15.** Bir function'ın, o trafiğin hiçbirini genel internete açığa çıkarmadan, yalnızca internal IP address'i olan bir Compute Engine VM instance'ına erişmesi gerekiyor. Bunu hangi Google Cloud yeteneği mümkün kılar, ve hangi tür kaynağa bağlanır?

A. Özellikle Cloud Run functions'a internal IP address'leri açığa çıkarmak için tasarlanmış Cloud NAT.
B. Cloud Run functions'ı doğrudan bir VPC network'e bağlayan, Compute Engine VM instance'ları ve Memorystore gibi kaynaklara internal DNS ve internal IP address'ler üzerinden erişim sağlayan Serverless VPC Access.
C. Yalnızca on-premises network'leri Google Cloud'a bağlamak için kullanılan Cloud Interconnect.
D. Bir Cloud Run function'ının yalnızca internal-IP kaynağına erişmesi için hiçbir mekanizma yoktur; yalnızca public IP kaynaklarına erişilebilir.

**16.** Bir Serverless VPC Access connector'ı oluşturulmuş, bir VPC network'e bağlanmış ve kendine ayrılmış bir subnet verilmiş, ama function hâlâ VPC kaynaklarına erişemiyor. Connector, function'dan farklı bir region'da oluşturulmuş. Hangi gereksinim ihlal edildi, ve function connector'ı kullanabilmeden önce başka ne olması gerekir?

A. Region'ların eşleşmesi gerekmez; hata bunun yerine eksik bir firewall rule'undan kaynaklanmalıdır.
B. Connector'ın region'ının bağlantı üzerinde hiçbir etkisi yoktur; gerçek eksik adım Cloud NAT'i etkinleştirmektir.
C. Connector'ın region'ı, function'ın deploy edildiği region ile eşleşmelidir, ve buna ek olarak, connector'ı kullanması gereken her function'ın bunu kullanacak şekilde ayrı ayrı yapılandırılması gerekir.
D. Subnet'in diğer Google servisleriyle paylaşılması gerekir, ve onu münhasıran connector'a ayırmak hatanın kendisiydi.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Bir event trigger, Google Cloud projesi içindeki event'lere tepki verir ve bir event-driven function'a karşılık gelir — bu tam olarak açıklanan "harici caller yok" senaryosudur. HTTP trigger'lar ise HTTP function'lara karşılık gelir ve bir HTTP(S) isteği gerektirir.

**2. Doğru cevap: D.**
Tek bir function aynı anda yalnızca bir trigger'a bağlanabilir, ama aynı event yine de birden fazla function'ı çalıştırabilir — bu fan-out'a, aynı trigger source ayarlarını paylaşan birden fazla function deploy ederek ulaşırsınız, tek bir function'a birkaç trigger bağlayarak değil.

**3. Doğru cevap: A.**
Tüm event-driven Cloud Run functions, event delivery için Eventarc kullanır, ve Eventarc, Cloud Audit Logs, harici SaaS event kaynakları ve Pub/Sub'a publish edilen custom kaynaklar dahil 90'dan fazla Google Cloud kaynağını destekler.

**4. Doğru cevap: C.**
HTTP trigger'lar GET, POST, PUT, DELETE ve OPTIONS request method'larını destekler, ki bu doğrudan HTTP üzerinden tam CRUD tarzı etkileşimleri desteklemek için yeterlidir.

**5. Doğru cevap: D.**
Bir function'ın Pub/Sub trigger kullanabilmesi için event-driven bir function olarak implemente edilmesi gerekir; Cloud Run functions'ta Pub/Sub trigger'ları arka planda bir Eventarc trigger türü olarak implemente edilir.

**6. Doğru cevap: B.**
Bir CloudEvent function kullanılıyorsa, Pub/Sub event verisi ona CloudEvents formatında geçirilir; bir Background function kullanılıyorsa, aynı veri bunun yerine `PubsubMessage` formatında geçirilir — implementasyon stili, payload'ın şeklini belirler.

**7. Doğru cevap: C.**
Bir Background function kullanılıyorsa, Cloud Storage event verisi ona `StorageObjectData` formatında geçirilir (bir CloudEvent function bunun yerine CloudEvents formatını alırdı).

**8. Doğru cevap: A.**
Firestore trigger'ları yalnızca document seviyesinde uygulanır — belirli bir document field'ına ya da tüm bir collection'a özgülenmiş bir trigger oluşturmak mümkün değildir; kullanılabilir en ince granülerlik document'in kendisidir.

**9. Doğru cevap: D.**
Bir Firestore trigger'ının çalışması için Firestore'un function ile aynı Google Cloud projesinde olması gerekir — cross-project Firestore trigger'ları, organizasyon üyeliğinden ya da event türünden bağımsız olarak desteklenmez.

**10. Doğru cevap: B.**
Modül, Google Analytics for Firebase ve Firebase Authentication trigger'larının yalnızca Cloud Run functions (1st gen) için kullanılabilir olduğunu belirtir; Firebase Realtime Database ve Firebase Remote Config trigger'ları için aynı kısıtlama söz konusu değildir.

**11. Doğru cevap: B.**
Workflows, tamamen yönetilen, serverless bir orchestration platformudur, ve modül, bir yıla kadar state tutabilme, retry yapabilme, poll edebilme ya da bekleyebilme yeteneğini, uzun süren business process'leri pratik hale getiren şey olarak özellikle belirtir.

**12. Doğru cevap: C.**
Workflow step'i olarak kullanılan function'lar, HTTP trigger'larla HTTP function olarak yazılır ve deploy edilir, çünkü Workflows her step'i function'ın URL endpoint'ine gönderilen bir HTTP isteği aracılığıyla invoke eder.

**13. Doğru cevap: A.**
Bir workflow tanımı — step'lerin kümesi — YAML ya da JSON formatında yazılabilir, ve bir step (örn. ilk bir Cloud Run function'ı) tarafından üretilen sonuç, bir sonraki step'e (örn. ikinci bir Cloud Run function'ı) input olarak sağlanabilir.

**14. Doğru cevap: D.**
Bir workflow tanımı, harici bir REST API endpoint'ine bağlanmak için yapılandırma (örneğin önceki bir sonucu bir query parametresi olarak geçirerek) ve sonucu genel workflow'un sonucu haline gelebilen bir Cloud Run service'i bağlayan yapılandırma içerebilir.

**15. Doğru cevap: B.**
Serverless VPC Access, Cloud Run functions'ı doğrudan bir VPC network'e bağlar, Compute Engine VM instance'ları ve Memorystore gibi yalnızca internal IP address'i olan kaynaklara internal DNS ve internal IP address'ler üzerinden erişim sağlar, böylece trafik hiçbir zaman internete açığa çıkmaz.

**16. Doğru cevap: C.**
Bir Serverless VPC Access connector'ı için yapılandırılan region, Cloud Run function'ının deploy edildiği region ile eşleşmelidir. Bunun ötesinde, bir connector'a sahip olmak tek başına yeterli değildir — VPC network'e erişmesi gereken her function'ın o connector'ı kullanacak şekilde ayrı ayrı yapılandırılması gerekir.