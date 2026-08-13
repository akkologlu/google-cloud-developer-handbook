# Modül 14 — Securing Cloud Run Functions: Pratik Sorular

Bu set, identity-based erişim kontrolünü (authentication ile authorization arasındaki fark, service account ile user account arasındaki fark, OAuth 2.0 access token ile ID token arasındaki fark, Cloud Functions Admin/Developer/Invoker/Viewer IAM rolleri, event-driven ve HTTP function'ları kimin invoke edebileceği, developer test akışı, runtime service account'lar, ve `roles/run.invoker` ile audience-scoped bir ID token kullanan function'dan function'a çağrılar), network-based erişim kontrolünü (ingress ayarları, Serverless VPC Access ile egress ayarları, ve VPC Service Controls), ve Cloud KMS customer-managed encryption keys (CMEK) ile veriyi korumayı — CMEK'in neyi koruduğu, nasıl kurulduğu, yalnızca primary versiyon kısıtlaması, ve bir anahtar yok edildiğinde/devre dışı bırakıldığında ne olduğu — kapsar. Bu modül, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun, Modül 2'yi (Modül 13) takip eden 3. Modülü'dür.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: function'ların neden varsayılan olarak private olduğu, authentication'ın authorization'dan önce gelen sabit sırası, bir function'dan function'a çağrıda `roles/run.invoker` rolünün hangi kimliğe ait olduğu, ID token'ın `aud` alanının gerçekte neyi kısıtladığı, ingress ile egress ayarları arasındaki fark, ve bir CMEK anahtarını yok etmenin neden çalışan işi anında öldürmediği.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Bir ekip, herhangi bir authentication ayarı belirtmeden yeni bir Cloud Run function deploy ediyor. Varsayılan davranış nedir, ve bir caller'ın ne sağlaması gerekir?

A. Function, varsayılan olarak private'dır ve authentication gerektirir; bir caller'ın onu invoke edebilmesi için authenticate ve authorize edilmiş olması gerekir.
B. Ekip açıkça kısıtlamadığı sürece, function varsayılan olarak public'tir ve hiçbir authentication gerekmez.
C. Function varsayılan olarak private'dır, ama (herhangi bir projeden) authenticate edilmiş herhangi bir Google Account, başka hiçbir authorization kontrolü olmadan onu invoke edebilir.
D. Authentication varsayılanları function'ın trigger türüne bağlıdır — HTTP function'lar varsayılan olarak public'tir, event-driven function'lar varsayılan olarak private'dır.

**2.** Cloud Run functions'ın identity-based erişim kontrolü modelinde, doğru adım sırası nedir, ve ikinci adım neyi değerlendirir?

A. Önce authorization gerçekleşir ve caller'ın IAM rollerini kontrol eder, ardından authentication identity credential'ını doğrular.
B. Önce authentication, identity credential'ını doğrular; bu doğrulandıktan sonra authorization, kimliğin erişim seviyesini ya da izinlerini değerlendirir.
C. Authentication ve authorization, tanımlı bir sıra olmadan, tek bir birleşik adım olarak eş zamanlı gerçekleşir.
D. Yalnızca authorization gerçekleştirilir; Cloud Run functions ayrı bir authentication adımı yapmaz.

**3.** Bir workload identity bir VM'i temsil ediyor, ve ayrı bir identity, bir Google Group'un parçası olan bireysel bir kişiyi temsil ediyor. Bunlar sırasıyla hangi kimlik türleridir?

A. İkisi de user account'tır, çünkü Cloud Run functions tüm workload ve insan kimliklerine aynı şekilde davranır.
B. VM bir user account'tır ve kişi bir service account'tır — tipik kullanımın tersi.
C. VM'in kimliği bir service account'tır (kişi olmayan bir kimlik); kişinin kimliği ise bir user account'tır (bireysel bir Google Account sahibi ya da bir Google Group'un parçası).
D. Cloud Run functions yalnızca service account'ları destekler; user account'lar function'ları invoke etmek için hiçbir şekilde kullanılamaz.

**4.** İstemciler, her istekte ham bir service account credential'ı göndermek yerine, Cloud Run functions'a authenticate olmak için bir token oluşturur. Neden, ve Cloud Run functions hangi iki token türünü kullanır?

