# Modül 12 — Introduction to Cloud Run Functions: Pratik Sorular

Bu soru seti şunları kapsıyor: Cloud Run functions'ın ne olduğu ve Cloud Run ile ilişkisi, iki versiyon (2nd gen vs. 1st gen), HTTP functions vs. event-driven functions, CloudEvent functions vs. Background functions, dil runtime'larının source kodu kuralları ve entry point, deployment IAM gereksinimleri ve `gcloud` flag'leri, üç source konumu seçeneği, ve otomatik Cloud Build → Artifact Registry build pipeline'ı. Bu modül, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 1. Modülü'dür — modül 09–11'de işlenen "Service Orchestration and Choreography on Google Cloud" kursundan bağımsız bir kurstur.

Sorular, insanların gerçekten takıldığı ayrımlara ağırlık veriyor: Cloud Run functions'ın neden Cloud Run'dan ayrı bir ürün olmadığı, Background functions'ın neden kendi başına serbestçe seçilebilecek bir alternatif stil olmadığı, HTTP vs. event-driven arasındaki kimlik doğrulama ve trigger farkları, ve source kodunu deploy etmekle çalışan bir function elde etmek arasında (otomatik olarak) gerçekte ne olduğu.

Önce tüm soruları cevaplamayı dene, ardından cevaplarını aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle karşılaştır.

---

## Sorular

**1.** Bir geliştirici şöyle diyor: "Cloud Run functions, Cloud Run'dan tamamen ayrı bir execution ürünüdür — sadece isimleri aynı." Bu doğru mu?

A. Evet — Cloud Run functions'ın Cloud Run ile hiçbir ilişkisi olmayan, kendine ait bağımsız bir execution ortamı vardır.
B. Evet, ama yalnızca HTTP functions için; event-driven functions tamamen farklı bir platformda çalışır.
C. Hayır — Cloud Run functions (2nd gen), function'ları arka planda Cloud Run service'leri olarak deploy eder; bu ilişki, gerektiğinde bir function'ın düz Cloud Run'a ya da Kubernetes'e taşınabilmesinin de sebebidir.
D. Hayır — Cloud Run functions, Cloud Run'ın yerini almıştır ve Cloud Run artık deprecated bir üründür.

**2.** Bir takım, yeni bir proje için "Cloud Run functions" ile "Cloud Run functions (1st gen)" arasında karar vermeye çalışıyor. İkisi arasındaki temel yapısal fark nedir?

A. Cloud Run functions (2nd gen), Cloud Run üzerinde bir service olarak deploy edilir ve Eventarc ve Pub/Sub kullanılarak tetiklenir; Cloud Run functions (1st gen), daha sınırlı event trigger'lara ve yapılandırılabilirliğe sahip orijinal versiyondur.
B. Gerçek bir fark yoktur; iki isim aynı altyapıya işaret eder.
C. Cloud Run functions (1st gen), 2nd gen versiyonundan daha fazla dil runtime'ı destekler.
D. Cloud Run functions (2nd gen), HTTP trigger'ları desteklemez, yalnızca event trigger'ları destekler.

**3.** Bir function'ın, bir webhook çağıran dış bir sistem tarafından invoke edilmesi ve istekleri alacağı stabil bir URL'e ihtiyacı var. Bu hangi function türüne uyar, ve varsayılan erişim ayarı hakkında ne doğrudur?

A. Bir event-driven function; hiçbir türde varsayılan erişim kısıtlaması yoktur.
B. Bir Background function; URL'ler yalnızca Background functions'a atanır, HTTP functions'a asla atanmaz.
C. Bir HTTP function; kimlik doğrulama HTTP functions için hiçbir zaman kullanılamaz, yalnızca event-driven functions için kullanılabilir.
D. Bir HTTP function; istekleri almak için bir URL atanır, ve varsayılan olarak ona gelen istekler kimlik doğrulama gerektirir (unauthenticated erişim deployment sırasında etkinleştirilebilir).

**4.** Bir function'ın, hiçbir dış çağıran olmadan, belirli bir Cloud Storage bucket'ına yeni bir obje yüklendiğinde otomatik olarak çalışması gerekiyor. Bu hangi function türünü ve mekanizmayı tanımlar?

