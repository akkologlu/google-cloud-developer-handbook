# Modül 16 — Best Practices for Cloud Run Functions: Pratik Sorular

Bu set, implementation best practice'lerini (idempotency, her zaman bir HTTP response döndürmek, arka plan/async işlerini ve temp dosyalarını temizlemek, `process.exit()`/`sys.exit()`'ten kaçınmak, uncaught exception'ları handle etmek, Error Reporting ve varsayılan logging), performans ve networking'i (bir cold start'ın gerçekte ne yaptığı, kullanılmayan dependency'leri azaltmak, instance recycling nedeniyle pahalı nesneleri global scope'ta cache'lemek, lazy initialization, minimum instance sayıları, persistent bağlantılar, Google API client'ları, ve Serverless VPC Access), retry on failure'ı (kullanılabilirlik, etkinleştirme/devre dışı bırakma, transient hatalar için güvenle kullanmak, ve sonsuz retry döngülerinden kaçınmak), ve configuration/scaling best practice'lerini (least privilege ve function'dan function'a erişim, dedicated runtime service account'lar, memory/CPU ilişkisi, timeout, concurrency, ve revisions/traffic) kapsar. Bu modül, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun, Modül 4'ü (Modül 15) takip eden 5. Modülü'dür.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: idempotency'nin neden güvenli retry'ların ön koşulu olduğu, uncaught bir exception'ın neden yalnızca mevcut değil gelecekteki invocation'ları da etkilediği, global-scope caching'in neden instance recycling sayesinde işe yaradığı, lazy initialization'ın neden aynı caching tavsiyesine bir denge unsuru olarak var olduğu, retry'ın bug'lar test yoluyla düzeltilmeden asla etkinleştirilmemesi gerektiği, ve concurrency'yi etkinleştirmenin function'ın kendi koduna neden gerçek bir sorumluluk yüklediği.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Bir function bazen yarıda başarısız oluyor ve tekrarlanan yan etkiler üretmeden (örn. bir müşteriyi iki kez ücretlendirmeden) güvenle retry edilmesi gerekiyor. Function hangi özelliğe sahip olmalı, ve modül bunu doğrudan neden retry güvenliğine bağlıyor?

A. Function stateless olmalıdır, çünkü statelessness tek başına, function'ın ne yaptığından bağımsız olarak güvenli retry'ları garanti eder.
B. Function idempotent olmalıdır — birden fazla kez çağrıldığında aynı sonucu üretmelidir — bu da kısmen başarısız olan bir invocation'ı retry etmeyi güvenli kılan şeydir.
C. Function, herhangi bir retry denenmeden önce temiz bir sonlanmayı garanti etmek için `process.exit()` kullanmalıdır.
D. Function otomatik retry'ı tamamen devre dışı bırakmalıdır, çünkü retry'lar tasarımdan bağımsız olarak her function için doğası gereği güvensizdir.

**2.** Bir HTTP function bazen bir response göndermeyi başaramıyor, ve ayrı olarak, bazı function'lar invocation bitmiş gibi görünse de asenkron işi çalışır durumda bırakıyor. Modül, bu durumların her biri hakkında ne uyarıyor?