A. Token'lar yalnızca performans nedeniyle kullanılır; iki tür refresh token ve session cookie'dir.
B. Token'ların ömrü sınırsızdır ve yalnızca credential'ları HTTPS üzerinden göndermekten kaçınmak için kullanılır; iki tür API key ve bearer token'dır.
C. Token'lar, hiçbir güvenlik faydası olmayan, credential'lar etrafındaki isteğe bağlı kolaylık sarmalayıcılarıdır; Cloud Run functions SAML assertion'ları ve JWT'ler kullanır.
D. Sınırlı ömürlü token'lar, altta yatan credential sızarsa oluşabilecek potansiyel zararı sınırlar; Cloud Run functions, OAuth 2.0 framework'ü ve OpenID Connect (OIDC) ile oluşturulan OAuth 2.0 access token'ları (API çağrıları için) ve ID token'ları (geliştirici tarafından yazılmış koda yapılan çağrılar için) kullanır.

**5.** Bir developer'a bir function üzerinde Cloud Functions Invoker rolü veriliyor, ama Developer ya da Admin değil. Ne yapabilir ve ne yapamaz?

A. Function'ın kodunu ve yapılandırmasını güncelleyebilir, ama silemez.
B. Function'ı invoke edebilir (çağırabilir), ama rol, function'ın kendisini değiştirme ya da yönetme izni vermez.
C. Function'ın yapılandırmasını ve loglarını görüntüleyebilir, ama doğrudan invoke edemez.
D. Function'ı silebilir, çünkü Invoker, daha düşük ayrıcalıklı tüm administrative işlemleri örtük olarak içerir.

**6.** Pub/Sub-triggered, event-driven bir Cloud Run function'ının, tek seferlik manuel bir test için harici bir HTTP client tarafından doğrudan çağrılması gerekiyor. Bu, anlatıldığı şekliyle mümkün müdür?

A. Hayır — event-driven function'lar yalnızca abone oldukları event kaynağı tarafından invoke edilebilir, rastgele bir harici caller tarafından değil.
B. Evet — event-driven ya da HTTP fark etmeksizin, herhangi bir Cloud Run function her zaman harici bir HTTP client tarafından doğrudan invoke edilebilir.
C. Evet, ama yalnızca caller bir ID token yerine bir OAuth 2.0 access token sağlarsa.
D. Hayır — event-driven function'lar deploy edildikten sonra hiçbir şekilde invoke edilemez; herhangi bir test için HTTP function olarak yeniden deploy edilmeleri gerekir.

**7.** Bir developer, authentication gerektiren bir HTTP Cloud Run function'ını manuel olarak test etmek istiyor. Modül hangi adım sırasını anlatıyor?

A. Herhangi bir Google Account'tan bir OAuth 2.0 access token oluştur ve bunu `token` adlı bir query parametresinde geçir.
B. Developer testi için hiçbir token gerekmez; console, authenticate olmuş herhangi bir Google kullanıcısı için yerleşik bir bypass sağlar.
C. Bir service account key dosyası iste, bunu function'ın environment variable'larına göm, ve function'ı hiçbir header olmadan çağır.
D. Function üzerinde gerekli izinleri veren bir role sahip bir user account edin, o hesabı kullanarak bir ID token oluştur, ve token'ı isteğin `Authorization` header'ında geçir.

**8.** Bir hackathon sırasında en az direnç yolu olduğu için, bir function production'a, runtime kimliği olarak default Compute Engine service account'u kullanılarak deploy ediliyor. Modül bu seçim hakkında ne söylüyor?

A. Bu, önerilen production yapılandırmasıdır, çünkü default service account her bir function'a sıkı bir şekilde kapsamlandırılmıştır.
B. Runtime service account'lar production'da önemsizdir, çünkü function'lar runtime'da başka Google Cloud kaynaklarına asla erişmez.
C. Default service account yalnızca test ve geliştirme için kullanılmalıdır; production function'lar, yalnızca gereken minimum izinlere sahip dedicated bir runtime service account kullanmalıdır.
D. Default service account, geliştirmede bile hiçbir şekilde kullanılamaz — Cloud Run functions ilk deployment'tan itibaren custom bir service account gerektirir.

**9.** Function A'nın function B'yi çağırması gerekiyor, ve yalnızca function A'nın (başka hiçbir function'ın değil) B'yi invoke etmesine izin verilmeli. Bu, hangi yapılandırmayla sağlanır, ve A'nın ne sunması gerekir?