A. Bir webhook trigger kullanan bir HTTP function.
B. Cloud Storage source'una bağlı bir event trigger kullanan bir event-driven function.
C. Cloud Storage'a yalnızca Background functions tepki verebilir; CloudEvent functions veremez.
D. Bu, Cloud Scheduler gerektirir, çünkü Cloud Run functions storage event'lerine doğrudan tepki veremez.

**5.** Bir takım event-driven bir function implemente ediyor ve Cloud Run functions'ın tüm dil runtime'larında desteklediği, güncel, endüstri-standardı tabanlı yaklaşımı kullanmak istiyor. Hangi implementasyon stilini kullanmalılar, ve bu ne üzerine kuruludur?

A. CloudEvent functions; CloudEvents endüstri standardı spesifikasyonu üzerine kuruludur ve kullanıcı function'larını kalıcı bir HTTP uygulamasında saran açık kaynaklı bir kütüphane olan Functions Framework'e kayıtlıdır.
B. Background functions, çünkü ikisinden daha yeni olan stildir.
C. HTTP functions, çünkü event-driven functions'ın kendine ait bir implementasyon stili yoktur.
D. İki stil de her runtime'da ve her generation'da hiçbir kısıtlama olmadan aynı şekilde çalışır.

**6.** Background functions, modülde event-driven function'ın "daha eski stili" olarak tanımlanıyor. Aslında nerede kullanılabilirler?

A. Her yerde — Background functions, hem Cloud Run functions'da hem de Cloud Run functions 1st gen'de, desteklenen her dilde aynı şekilde çalışır.
B. Yalnızca Cloud Run functions'da (2nd gen), ve yalnızca .NET, Ruby ve PHP ile.
C. Yalnızca Cloud Run functions 1st gen'de, ve yalnızca Node.js, Python, Go ve Java runtime'larıyla.
D. Background functions aslında hiç desteklenmemiştir; modül onlardan yalnızca varsayımsal olarak bahseder.

**7.** Bir Node.js Cloud Run function, Cloud Run functions source'unu bulamadığı için deploy edilemiyor. Varsayılan yapılandırmanın override edilmediği varsayıldığında, en olası sebep nedir?

A. Node.js function'ları, dilden bağımsız olarak her zaman `main.py` adlandırılmalıdır.
B. Node.js, Cloud Run functions'ı hiç desteklemez.
C. `package.json` dosyası silinmiş, bu da Cloud Run functions'ın source kodunu nerede aradığıyla ilgisizdir.
D. Source kodu, function dizininin kökünde `index.js` adlı bir dosyada tanımlanmamış.

**8.** Bir geliştirici bir function deploy ediyor ve `--entry-point processOrder` belirtiyor. Bu flag neyi yapılandırır, ve `processOrder` nerede tanımlanmış olmalıdır?

A. Deployment region'ını ayarlar; `processOrder` geçerli bir Google Cloud region adı olmalıdır.
B. Function invoke edildiğinde çalıştırılan function'ı (ya da dile bağlı olarak class'ı) belirtir; function'ın main source dosyasında ya da root package'ında tanımlanmış olmalıdır.
C. Source kodunu stage etmek için kullanılan Cloud Storage bucket'ını adlandırır.
D. Hangi kodun gerçekten çalıştığı üzerinde hiçbir etkisi olmayan opsiyonel metadata'dır.

**9.** Bir function'ın, tek tek 45 dakikaya kadar çalışan istekleri işlemesi gerekiyor, ve client uygulamalar tarafından doğrudan HTTP ile invoke ediliyor. Bu, Cloud Run functions'ın desteklediği limitler içinde midir?

A. Evet — HTTP functions, event-driven functions için 10 dakikaya kadar olana karşılık, 60 dakikaya kadar bir run time limitini destekler.
B. Hayır — hiçbir Cloud Run function, hiçbir koşulda 10 dakikadan uzun çalışamaz.
C. Hayır — yalnızca event-driven functions birkaç saniyeden uzun çalışabilir.
D. Evet, ama yalnızca function Cloud Run functions (1st gen) olarak deploy edilmişse.

