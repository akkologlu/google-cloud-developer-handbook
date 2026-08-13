# Modül 15 — Integrating with Cloud Databases: Pratik Sorular

Bu set, Cloud Run functions'ı Memorystore'a (Redis/Memcached, fully managed özellikler, region eşleşmesi gereksinimiyle Serverless VPC Access bağlantı akışı) bağlamayı, environment variables'ı (saklama, kapsam, nasıl ayarlanıp okunduğu), Firestore'u (document/collection veri modeli, trigger event türleri, yalnızca-Native-mode kısıtlaması, document path kuralları, snapshot'lar, `DocumentReference` ile Firebase Admin SDK arasındaki fark), Secret Manager'ı (secret ile secret version arasındaki fark, `secretmanager.secretAccessor` rolü, volume-mount ile environment-variable version davranışı arasındaki fark, cross-project erişim), ve BigQuery Remote Functions'ı (`CLOUD_RESOURCE` connection, nesle bağlı Invoker rolleri, ve bir remote function'ı bir query'den invoke etmek için gereken izinler) kapsar. Bu modül, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun, Modül 3'ü (Modül 14) takip eden 4. Modülü'dür.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: Serverless VPC Access connector'ının region'ının neden veritabanının değil function'ın region'ıyla eşleşmesi gerektiği, Firestore trigger'larının neden yalnızca Native mode'da çalıştığı, bir secret ile bir secret version arasındaki fark, volume-mount edilen bir secret'ın rotation'ı neden anında yakaladığı ama bir environment-variable secret'ının yakalamadığı, ve BigQuery connection'ının Invoker rolünün neden function'ın nesline bağlı olduğu.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Modül, Cloud Run functions'ın birden fazla Google Cloud veritabanıyla entegre olabileceğini belirtiyor, ama yalnızca ikisine derinlemesine giriyor, üçüncüsü ise ek okuma olarak kapsanıyor. Bu üçü hangileridir, ve nasıl ele alınırlar?

A. Altı veritabanının hepsi (Firestore, Cloud SQL, Spanner, Bigtable, BigQuery, Memorystore), hiçbir ek okuma olmadan eşit derinlikte kapsanır.
B. Memorystore ve Firestore derinlemesine kapsanır; BigQuery, BigQuery Remote Functions üzerine ayrı bir ek okuma materyali olarak kapsanır.
C. Bu modülde yalnızca Cloud SQL ve Spanner kapsanır; Firestore ve Memorystore için dokümantasyona yönlendirilir.
D. Modül, Bigtable ve BigQuery'yi derinlemesine kapsar, Memorystore ise yalnızca geçerken bahsedilir.

**2.** Memorystore'u değerlendiren bir ekip, hangi operasyonel görevleri otomatikleştirdiğini ve hangi diğer Google Cloud servisleriyle entegre olduğunu bilmek istiyor. Modül ne söylüyor?

A. Memorystore yalnızca patching'i otomatikleştirir; provisioning, replication ve failover hâlâ müşteri tarafından manuel olarak yönetilmelidir.
B. Memorystore, müşterinin kendi Redis cluster node'larını yönetmesini gerektirir; yalnızca onlara yönetilen bir network yolu sağlar.
C. Memorystore, cache içeriğini sorgulamak için BigQuery ile ve otomatik cache warming için Cloud Build ile entegre olur.
D. Memorystore, provisioning, replication, failover ve patching'i otomatikleştirir, ve güvenli erişim için IAM ile, servis izleme ve alerting için Cloud Monitoring ile entegredir.

**3.** Bir geliştirici, Memorystore'un desteklediği iki caching engine'i birbirinden ayırt etmesi gerekiyor. Modül, Redis ile Memcached'i nasıl tanımlıyor?

A. Redis, veritabanı, cache, message broker ve streaming engine olarak kullanılabilen açık kaynaklı bir in-memory veri yapısı deposudur; Memcached, açık kaynaklı, dağıtık bir memory object caching sistemidir.
B. Redis, dağıtık bir memory object caching sistemidir; Memcached, aynı zamanda bir message broker olarak da hizmet verebilen bir veri yapısı deposudur.
C. Redis ve Memcached, Memorystore'un sunduğu aynı altta yatan teknolojinin yalnızca iki farklı marka adıdır.
D. Redis, Google'a özel bir caching engine'dir; Memcached, Memorystore'un desteklediği gerçekten açık kaynaklı tek seçenektir.

**4.** Bir Redis instance'ı `us-east1`'de çalışıyor, ve ona bağlanacak function `europe-west1`'de deploy ediliyor. Serverless VPC Access connector'ı hangi region'da oluşturulmalıdır?

A. `us-east1`, Redis instance'ının region'ıyla eşleşerek, çünkü connector o instance'a ulaşmaya adanmıştır.
B. Her iki region da eşit derecede iyi çalışır, çünkü connector otomatik olarak tüm region'larda çalışır.
C. `europe-west1`, function'ın deployment region'ıyla eşleşerek — connector ardından ayrı bir adım olarak Redis instance'ının authorized VPC network'üne bağlanır.
D. Function'ın ve veritabanının region'ları arasındaki çakışmaları önlemek için üçüncü, ilgisiz bir region seçilmelidir.

**5.** Serverless VPC Access connector `Ready` durumuna ulaştıktan sonra, function deployment'ının ne belirtmesi gerekir, ve function daha sonra nasıl invoke edilir?

A. Yalnızca connector'ın adı; Redis host ve port, hiçbir yapılandırma gerekmeden runtime'da otomatik olarak keşfedilir, ve function bir POST isteğiyle invoke edilir.
B. Connector'ın path/adı artı Redis host IP adresi ve port'u için environment variable'lar; function, URL endpoint'ine bir HTTP GET isteği gönderilerek invoke edilir.
C. Function'ın source code'una derlenmiş, sabit kodlanmış (hardcoded) bir Redis bağlantı string'i; invocation sabit bir zamanlamada otomatik olarak gerçekleşir.
D. Redis credential'larına bir Secret Manager referansı; invocation, özellikle Redis erişimi için verilmiş, imzalı bir ID token gerektirir.

**6.** Bir Cloud Run function'ının environment variable'ları nerede saklanır, ve kapsamları nedir?

A. Secret Manager'da saklanır ve proje içindeki her function arasında global olarak paylaşılır.
B. Function'ın source code repository'sinde saklanır ve her function'ın tüm version'ları arasında paylaşılır.
C. Cloud Storage'da saklanır ve herhangi bir belirli function'ın lifecycle'ından bağımsız olarak kalıcıdır.
D. Cloud Run functions backend'inde saklanır, tek bir function'a bağlıdır, ve o function'ın lifecycle'ı içinde var olur.

**7.** Bir ekip, her deploy'da manuel olarak yazmak yerine function'larının environment variable tanımlarını source control'de tutmak istiyor, ve function'ları Python ile yazılmış. Modül hangi seçenekleri anlatıyor?

A. Key-value çiftlerini source control'deki bir YAML dosyasında saklayın ve dosyanın adını deployment zamanında sağlayın; Python'da, runtime variable'larına `os` modülünü kullanarak erişin.
B. Environment variable'lar bir dosyadan kaynaklanamaz; her deploy'da her zaman gcloud CLI ya da console üzerinden manuel olarak yazılmalıdır.
C. Cloud Run functions'ın hiçbir deployment flag'i olmadan otomatik olarak algıladığı bir `.env` dosyasında saklayın; Python bunları okumak için özel bir modül gerektirmez.
D. Yalnızca bir Secret Manager secret'ında saklayın; `os` modülü Python'da Cloud Run functions environment variable'larını okuyamaz.

**8.** Modüle göre, Firestore sakladığı veriyi nasıl yapılandırır?

A. Veri, ilişkisel bir veritabanına benzer şekilde, tablolar içinde satır ve sütunlar olarak saklanır.
B. Veri, hiçbir gruplama mekanizması olmadan düz key-value çiftleri olarak saklanır.
C. Bir key-value çiftleri kümesi bir document olarak saklanır, ve tüm document'lar collection'larda saklanır.
D. Veri, yalnızca sayısal bir offset ile indekslenen, key-value yapısı olmayan binary blob'lar olarak saklanır.

**9.** Bir Cloud Run function'ının kodu, hangi Firestore event türlerini işleyecek şekilde implemente edilebilir, ve bu event'leri hangi SDK exposes eder?

A. Yalnızca "read" event'leri; Firestore trigger'ları write'lara tepki veremez, yalnızca veritabanına karşı yapılan sorgulara tepki verebilir.
B. Document created, updated, deleted, ya da bunlardan herhangi biri (genel bir write event'i) — Cloud Run functions for Firebase SDK tarafından exposed edilir.
C. Yalnızca "created" event'leri; update'ler ve delete'ler ayrı bir polling mekanizmasıyla işlenmelidir.
D. Yalnızca collection seviyesinde "created" ve "deleted" event'leri; bireysel document event'leri desteklenmez.

**10.** Bir ekibin Firestore veritabanı Datastore mode'da çalışıyor, ve ona bir Cloud Run function trigger'ı eklemek istiyorlar. Bu destekleniyor mu?

A. Hayır — Cloud Run functions için Firestore trigger'ları yalnızca Native mode'daki Firestore için kullanılabilir, Datastore mode'daki Firestore için değil.
B. Evet — veritabanı Native mode'da mı yoksa Datastore mode'da mı olduğuna bakılmaksızın Firestore trigger'ları aynı şekilde çalışır.
C. Evet, ama yalnızca "delete" event'leri için; Datastore mode, kısıtlı bir trigger türleri alt kümesini destekler.
D. Hayır — Firestore trigger'ları her iki mode'da da kullanılamaz; yalnızca doğrudan client-library polling desteklenir.

**11.** Firestore-triggered bir function yapılandırırken, document path ile ilgili ne belirtilmelidir, ve hangi syntax kuralı geçerlidir?

A. Hiçbir document path gerekmez — yalnızca event type belirtilmesi gerekir; trigger tüm veritabanına uygulanır.
B. Bir document path belirtilmelidir, ve her zaman tek, kesin bir document'e referans vermelidir — wildcard pattern'lere izin verilmez.
C. Bir document path belirtilmelidir, ve path'in sonunu belirtmek için her zaman bir trailing slash ile bitmelidir.
D. Bir document path belirtilmelidir — belirli bir document'e referans verebilir ya da bir wildcard pattern kullanabilir, ve bir trailing slash içeremez.

**12.** Firestore-triggered bir function'ın (1) kendisini tetikleyen document'in hem güncelleme öncesi hem güncelleme sonrası durumunu görmesi, ve (2) tamamen farklı, ilgisiz bir document'i de güncellemesi gerekiyor. Modül, her birini sağlamak için ne söylüyor?

A. İkisi de mümkün değildir — triggered function'lar yalnızca güncelleme sonrası durumu görebilir ve tetikleyen ya da başka hiçbir document'i asla değiştiremez.
B. İkisi de Firebase Admin SDK gerektirir; snapshot'ın `ref` property'si hiçbir document değişikliği için kullanılamaz.
C. Update event'lerinde, hem güncelleme öncesi hem sonrası snapshot verisi mevcuttur; tetikleyen document, snapshot'ın `ref` property'sindeki `DocumentReference` aracılığıyla değiştirilebilir, tetikleyenin dışındaki document'leri okumak ya da yazmak ise Firebase Admin SDK gerektirir.
D. Yalnızca güncelleme sonrası durum her zaman mevcuttur, ve tetikleyen de dahil olmak üzere herhangi bir document'i değiştirmek her zaman Firebase Admin SDK gerektirir.

**13.** Secret Manager'da bir "secret" ile bir "secret version" arasındaki ilişki nedir, ve bir function'ın runtime service account'una, bir tanesini okuyabilmesi için ne verilmelidir?

A. Bir secret, metadata (replication location'lar, label'lar, permission'lar) artı secret version'larını tutan bir nesnedir; bir secret version, gerçek veriyi bir text string ya da binary blob olarak saklar; runtime service account'un, secret üzerinde `roles/secretmanager.secretAccessor` rolüne ihtiyacı vardır.
B. Bir secret ve bir secret version aynı şeydir, birbirinin yerine kullanılır; function aynı projedeyse bir secret'ı okumak için hiçbir IAM rolü gerekmez.
C. Bir secret version, metadata'yı tutan üst konteynerdir, secret ise yalnızca ham text ya da binary değeri saklar; function'ın onu okumak için Cloud Functions Admin rolüne ihtiyacı vardır.
D. Bir secret, gerçek veri olmadan yalnızca replication metadata'sı saklar; gerçek credential değeri her zaman Cloud Storage'da ayrı olarak saklanır, Storage Object Viewer rolüyle erişilebilir.

**14.** Bir ekip, çalışan bir function tarafından kullanılan bir secret'ı rotate ediyor ve değişikliğin, bir redeploy beklemeden, hâlihazırda çalışan instance'lar tarafından anında yakalanmasını istiyor. Hangi erişim yöntemi bunu sağlar, ve neden?

A. Environment variable — çünkü environment variable'lar yalnızca startup'ta değil, her function invocation'ında yeniden çözümlenir.
B. Secret'ı bir volume olarak mount etmek — çünkü onu dosyadan okumak her zaman en son version'ı getirir, oysa bir environment variable instance startup'ında bir kez çözümlenir ve o version'a sabitlenmiş kalır.
C. Hiçbir yöntem rotate edilmiş bir secret'ı otomatik olarak yakalamaz; projenin tamamen silinip yeniden oluşturulması gerekir.
D. Environment variable — çünkü Secret Manager, rotation event'lerini doğrudan çalışan bir instance'ın process environment'ına push eder.

**15.** Project A'daki bir function'ın, Project B'de saklanan bir secret'ı okuması gerekiyor. Bunun çalışması için, olağan erişim izninin ötesinde ne yapılmalıdır?

A. Ekstra hiçbir şey — Secret Manager, bir organizasyondaki her secret'ı otomatik olarak içindeki herhangi bir function'a erişilebilir kılar.
B. Secret önce Project A'ya kopyalanmalıdır, çünkü Secret Manager hiçbir cross-project referans biçimini desteklemez.
C. Project A ve Project B tek bir projeye birleştirilmelidir, çünkü secret'lar hiçbir koşulda proje sınırları arasında referans verilemez.
D. Function'ın runtime service account'una secret'a olağan şekilde erişim verin, ve secret'ı, Project B'nin project ID'sini ve secret adını içeren bir resource path kullanarak referanslayın.

**16.** Bir ekip, BigQuery'deki bir Google Standard SQL sorgusundan doğrudan bir Cloud Run function'ı çağırmak istiyor. Bunu hangi mekanizma sağlar, connection'ın service account'una hangi rol verilmelidir (ve bu, function'ın nesline nasıl bağlıdır), ve bir caller'ın sorguyu gerçekten çalıştırması için hangi izinlere ihtiyacı vardır?

A. Bu mümkün değildir; BigQuery yalnızca diğer BigQuery-native function'ları çağırabilir, Cloud Run functions gibi harici servisleri asla çağıramaz.
B. Bir BigQuery remote function bunu sağlar, ama gereken rol, nesilden bağımsız olarak aynıdır (hem 1st hem 2nd gen için `roles/run.invoker`), ve sorguyu çalıştırmak için hiçbir dataset-seviyesi izin gerekmez.
C. `CLOUD_RESOURCE` bir connection aracılığıyla oluşturulan bir BigQuery remote function bunu sağlar; connection'ın service account'unun, function üzerinde Cloud Functions Invoker'a (1st gen) ya da Cloud Run Invoker'a (2nd gen) ihtiyacı vardır, ve caller'ın dataset üzerinde `roles/bigquery.dataViewer`'a artı connection üzerinde `roles/bigquery.connectionUser`'a ihtiyacı vardır.
D. Bir BigQuery scheduled query bunu sağlar, proje üzerinde yalnızca `roles/bigquery.admin` gerektirir, hiçbir connection ya da function-seviyesi IAM yapılandırması gerekmez.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
Modül, entegre edilebilir veritabanları olarak Firestore, Cloud SQL, Spanner, Bigtable, BigQuery ve Memorystore'u listeler, ama yalnızca Memorystore ve Firestore'a derinlemesine girer; BigQuery ise BigQuery Remote Functions üzerine ayrı bir ek okuma ile kapsanır.

**2. Doğru cevap: D.**
Memorystore, provisioning, replication, failover ve patching'i otomatikleştiren fully managed bir servistir, ve güvenli erişim için IAM ile, servis izleme ve alerting için Cloud Monitoring ile entegredir.

**3. Doğru cevap: A.**
Redis, veritabanı, cache, message broker ve streaming engine olarak kullanılan açık kaynaklı bir in-memory veri yapısı deposu olarak tanımlanır; Memcached, açık kaynaklı, dağıtık bir memory object caching sistemi olarak tanımlanır.

**4. Doğru cevap: C.**
Serverless VPC Access connector, function ile aynı region'da — burada `europe-west1` — oluşturulmalıdır, ve ardından ayrı bir adım olarak Redis instance'ının authorized VPC network'üne bağlanır.

**5. Doğru cevap: B.**
Function deployment'ı, connector'ın path/adını, Redis host IP adresi ve port'u için environment variable'larla birlikte belirtmelidir; function, URL endpoint'ine bir HTTP GET isteği gönderilerek invoke edilir.

**6. Doğru cevap: D.**
Environment variable'lar, Cloud Run functions backend'inde saklanır, tek bir function'a bağlıdır, ve o function'ın lifecycle'ı içinde var olur.

**7. Doğru cevap: A.**
Environment variable key-value çiftlerini, source control'de tutulan bir YAML dosyasında saklayabilir ve dosyanın adını function deployment'ı sırasında sağlayabilirsiniz; Python'da, runtime environment variable'larına `os` modülünü kullanarak erişirsiniz.

**8. Doğru cevap: C.**
Firestore, bir key-value çiftleri kümesini bir document olarak saklar, ve tüm document'lar collection'larda saklanır.

**9. Doğru cevap: B.**
Function kodu, bir document oluşturulduğunda, güncellendiğinde, silindiğinde, ya da bu event'lerden herhangi biri gerçekleştiğinde tetiklenen Firestore event'lerini işleyecek şekilde implemente edilebilir; bu event'ler Cloud Run functions for Firebase SDK tarafından exposed edilir.

**10. Doğru cevap: A.**
Cloud Run functions için Firestore trigger'ları yalnızca Native mode'daki Firestore için kullanılabilir — Datastore mode'daki Firestore için kullanılamazlar.

**11. Doğru cevap: D.**
Firestore event'leri üzerinde tetiklenen bir function, bir document path (belirli bir document'e referans verebilir ya da bir wildcard pattern kullanabilir) belirtmelidir ve bir trailing slash içeremez.

**12. Doğru cevap: C.**
Update event'lerinde, güncelleme öncesi ve sonrası document snapshot verisi function'a ulaşır. Tetikleyen document, snapshot'ın `ref` property'sinde bulunan (Firestore Node.js SDK'sından gelen) `DocumentReference` aracılığıyla değiştirilebilir, tetikleyenin dışındaki document'leri okumak ya da yazmak ise Firebase Admin SDK gerektirir.

**13. Doğru cevap: A.**
Bir secret, metadata koleksiyonu (replication location'lar, label'lar, permission'lar ve diğer bilgiler) ve secret version'larını içeren bir nesnedir; bir secret version, gerçek secret verisini bir text string ya da binary blob olarak saklar. Bir secret'a erişmek için, function'ın runtime service account'una, secret üzerinde `roles/secretmanager.secretAccessor` rolü verilmelidir.

**14. Doğru cevap: B.**
Secret'ı bir volume olarak mount etmek, function'ın dosya her okunduğunda secret'ın en son version'ına referans vermesini sağlar. Secret'ı bir environment variable olarak geçirmek, onu function instance startup time'ında bir kez çözümler, bu yüzden function o anda geçerli olan version'a sabitlenir.

**15. Doğru cevap: D.**
Function'ın runtime service account'una secret'a olağan şekilde erişim verirsiniz, ve secret'ı, project ID'yi (Project B'ninkini) ve secret adını içeren resource path'ini belirterek function'a ulaştırırsınız.

**16. Doğru cevap: C.**
`CLOUD_RESOURCE` bir connection aracılığıyla oluşturulan bir BigQuery remote function, bir Google Standard SQL sorgusunun bir Cloud Run function'ı invoke etmesini sağlar. Connection'ın service account'una, 1st gen bir function için Cloud Functions Invoker rolü ya da 2nd gen bir function için Cloud Run Invoker rolü verilmelidir. Remote function'ı bir query'den invoke etmek için, caller'ın dataset üzerinde `roles/bigquery.dataViewer`'a ve connection üzerinde `roles/bigquery.connectionUser`'a ihtiyacı vardır.