A. Alıcı function (B) üzerinde `roles/run.invoker`'ı, çağıran function'ın (A'nın) service account'una ver, ve A'nın, `aud` alanı B'nin URL'ine ayarlanmış, Google-signed bir ID token'ı `Authorization` header'ında sunmasını sağla.
B. Function A üzerinde `roles/run.invoker`'ı function B'nin service account'una ver, ve B'nin A'nın URL'ine scoped bir OAuth 2.0 access token sunmasını sağla.
C. Hiçbir IAM yapılandırması gerekmez; aynı proje içindeki herhangi bir function, varsayılan olarak başka herhangi bir function'ı çağırabilir.
D. Function B üzerinde Cloud Functions Admin rolünü function A'nın service account'una ver, çünkü function'dan function'a çağrılara yalnızca Admin izin verir.

**10.** Bir ekip, bir function'ın aynı proje içindeki Workflows'tan ve VPC network'lerinden gelen invocation'ları kabul etmesini, ama projenin ya da VPC Service Controls perimeter'ının dışından gelen istekleri reddetmesini istiyor. Bu, hangi ingress ayarıyla eşleşir, ve bu ayar aynı zamanda genel internet trafiğine de izin verir mi?

A. Allow all traffic — bu tek ayardır, ve kaynak ne olursa olsun her zaman internet trafiğine izin verir.
B. Allow internal traffic and traffic from Cloud Load Balancing — bu, load balancer üzerinden genel internet erişimine izin verir.
C. Bunun için bir ingress ayarı yoktur; yalnızca function'ın public URL'ini tamamen devre dışı bırakarak sağlanabilir.
D. Allow internal traffic only — internal traffic, aynı proje ya da VPC Service Controls perimeter'ı içindeki Workflows ve VPC network'lerinden gelen trafik olarak tanımlanır, ve genel internetten gelen trafiğe izin verilmez.

**11.** Bir function'ın private IP adreslerine giden outbound isteklerinin bir VPC network üzerinden yönlendirilmesi gerekiyor, diğer outbound trafiğin ise normal şekilde yönlendirilmesi gerekiyor. Önce ne yapılandırılmalı, ve hangi egress seçeneği bu gereksinimle eşleşir?

A. Önce hiçbir şey yapılandırılmasına gerek yoktur; egress ayarları herhangi bir VPC bağlantı mekanizmasından bağımsız çalışır.
B. Önce bir Serverless VPC Access connector, function'ı VPC network'e bağlamalıdır; ardından "route only requests to private IPs through the connector" egress seçeneği gereksinimle eşleşir.
C. Önce bir Cloud NAT gateway oluşturulmalıdır; ardından "route all outbound traffic through the connector" seçeneği eşleşir.
D. Önce "allow internal traffic only" ingress ayarı uygulanmalıdır; egress ayarları yalnızca ingress kısıtlandıktan sonra kullanılabilir.

**12.** Bir organizasyon, Cloud Run functions'ı için VPC Service Controls ile bir service perimeter kurar ve gerekli organization policy'leri yapılandırır. Modül, bunun sonucunda ortaya çıkan hangi üç davranıştan bahsediyor?

A. Function'lar, ingress ayarlarından bağımsız olarak tamamen public hale gelir, function'ların artık runtime service account'lara ihtiyacı kalmaz, ve egress trafiği kısıtlanmaz.
B. Yalnızca HTTP function'lar etkilenir; event-driven function'lar ve onların egress davranışı tamamen kısıtlanmamış kalır.
C. HTTP function'lar yalnızca perimeter içindeki bir VPC network'ten kaynaklanan trafiği kabul eder, tüm function'lar bir Serverless VPC Access connector kullanmak zorundadır, ve tüm function'lar tüm egress trafiğini VPC network üzerinden yönlendirmek zorundadır.
D. Function'lara, başka hiçbir yapılandırma gerekmeden otomatik olarak CMEK koruması verilir, ve ingress/egress ayarları önemsiz hale gelir.

**13.** Bir güvenlik ekibi, tamamen Google tarafından yönetilen anahtarlar yerine, kendilerinin sahip olduğu ve kontrol ettiği Cloud Run functions encryption key'leri istiyor. Bu anahtarlar ne olarak adlandırılır, ve hangi function verisi kategorilerini at rest korurlar?

A. Google-managed encryption key'ler; yalnızca function'ın environment variable'larını korurlar, başka hiçbir şeyi korumazlar.
B. Secret Manager key'leri; yalnızca function'ın kodu tarafından açıkça referans verilen secret'ları korurlar.
C. Default encryption key'ler; at rest veriyi değil, transit halindeki network trafiğini korurlar.
D. Customer-managed encryption keys (CMEK); function source code'unu, build sonuçlarını (container image ve deploy edilen instance'lar) ve internal event transport channel'ların at-rest verisini korurlar.