**10.** Yüksek trafikli bir function instance'ının, çok sayıda ayrı instance başlatmadan aynı anda birçok isteği işlemesi gerekiyor. Cloud Run functions hangi concurrency kapasitesini sağlar, ve modül buna hangi faydayı atfediyor?

A. Cloud Run functions instance'ları, tasarım gereği her zaman aynı anda yalnızca bir isteği işleyebilir.
B. Concurrency, instance başına 10 isteğe kadar yapılandırılabilir, cold start'lar üzerinde hiçbir etkisi yoktur.
C. Concurrency yalnızca event-driven functions için geçerlidir, HTTP functions için hiçbir zaman geçerli değildir.
D. Her instance, 1000'e kadar eşzamanlı isteği işleyebilir; bu, cold start'ları azaltır ve scale olurken genel latency'yi iyileştirir.

**11.** Bir Cloud Run function'ı deploy etmeye çalışan bir kullanıcının Cloud Functions Developer IAM rolü var, ama deployment yine de runtime service account'uyla ilgili bir yetki hatasıyla başarısız oluyor. En olası eksik olan nedir?

A. Başka hiçbir şeye gerek olmamalı; Cloud Functions Developer rolü tek başına her zaman yeterlidir.
B. Kullanıcının Organization Admin'e ihtiyacı vardır, çünkü function deployment'ı her zaman organizasyon seviyesinde izinler gerektirir.
C. Kullanıcının, Cloud Run functions runtime service account'u üzerinde Service Account User IAM rolüne de ihtiyacı vardır.
D. Cloud Run functions'ı deploy etmek IAM rollerini içermez; yalnızca API key'ler kullanılır.

**12.** Bir takım, bir function'ın source kodunu bir Cloud Storage bucket'ında saklanan bir zip dosyasından deploy etmek istiyor. Bunun çalışması için hangi yapısal ve izin gereksinimleri vardır?

A. Cloud Storage bir source konumu olarak kullanılamaz; yalnızca local machine ve source repository'ler desteklenir.
B. Zip'in source dosyaları zip dosyasının kökünde olmalıdır, ve deploy eden hesap (1st gen) ya da Cloud Run functions service agent'ı (2nd gen) bucket'tan okuma izni gerektirir.
C. Zip dosyası, Cloud Run functions tüm arşivi taradığı için source'u herhangi bir derinlikte iç içe barındırabilir.
D. Generation'dan bağımsız olarak bucket'tan okumak için hiçbir izin gerekmez.

**13.** Bir takım, source repository'lerinin belirli bir revision'ından, yalnızca o repository'nin bir alt dizinindeki kodu kullanarak deploy etmek istiyor. Bu, `--source` değerinde nasıl ifade edilir, ve deploy eden service agent'ın repository üzerinde hangi IAM rolüne ihtiyacı vardır?

A. Source repository yolu, revision için `revisions/<revision_name>` içerir, alt dizine işaret etmek için sonuna `paths/<source_directory_path>` eklenir; Cloud Run functions service agent'ının repository üzerinde Source Repository Reader (`roles/source.reader`) rolüne ihtiyacı vardır.
B. Bir alt dizinden deploy etmenin bir yolu yoktur; repository kökü her zaman tamamen kullanılmalıdır.
C. Alt dizinler, `--source` yerine `--entry-point` flag'iyle belirtilir.
D. Bir source repository'den deploy etmek, diğer iki source seçeneğinin aksine, hiçbir zaman bir IAM rolü gerektirmez.

**14.** Bir mimar, yeni bir Cloud Run function için bir deployment region'ı seçiyor ve önemli olan tek faktörün, son kullanıcılara fiziksel olarak en yakın region'ı seçmek olduğunu savunuyor. Modüle göre bu akıl yürütmede eksik olan nedir?

A. Hiçbir şey — kullanıcılara yakınlık, modülün tartıştığı tek faktördür.
B. Region seçiminin hiçbir koşulda fiyatlandırma üzerinde etkisi yoktur.
C. Cloud Run functions yalnızca tek bir global region'a deploy edilebilir, bu yüzden region seçimi gerçek bir karar değildir.
D. Latency ve availability temel değerlendirmelerdir, ama uygulamanın bağımlı olduğu diğer Google Cloud servislerinin konumu da önemlidir, çünkü birden fazla lokasyona yayılmış servisler kullanmak hem latency'yi hem fiyatlandırmayı etkileyebilir.