A. İkisi de gerçek bir endişe değildir — Cloud Run functions, herhangi bir invocation'ı hiçbir maliyet etkisi olmadan sabit, kısa bir timeout'tan sonra otomatik olarak kapatır, ve arka plan işi her invocation için izole edilir.
B. Response'suz bir HTTP function, timeout'tan bağımsız olarak yalnızca birkaç milisaniye için faturalandırılır, ve kalan arka plan işi sonraki invocation'ları etkilemeden güvenle atılır.
C. Yalnızca eksik HTTP response bir sorundur (timeout'a kadar faturalandırma); kalan arka plan async işi, bir sonraki invocation'ı ısıtmanın bir yolu olarak açıkça desteklenir.
D. Hiçbir zaman response döndürmeyen bir HTTP function, timeout'a kadar çalışmaya (ve faturalandırılmaya) devam edebilir, ve kalan arka plan aktivitesi, aynı ortamdaki sonraki bir invocation sırasında devam edip ona müdahale edebilir — bu yüzden function sonlanmadan önce tüm asenkron işlemler bitmelidir.

**3.** Bir function, her invocation sırasında geçici dizine birkaç dosya yazıyor ama bunları hiç silmiyor. Modül, bunun neyi riske attığını ve nedenini söylüyor?

A. Geçici dizin, in-memory bir dosya sisteminin parçasıdır, bu yüzden dosyalar function'ın kullanılabilir belleğini tüketir ve açıkça silinmedikçe sonunda bir cold start'a zorlayabilir.
B. Hiçbir şey — geçici dizine yazılan dosyalar, her tek invocation'dan sonra belleğe hiçbir risk oluşturmadan otomatik olarak silinir.
C. Dosyalar risksizdir çünkü geçici dizin, function'ın bellek tahsisinden tamamen ayrı, kalıcı disk depolamasıyla desteklenir.
D. Bu yalnızca function'ın belleğiyle ya da performansıyla ilgisi olmayan ayrı bir disk kotasını aşma riski taşır; cold start'lar etkilenmez.

**4.** Bir Node.js function, temiz bir şekilde sonlandığından emin olmak için handler'ının sonunda `process.exit()` çağırıyor. Modül bu pratik hakkında ne söylüyor?

A. Bu, herhangi bir function'ı sonlandırmanın önerilen yoludur, çünkü mümkün olan en hızlı kapanmayı ve en düşük faturalandırmayı garanti eder.
B. Bu, özellikle event-driven function'lar için gereklidir, ama HTTP function'lar bunu asla yapmamalıdır.
C. `process.exit()` (ya da Python'da `sys.exit()`) ile manuel olarak çıkmak beklenmedik davranışa neden olabilir; function bunun yerine event-driven function'lardan örtük/açık olarak return etmeli, HTTP function'lardan ise HTTP response döndürmelidir.
D. Bunun her iki durumda da hiçbir etkisi yoktur, çünkü Cloud Run functions açık exit çağrılarını tamamen görmezden gelir.

**5.** Bir function bazen kodun hiçbir yerinde yakalanmayan bir exception fırlatıyor. Mevcut invocation'ı potansiyel olarak bozmanın ötesinde, modül bunun neye neden olduğunu söylüyor?

A. Mevcut invocation'ın ötesinde hiçbir şey — uncaught exception'ların, onları fırlatan invocation dışında hiçbir invocation üzerinde etkisi yoktur.
B. Bir yönetici tarafından manuel olarak yeniden deploy edilene kadar function'ı kalıcı olarak devre dışı bırakır.
C. Function için otomatik olarak retry'ı etkinleştirir, önceden yapılandırılmış olan retry ayarını geçersiz kılar.
D. Mevcut invocation üzerindeki etkisine ek olarak, function'ın gelecekteki invocation'larında da bir cold start'a zorlar — bu yüzden runtime hataları her zaman kodda handle edilmelidir.

**6.** Bir function'ın runtime exception'ları nereye gider, `stdout`/`stderr`'e yazılan log'lara ne olur, ve bir hata oluştuğunda bir HTTP function ile bir event-driven function sırasıyla nasıl tepki vermelidir?

A. Bir üçüncü parti logging kütüphanesi açıkça entegre edilmedikçe hem exception'lar hem loglar atılır; hiçbir function türünün tanımlı bir hata-yanıt deseni yoktur.
B. Runtime exception'lar, toplama, görüntüleme ve bildirim için Error Reporting'e gönderilir; `stdout`/`stderr` log'ları hiçbir ek yapılandırma olmadan otomatik olarak console'da görünür; HTTP function'lar hatayı raporlamalı ve uygun bir HTTP status code ile yanıt vermeli, event-driven function'lar ise raporlamalı ve bir hata mesajı döndürmelidir.
C. Runtime exception'lar yalnızca Cloud Trace'e gider, Error Reporting'e değil; her iki function türü de hiçbir şey raporlamadan sessizce retry etmelidir.
D. `stdout`/`stderr` log'ları, console'da herhangi bir şey görünmeden önce açıkça manuel olarak kurulması gereken bir logging agent gerektirir.

**7.** Bir cold start gerçekte ne yapar, ve modül, eklediği gecikmeyi azaltmak için hangi belirli pratiği öneriyor?

A. Bir cold start, function'ın execution environment'ını oluşturur ve initialize eder; bu süreçte, function'ın import ettiği dependency'ler yüklenir, invocation gecikmesine eklenir; function'ın kullanmadığı dependency'leri yüklemekten kaçınmak bu gecikmeyi ve deploy süresini azaltır.
B. Bir cold start yalnızca function'ın faturalandırma katmanını etkiler, gecikmesini değil; onu azaltmak, dependency'lere dokunmak yerine memory tahsisini artırmayı gerektirir.
C. Bir cold start, function'ın tüm deployment pipeline'ını her tek invocation'da yeniden çalıştırır; hiçbir kod-seviyesi değişiklikle azaltılamaz.
D. Bir cold start, trafiği kasıtlı olarak kısıtlayan bir Cloud Run functions özelliğidir; önceden daha fazla dependency yüklemek, etkisini azaltmanın önerilen yoludur.

**8.** Bir function, handler'ının içinde her tek invocation'da yeni bir database client nesnesi oluşturuyor, bu da pahalı. Modül ne öneriyor, ve bu neden işe yarıyor?

A. Bunun yerine client'ı bir try/catch bloğu içinde oluşturmaya geçin — yalnızca exception handling, yeniden oluşturma maliyetini ortadan kaldırır.
B. Hiçbir şey yapılamaz — her invocation her zaman yepyeni, ilgisiz bir ortamda çalışır, bu yüzden hiçbir değer invocation'lar arasında asla yeniden kullanılamaz.
C. Client'ı bir global-scope değişkeni olarak tanımlayın — çünkü bir function instance'ının execution environment'ı invocation'lar arasında sıklıkla recycled olur, bir global-scope değeri, aynı instance'a yapılan sonraki invocation'larda yeniden hesaplanmadan yeniden kullanılabilir.
D. Client oluşturmayı ayrı bir Cloud Run function'a taşıyın ve bunun yerine her invocation'da senkron olarak çağırın.

**9.** Bir function'ın birkaç pahalı global-scope değişkeni var, ama function'ın kod yollarının yalnızca bazıları her birini gerçekten kullanıyor. Modül ne öneriyor, ve bu hangi sorunu çözüyor?

A. Bunun yerine her global değişkeni bir Secret Manager secret'ına taşıyın, çünkü Secret Manager değerleri global scope'tan daha hızlı çözümler.
B. Bu global değişkenleri tembel (lazy), yani ihtiyaç anında initialize etmeyi düşünün — çünkü global değişkenleri initialize etmek her zaman cold-start gecikmesini artırır, ve belirli bir değişkeni asla kullanmayan kod yolları onu initialize etmenin maliyetini ödememelidir.
C. Kullanılmayan global değişkenleri tamamen silin ve cold ya da warm start'tan bağımsız olarak değerlerini handler içinde her invocation'da yeniden hesaplayın.
D. Hiçbir şeyin değişmesine gerek yok — global değişken initialization'ının cold-start gecikmesi üzerinde ölçülebilir bir etkisi yoktur.

**10.** Bir ekip, herhangi bir kod-seviyesi optimizasyondan bağımsız olarak cold start'ları azaltmak ve uygulamalarının genel gecikmesini iyileştirmek istiyor. Modül hangi yapılandırmayı öneriyor?

A. Retry'ı tamamen devre dışı bırakın, çünkü retry, cold start'ların önde gelen nedeni olarak tanımlanır.
B. Function'ın timeout'unu mümkün olan en düşük değere ayarlayın, modül bunun cold start'ları doğrudan önlediğini söylüyor.
C. Maksimum instance sayısını mümkün olduğunca yükseltin, çünkü daha fazla maksimum instance her zaman daha az cold start anlamına gelir.
D. İstekleri karşılamaya hazır tutulacak minimum bir function instance sayısı belirleyin, bu cold start'ları azaltır ve genel performansı iyileştirir.

**11.** Bir function, harici bir URL'ye sık sık outbound HTTP çağrıları yapıyor, her invocation'da bir Google API'yi çağırıyor, ve ayrıca bir VPC network içindeki bir kaynağa erişmesi gerekiyor. Modül hangi üç networking pratiğini öneriyor?

A. Harici URL için persistent HTTP bağlantıları oluşturup global scope'ta cache'leyin, gereksiz bağlantıları ve DNS sorgularını önlemek için Google API service client nesnesini global scope'ta oluşturun, ve VPC-internal kaynak için internal DNS/IP adresleriyle bir Serverless VPC Access connector kullanın.
B. Bayat bağlantı riskini önlemek için harici URL için her invocation'da yeni bir bağlantı açıp kapatın, ve basitlik için tüm Google API çağrılarını ve VPC trafiğini genel internet üzerinden yönlendirin.
C. Harici HTTP bağlantısını yalnızca function handler'ının local scope'unda cache'leyin, böylece her invocation temiz bir kopya alır, ve her zaman gecikmeyi artırdığı için Serverless VPC Access kullanmaktan kaçının.
D. Üç bağlantının tümünü yapılandırmak için yalnızca environment variable'lar kullanın, çünkü global scope hiçbir türde network bağlantı nesnesi tutamaz.

**12.** Bir ekip, başarısız olduğunda bir HTTP function'ının otomatik olarak kendini retry etmesini istiyor. Bu mümkün mü, ve ayrı olarak, otomatik retry gerçekte nasıl etkinleştirilir ya da devre dışı bırakılır, ve varsayılan olarak ne kadar süre retry eder?

A. Evet, otomatik retry hem HTTP hem event-driven function'lar için kullanılabilir; varsayılan olarak etkindir ve manuel olarak durdurulana kadar süresiz retry eder.
B. Cloud Run functions'da, herhangi bir function türü için, herhangi bir yapılandırma altında hiçbir otomatik retry mekanizması yoktur.
C. Otomatik retry, HTTP function'lar için kullanılamaz — yalnızca event-driven function'lar için kullanılabilir — ve orada bile varsayılan olarak devre dışıdır; `gcloud functions deploy` CLI komutunda `--retry` flag'i ile (ya da console'un "Retry on failure" seçeneğiyle) etkinleştirilir ve varsayılan olarak en fazla 7 gün retry eder.
D. Otomatik retry yalnızca HTTP function'lar için kullanılabilir, varsayılan olarak etkindir, ve devre dışı bırakmanın hiçbir yolu olmadan tam olarak 24 saat retry eder.

**13.** Bir ekip, her tek invocation'da başarısız olmasına neden olan bir bug'a sahip bir event-driven function üzerinde, önce bug'ı test edip düzeltmeden retry'ı etkinleştiriyor. Modül ne olacağını söylüyor, ve önce ne yapılmalıydı?

A. Retry, kalıcı bir bug'a sahip herhangi bir function'ı otomatik olarak tespit edip atlar, bu yüzden sorunlu hiçbir şey olmaz.
B. Function, ilk başarısız retry'ından sonra platform tarafından otomatik olarak devre dışı bırakılır, daha fazla maliyeti önler.
C. Retry, function'ın kodundaki bug'ları otomatik olarak düzeltmek için özel olarak tasarlanmıştır, bu yüzden onu önce etkinleştirmek aslında önerilen sorun giderme adımıdır.
D. Function başarılı olana kadar sürekli retry edildiği için, kalıcı bir bug, hiçbir zaman başarılı olmadan tekrar tekrar başarısız olmasına neden olur — başarısızlığa neden olan bug'lar, retry'ı etkinleştirmeden önce test yoluyla bulunup düzeltilmelidir, ve kalıcı başarısızlıklarda sonsuz retry döngülerini önlemek için bir end condition (örn. bir timestamp'ten daha eski event'leri discard etmek) eklenmelidir.

**14.** Bir ekip, bir login function'ının bir user-profiles function'ını çağırması gerektiği ama bir search function'ını çağıramaması gereken çoklu-function bir servis için IAM tasarlıyor, ve henüz function'ların hiçbirine dedicated bir kimlik atamadılar. Modül bu iki konu genelinde ne öneriyor?

A. Basitlik için her function'a diğer her function'a geniş erişim verin, ve production'da bile yalnızca default service account'a güvenin, çünkü IAM rolleri kimlikten bağımsız olarak tek başına yeterlidir.
B. Least privilege'ı izleyin — her function'ın erişimini gereken minimum kullanıcı/service account sayısına ve izne sınırlayın, her function'ı yalnızca gerçekten ihtiyaç duyduğu belirli function alt kümesini çağırmakla kısıtlayın (login → user-profiles, ama search değil), ve production için, her function'a default olana güvenmek yerine dedicated bir user-managed runtime service account atayın.
C. Admin, function'dan function'a erişimi kısıtlayabilen tek rol olduğu için, Cloud Functions Admin rolünü tüm çağıran function'lara geniş bir şekilde verin.
D. Modül, bu tür çağrıların bir proje içinde her zaman örtük olarak güvenilir olduğunu belirttiği için, internal function'dan function'a çağrılar için IAM yapılandırmasını tamamen atlayın.

**15.** Bir ekip, CPU-yoğun bir function için bir memory ayarı seçiyor ve ayrı olarak erken timeout'lardan kaçınmak istiyor. Modül, memory'nin CPU ile nasıl ilişkili olduğunu ve timeout'un nasıl ayarlanması gerektiğini söylüyor?

A. Tahsis edilen memory miktarı, tahsis edilen CPU miktarına karşılık gelir, bu yüzden yetersiz bir memory ayarı, function'ı dolaylı olarak CPU açısından aç bırakabilir; timeout, function'ın gerçek execution süresinden biraz daha yüksek ayarlanmalıdır.
B. Memory ve CPU, aralarında hiçbir ilişki olmadan tamamen bağımsız olarak yapılandırılır, ve timeout, execution süresinden bağımsız olarak her zaman platformun mutlak maksimumuna ayarlanmalıdır.
C. CPU tahsisi memory tahsisini belirler (ters ilişki), ve timeout, milisaniyeye kadar tam olarak execution süresiyle eşleşecek şekilde ayarlanmalıdır.
D. Memory ve CPU ilişkisizdir, ve modül, CPU-yoğun function'lar için timeout'ları tamamen devre dışı bırakmayı önerir.

**16.** Bir ekip, tek bir instance'ın aynı anda birden fazla isteği işleyebilmesi için bir Cloud Run function'ında concurrency'yi etkinleştiriyor, cold start'ları azaltmayı umuyor. Bu, hangi varsayılan davranışı değiştirir, ve function'ın koduna hangi yeni sorumluluğu yükler?

A. Varsayılan olarak, tek bir instance zaten sınırsız eşzamanlı isteği işler, bu yüzden concurrency'yi etkinleştirmenin davranış ya da kod gereksinimleri üzerinde hiçbir etkisi yoktur.
B. Concurrency yalnızca faturalandırma granülerliğini değiştirir, gerçek istek işlemeyi değil; function'ın koduna hiçbir zaman değişiklik gerekmez.
C. Varsayılan olarak, bir function instance'ı aynı anda yalnızca bir isteği işler; concurrency'yi (function'ın altta yatan Cloud Run servisi aracılığıyla yapılandırılarak) etkinleştirmek, zaten warm olan bir instance'ın ek eşzamanlı istekleri absorbe etmesine izin verir, ama Cloud Run functions aynı instance'taki istekler arasında izolasyon sağlamadığı için function'ın kodu eşzamanlı çalıştırmaya güvenli olmalıdır.
D. Concurrency, bir minimum instance sayısı belirlendiğinde otomatik olarak etkinleşir, ve her eşzamanlı istek tamamen izole bir alt-ortamda çalıştığı için hiçbir kod değişikliği gerektirmez.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
Function'ları idempotent yazmak, birden fazla kez çağrıldıklarında aynı sonucu ürettikleri anlamına gelir, bu da bir nedenden dolayı kısmen başarısız olan bir invocation'ı retry etmeyi güvenli kılan tam olarak şeydir.

**2. Doğru cevap: D.**
Bir HTTP function her zaman bir HTTP response döndürmelidir, aksi halde timeout'a kadar çalışmaya devam edebilir ve o süre boyunca ücretlendirilebilir. Ayrı olarak, invocation sonlandıktan sonra hiçbir arka plan aktivitesi çalışmamalıdır — çünkü CPU erişilemezdir, ve aynı ortamdaki sonraki bir invocation, bu arka plan aktivitesinin devam etmesine ve ona müdahale etmesine neden olabilir, bu yüzden function sonlanmadan önce tüm asenkron işlemler bitmelidir.

**3. Doğru cevap: A.**
Function'ların yazdığı geçici dizin, in-memory bir dosya sisteminin parçasıdır; bu dosyalar function'a kullanılabilir belleği tüketir ve bazen invocation'lar arasında kalıcı olur, bu yüzden oluşturulan dosyaları açıkça silmek, sonunda bellek tükenmesini ve bir cold start'ı tetiklemeyi önler.

**4. Doğru cevap: C.**
Function'lardan `process.exit()` (Node.js) ya da `sys.exit()` (Python) kullanarak manuel olarak çıkmak beklenmedik davranışa neden olabilir; bunun yerine, event-driven function'lardan örtük ya da açık olarak return edin, ve HTTP function'lardan HTTP response döndürün.

**5. Doğru cevap: D.**
Uncaught exception'lar function'ı sonlandırabilir ve gelecekteki function invocation'larında cold start'lara zorlayabilir — etki, exception'ı fırlatan invocation ile sınırlı değildir, bu yüzden runtime hataları ve exception'lar her zaman function kodunda handle edilmelidir.

**6. Doğru cevap: B.**
Bir function'dan yayılan runtime exception'lar Error Reporting'e gönderilir, orada toplanabilir, görüntülenebilir, ve bildirimleri tetiklemek için kullanılabilir. Cloud Run functions varsayılan olarak basit runtime logging içerir, bu yüzden `stdout` ya da `stderr`'e yazılan log'lar otomatik olarak console'da görünür. HTTP function'lar hatayı raporlamalı ve hata türüne göre uygun bir HTTP status code ile yanıt vermeli, event-driven function'lar ise bir exception oluştuğunda raporlamalı ve bir hata mesajı döndürmelidir.

**7. Doğru cevap: A.**
Bir cold start, bir function'ın execution environment'ını oluşturur ve initialize eder; bu süreç sırasında, function'ın import ettiği tüm dependency'ler yüklenir, invocation gecikmesine eklenir — function'ın kullanmadığı dependency'leri yüklemekten kaçınmak, bu gecikmeyi ve function'ı deploy etmek için gereken süreyi azaltır.

**8. Doğru cevap: C.**
Bir function instance'ının önceki bir invocation'ından kalan execution environment'ı sıklıkla recycled olur; bir değişkeni global scope'ta tanımlamak, değerinin, yeniden hesaplanmadan, aynı instance'a yapılan sonraki invocation'larda yeniden kullanılmasına izin verir — bu, her invocation'da yeniden oluşturulması pahalı olan API client nesneleri gibi nesneleri cache'lemek için uygun bir yaklaşımdır.

**9. Doğru cevap: B.**
Global değişkenlerin initialization'ı, her zaman bir function'ın cold-start invocation'ları sırasındaki gecikmesini artırır; bazı global değişkenler her kod yolunda kullanılmıyorsa, bunları ihtiyaç anında, tembel (lazy) olarak initialize etmeyi düşünmelisiniz, böylece belirli bir değişkene ihtiyaç duymayan yollar onu initialize etmenin maliyetini ödemez.

**10. Doğru cevap: D.**
Function instance'ları gelen istek hacmine göre ölçeklendiğinden, istekleri karşılamaya hazır tutulacak minimum bir instance sayısı belirlemek, function'ın cold start'larını azaltır ve uygulamanın genel performansını iyileştirir.

**11. Doğru cevap: A.**
Function'lardan URL'lere erişirken, per-invocation bağlantı kurma CPU süresini ve connection-quota riskini azaltmak için persistent HTTP bağlantıları oluşturun ve global scope'ta cache'leyin. Google API'leriyle iletişim kurarken gereksiz bağlantıları ve DNS sorgularını önlemek için, Google service client nesnesini global scope'ta oluşturun. VPC-internal kaynaklara, trafiği internete açmadan erişmek için, internal DNS ve internal IP adresleriyle Serverless VPC Access connector'ları kullanın.

**12. Doğru cevap: C.**
Otomatik retry yalnızca event-driven function'lar için kullanılabilir ve varsayılan olarak devre dışıdır. `gcloud functions deploy` CLI komutunda `--retry` flag'i ile, ya da deploy sırasında Google Cloud console'da "Retry on failure" seçilerek etkinleştirilir; retry etkinken, bir event, function başarıyla çalışana ya da maksimum retry süresi dolana kadar, varsayılan olarak en fazla yedi gün boyunca tekrar tekrar retry edilir.

**13. Doğru cevap: D.**
Bir function başarıyla çalışana kadar sürekli retry edildiği için, function başarısızlığına neden olan bug'lar, retry'ları etkinleştirmeden önce test yoluyla kodda bulunup düzeltilmelidir — ve başarısızlıklar kalıcı olabileceği için, sonsuz retry döngülerini önlemek amacıyla, function'ın işleme kodu çalışmadan önce bir end condition (belirli bir timestamp'ten daha eski event'leri discard etmek gibi) eklenmelidir.

**14. Doğru cevap: B.**
En az ayrıcalık ilkesini izleyerek, function'lara erişim minimum sayıda kullanıcı ve service account ile, gereken minimum izinle sınırlandırılmalıdır. Her function, yalnızca gerçekten ihtiyaç duyduğu belirli diğer function alt kümesine istek gönderebilmekle kısıtlanmalıdır — modülün kendi örneği, bir login function'ının bir user-profiles function'ına erişebilmesi gerektiği ama muhtemelen bir search function'ına erişememesi gerektiğidir. Production için, her function'a, default olana güvenmek yerine bir user-managed service account aracılığıyla dedicated bir kimlik verilmelidir.

**15. Doğru cevap: A.**
Bir function için seçilen tahsis edilmiş memory miktarı, tahsis edilmiş bir CPU miktarına karşılık gelir, bu yüzden memory ve CPU bağımsız değil, birbirine bağlıdır. Ayrı olarak, bir function'ın erken timeout'a uğramasını önlemek için, timeout süresi, function'ın gerçek execution süresinden biraz daha yüksek belirtilmelidir.

**16. Doğru cevap: C.**
Varsayılan olarak, function instance'ları aynı anda yalnızca bir isteği işler. Bu davranış, Cloud Run functions için, tek bir instance'ın function'ın altta yatan Cloud Run servisi üzerinden yapılandırılan bir concurrency değeriyle birden fazla eşzamanlı isteği işleyebilmesi için değiştirilebilir — bu, zaten warm olan bir instance'ın ek istekleri işlemesine izin vererek cold start'ları azaltır. Concurrency etkinken, function'ın kodu eşzamanlı çalıştırmaya güvenli olmalıdır, çünkü Cloud Run functions, aynı instance tarafından işlenen eşzamanlı istekler arasında izolasyon sağlamaz.