**14.** Bir function için CMEK kurarken, bir mühendis Artifact Registry repository'sini function'ın kendisinde kullanılandan farklı bir anahtarla oluşturuyor, ve anahtara yalnızca Cloud Storage service account'una erişim veriyor. Bu kurulum doğru mudur?

A. Hayır — repository, function ile aynı anahtarı kullanmalıdır, ve Cloud Run Functions, Artifact Registry ve Cloud Storage service account'larının her birine, anahtar üzerinde Cloud KMS CryptoKey Encrypter/Decrypter rolü verilmelidir.
B. Evet — repository'ler, function'dan bağımsız olarak herhangi bir anahtar kullanabilir, ve yalnızca bir service account'a erişim vermek yeterlidir.
C. Hayır — repository'nin CMEK'i etkinleştirmesine hiç gerek yoktur; yalnızca function'ın kendisinin bir anahtara ihtiyacı vardır.
D. Evet — Cloud Storage service account'unun erişimi olduğu sürece, diğer iki service account erişimi otomatik olarak devralır.

**15.** Bir mühendis, anahtar yeni bir primary versiyona rotate edildikten sonra bile, bir function için CMEK'in her zaman belirli, sabitlenmiş bir key versiyonunu kullanmasını istiyor. Cloud Run functions bunu destekliyor mu?

A. Evet — CMEK'i etkinleştirirken istenen key versiyon numarasını belirtin, ve süresiz olarak sabitlenmiş kalır.
B. Evet, ama yalnızca 1st generation function'lar için; 2nd generation function'lar her zaman primary versiyonu kullanır.
C. Hayır — Cloud Run functions, CMEK koruması için her zaman anahtarın primary versiyonunu kullanır; belirli bir key versiyonu seçilemez.
D. Evet, ama yalnızca anahtar bir software key olarak değil, bir HSM cluster'ında saklanıyorsa.

**16.** Bir function'ın verisini koruyan bir CMEK anahtarı, birkaç execution zaten devam ederken ve bir function instance'ı aktif olarak çalışırken yok ediliyor. Bu devam eden execution'lara ve çalışan instance'a, yeni execution'lara kıyasla ne olur?

A. Devam eden execution'lar ve aktif instance dahil her şey, daha fazla veri açığa çıkmasını önlemek için anında sonlandırılır.
B. Aktif instance'lar kapatılmaz ve devam eden execution'lar çalışmaya devam eder; yeni execution'lar, ve yeni bir instance gerektiren herhangi bir execution, anahtar erişilemez kaldığı sürece başarısız olur.
C. Devam eden execution'lar anında başarısız olur, ama yeni execution'ların cache'lenmiş, şifrelenmemiş veriyi kullanarak normal şekilde başlamasına izin verilir.
D. Yok edilen anahtarlar 24 saat sonra otomatik olarak geri yüklendiği için, gelecekteki execution'lar dahil hiçbir şey etkilenmez.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Function'lar varsayılan olarak private olarak deploy edilir ve authentication gerektirir; bir function'ı public olarak deploy edip authentication'ı atlamak, açıkça seçmeniz gereken bir şeydir.

**2. Doğru cevap: B.**
Önce authentication gelir — requestor'ın gerçekten iddia ettiği kişi olduğunu teyit etmek için identity credential'ını doğrular. Bu teyit edildikten sonra ancak authorization, kimliğin erişim seviyesini ya da izinlerini değerlendirir.

**3. Doğru cevap: C.**
Service account'lar, bir function, application ya da VM gibi kişi olmayan kimlikleri temsil eder; user account'lar ise kişileri, ya bireysel Google Account sahipleri olarak ya da bir Google Group'un parçası olarak temsil eder.