**15.** Bir geliştirici, local source dizinine işaret eden bir deploy komutu çalıştırdıktan sonra, function çalıştırılabilir hale gelmeden önce otomatik olarak ne olur, ve source kodunu Cloud Run functions'ın çalıştırabileceği bir şeye dönüştürmekten hangi iki servis sorumludur?

A. Hiçbir şey otomatik olarak olmaz; geliştiricinin bir container image'ı kendisi manuel olarak build edip push etmesi gerekir.
B. Container image'ı Cloud Scheduler build eder, ve Cloud Tasks onu saklar.
C. Source, bir Cloud Storage bucket'ında saklanır, ardından Cloud Build onu otomatik olarak bir container image'a build eder ve bu image'ı Artifact Registry'ye push eder; Cloud Run functions daha sonra function'ı çalıştırmak için bu image'a erişir.
D. Source, hiçbir container image build edilmeden, upload edildiği gibi doğrudan çalıştırılır.

**16.** Bir finans takımı, Cloud Run functions faturalarını neyin belirlediğini anlamak istiyor. Modüle göre fiyatlandırma modeli neye dayanır?

A. Kullanımdan bağımsız, deploy edilen function başına sabit bir aylık ücret.
B. Function invocation sayısı, function'ın ne kadar süre çalıştığı (compute time), ve outbound network trafiği için herhangi bir veri transferi ücreti — bir pay-as-you-go model.
C. Yalnızca Cloud Storage'da kullanılan source kodu depolama miktarı.
D. Gerçek invocation'larla ilgisiz, region başına sabit bir lisanslama ücreti.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: C.**
Cloud Run functions (2nd gen), function'ları arka planda Cloud Run üzerinde service'ler olarak deploy eder — bağımsız bir execution ürünü değildir. Bu ilişki, Cloud Run functions üzerine kurulu bir function'ın, zaten Cloud Run'ın container platformu üzerine kurulu olduğu için düz Cloud Run'a ya da Kubernetes'e taşınabilmesinin de tam olarak sebebidir.

**2. Doğru cevap: A.**
Cloud Run functions (2nd gen, eski adıyla Cloud Functions 2nd generation), function'ları Cloud Run service'leri olarak deploy eder, Eventarc ve Pub/Sub üzerinden tetiklenebilir. Cloud Run functions (1st gen, eski adıyla Cloud Functions 1st generation), daha sınırlı event trigger'lara ve yapılandırılabilirliğe sahip orijinal versiyondur — desteklenen dillerde ya da HTTP trigger'ların var olup olmadığında bir fark değildir.

**3. Doğru cevap: D.**
HTTP functions, HTTP(S) isteklerini almak için bir URL atanır, ve varsayılan olarak o isteklere gelen istekler kimlik doğrulama gerektirir; deployment anında unauthenticated isteklere izin vermeyi seçebilirsiniz. Bu, HTTP functions'ı dış bir sistemin function'ı doğrudan çağırdığı webhook/API senaryoları için doğru seçim yapar.

**4. Doğru cevap: B.**
Hiçbir dış çağıran olmadan bir Cloud Storage bucket'ına yeni bir objenin düşmesine otomatik olarak tepki vermek, tam olarak bir event trigger'a sahip event-driven functions'ın amacıdır — Cloud Storage, açıkça desteklenen event trigger kaynaklarından biridir.

**5. Doğru cevap: A.**
CloudEvent functions, CloudEvents endüstri standardı spesifikasyonu üzerine kuruludur ve kullanıcı function'larını kalıcı bir HTTP uygulamasında saran açık kaynaklı bir kütüphane olan Functions Framework'e kayıtlıdır; Cloud Run functions tarafından tüm dil runtime'larında kullanılır (Cloud Run functions 1st gen tarafından da .NET, Ruby ve PHP için kullanılır).

**6. Doğru cevap: C.**
Background functions, özellikle Cloud Run functions 1st gen tarafından Node.js, Python, Go ve Java runtime'larıyla kullanılan, daha eski stil bir event-driven implementasyondur — generation'dan ya da dilden bağımsız bir seçenek değildir.

**7. Doğru cevap: D.**
Varsayılan olarak, Cloud Run functions Node.js source kodunu, function dizininin kökünde `index.js` adlı bir dosyadan yükler (`package.json` içindeki `main` alanı ile farklı bir main dosya belirtilebilir). `main.py`, Node.js değil, Python kuralıdır, ve Node.js tam olarak desteklenen bir runtime'dır.

**8. Doğru cevap: B.**
`--entry-point` flag'i, Cloud Run function invoke edildiğinde çalıştırılan function'ı (ya da dile bağlı olarak class'ı) belirtir; source kodunuz bu entry point'i main dosyanızda ya da root package'ınızda tanımlamalıdır, ve bunu deploy anında açıkça belirtirsiniz.

**9. Doğru cevap: A.**
Cloud Run functions, HTTP functions için 60 dakikaya kadar, event-driven functions için ise 10 dakikaya kadar bir run time limitini destekler — doğrudan invoke edilen 45 dakikalık bir HTTP function, desteklenen HTTP limiti içindedir.

**10. Doğru cevap: D.**
Tek bir Cloud Run functions instance'ı, 1000'e kadar eşzamanlı isteği işleyebilir. Modül buna iki spesifik fayda atfeder: azalan cold start'lar ve scale olurken iyileşen genel latency, çünkü trafik patlamalarını karşılamak için daha az yeni instance başlatılması gerekir.

**11. Doğru cevap: C.**
Cloud Run functions'ı deploy etmek, hem Cloud Functions Developer IAM rolünü (ya da eşdeğerini) hem de Cloud Run functions runtime service account'u üzerinde Service Account User IAM rolünü gerektirir — yalnızca birinciye sahip olup ikinciye sahip olmamak, tam olarak bu türde bir yetki hatası üretir.

**12. Doğru cevap: B.**
Cloud Storage'dan deploy ederken, function'ın source dosyaları rastgele iç içe değil, zip dosyasının kökünde bulunmalıdır. Generation'a bağlı olarak, ya deploy eden hesap (Cloud Run functions 1st gen) ya da Cloud Run functions service agent'ı (Cloud Run functions/2nd gen), bucket'tan okuma iznine ihtiyaç duyar.

**13. Doğru cevap: A.**
Bir source repository'den deploy etmek, source yolunda `revisions/<revision_name>` kullanarak bir revision belirtmenize ve repository kökü dışında bir konuma işaret etmek için `paths/<source_directory_path>` eklemenize izin verir. Bunun çalışması için Cloud Run functions service agent'ının repository üzerinde Source Repository Reader (`roles/source.reader`) IAM rolüne sahip olması gerekir.

**14. Doğru cevap: D.**
Modül, latency ve availability'yi region seçimi için temel değerlendirmeler olarak çerçeveler, ama açıkça uygulamanızın kullandığı diğer Google Cloud ürünlerinin ve servislerinin konumunu da düşünmeniz gerektiğini ekler, çünkü birden fazla lokasyona yayılmış servisler kullanmak hem latency'yi hem fiyatlandırmayı etkileyebilir — yalnızca kullanıcılara yakınlık eksik bir tablodur.

**15. Doğru cevap: C.**
Function source kodunu deploy ettiğinizde, önce bir Cloud Storage bucket'ında saklanır; Cloud Build daha sonra bu source'u otomatik olarak bir container image'a build eder ve image'ı Artifact Registry'ye push eder, tamamen otomatik olarak, geliştiriciden doğrudan bir girdi gerekmeden. Cloud Run functions, function'ı çalıştırması gerektiğinde bu image'a Artifact Registry'den erişir.

**16. Doğru cevap: B.**
Modül, function invocation sayısına, function'ın ne kadar süre çalıştığına (compute time), ve outbound network trafiği için herhangi bir veri transferi ücretine dayalı bir pay-as-you-go fiyatlandırma modeli tanımlar — deployment sayısına bağlı sabit bir ücret, yalnızca depolamaya dayalı bir ücret, ya da region lisanslaması değil.