**4. Doğru cevap: D.**
Token tabanlı authentication, özellikle bir service ya da user account credential'ı sızarsa oluşabilecek potansiyel zararı sınırlamak için sınırlı ömürlü token'lar kullanır. Cloud Run functions, API çağrılarını authenticate etmek için OAuth 2.0 access token'ları ve geliştirici tarafından yazılmış koda yapılan çağrıları authenticate etmek için ID token'ları kullanır; ikisi de OAuth 2.0 framework'ü ve OpenID Connect (OIDC) kullanılarak oluşturulur.

**5. Doğru cevap: B.**
Cloud Functions Invoker rolü, function'ı invoke etme izni verir; Developer ya da Admin rollerinin sağladığı şekilde kodu, yapılandırmayı değiştirme ya da function'ı yönetme yeteneğini içermez.

**6. Doğru cevap: A.**
Event-driven function'lar yalnızca abone oldukları event kaynağı tarafından invoke edilebilir — harici bir HTTP client, hangi token'ı sunarsa sunsun, onları doğrudan çağıramaz.

**7. Doğru cevap: D.**
Developer test akışı, function üzerinde uygun izinleri veren bir role sahip bir user account gerektirir, o hesaptan bir ID token oluşturmayı, ve bu token'ı isteğin `Authorization` header'ında geçirmeyi gerektirir.

**8. Doğru cevap: C.**
Modül açıkça, default runtime service account'un (Compute Engine default service account, ya da 1st gen için App Engine default service account) yalnızca test ve geliştirme için kullanılması gerektiğini belirtir; production deployment'ları, yalnızca gereken minimum izinlere sahip dedicated bir runtime service account belirtmelidir.

**9. Doğru cevap: A.**
Bir function'ın başka bir function'ı çağırmasına izin vermek ve erişimi yalnızca o caller'a kısıtlamak için, alıcı function (B) üzerinde `roles/run.invoker`'ı çağıran function'ın (A'nın) service account'una verirsiniz. Çağıran function ayrıca, `Authorization` header'ında gönderilen, `aud` alanı alıcı function'ın URL'ine ayarlanmış, Google-signed bir ID token sağlamalıdır.

**10. Doğru cevap: D.**
"Allow internal traffic only", invocation'ları, aynı proje ya da VPC Service Controls perimeter'ı içindeki Workflows ve VPC network'lerinden gelen trafik olarak tanımlanan internal traffic ile kısıtlar — genel internetten kaynaklanan trafiğe izin vermez.

**11. Doğru cevap: B.**
Egress ayarları, önce function'ın bir Serverless VPC Access connector aracılığıyla bir VPC network'e bağlanmasını gerektirir; "route only requests to private IPs through the connector", diğer outbound trafiği etkilemeden yalnızca private-IP'ye giden trafiği connector üzerinden yönlendiren egress seçeneğidir.

**12. Doğru cevap: C.**
VPC Service Controls organization policy'leri yerinde olduğunda, HTTP function'lar yalnızca service perimeter içindeki bir VPC network'ten kaynaklanan trafiği kabul eder, tüm function'lar bir Serverless VPC Access connector kullanmak zorundadır, ve function'lar tüm egress trafiğini VPC network üzerinden yönlendirmek zorundadır.

**13. Doğru cevap: D.**
Customer-managed encryption keys (CMEK), Google'a değil müşteriye aittir, ve function source code'unu, build sürecinin sonuçlarını (container image ve deploy edilen her function instance'ı), ve internal event transport channel'ların at-rest verisini korur.

**14. Doğru cevap: A.**
Artifact Registry repository'si, function için CMEK'i etkinleştirmede kullanılan anahtarla aynı anahtarı kullanmalıdır, ve Cloud Run Functions, Artifact Registry ve Cloud Storage service account'larının her birine, anahtar üzerinde erişim — özellikle Cloud KMS CryptoKey Encrypter/Decrypter rolü — verilmelidir.

**15. Doğru cevap: C.**
Cloud Run functions, CMEK koruması için her zaman bir anahtarın primary versiyonunu kullanır; bir function için CMEK'i etkinleştirirken kullanılacak belirli bir key versiyonu belirtemezsiniz.

**16. Doğru cevap: B.**
Bir anahtar yok edilirse, devre dışı bırakılırsa, ya da gerekli izinleri geri alınırsa, aktif function instance'ları kapatılmaz ve zaten devam eden execution'lar çalışmaya devam eder; yeni execution'lar, ve yeni bir function instance'ı gerektiren execution'lar, Cloud Run functions anahtara erişimi olmadığı sürece başarısız olur.
