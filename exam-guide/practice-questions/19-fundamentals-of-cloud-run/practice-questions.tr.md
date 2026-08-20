# Modül 19 — Fundamentals of Cloud Run: Pratik Sorular

Bu set, daha önceki genel bakışın ötesindeki Cloud Run temellerini kapsar: service-vs-job ayrımı, HTTPS desteği ve Cloud Run ile kodun arasındaki sorumluluk paylaşımı, 64-bit Linux binary gereksinimi, job'lar/task'lar/execution'lar/array job'lar, tetikleme yöntemleri (HTTPS, gRPC, WebSockets, Pub/Sub, Cloud Scheduler, Cloud Tasks, Eventarc), kaynak modeli (service, revision, container instance), tam container yaşam döngüsü (starting, serving requests, idle, shutting down, stopped) — container image'ların gerçekte nereden geldiği dahil —, autoscaling'in iç işleyişi (internal load balancer, scale to zero, cold start'lar/request queuing, minimum ve maximum instance'lar, maximum concurrency), ve access control (Google Cloud'un API modeli, IAM policy'ler/binding'ler/role'ler, bir servisi public yapmak, ve network ingress ayarları artı Serverless VPC Access).

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: bir container image'ın neden her başlangıçta Artifact Registry'den değil Cloud Run'ın internal storage'ından pull edildiği, idle bir container'ın neden throttle edilmesine rağmen hiçbir maliyete yol açmadığı, Cloud Run'ın neden isteği serve eden bir container'ı özünde asla durdurmadığı (iki spesifik istisna dışında), minimum instance'ların neden idle olmalarına rağmen para maliyeti çıkardığı, varsayılan max-instances configuration'ının neden platformun instance kotasından farklı olduğu, ve IAM yetkilendirmesi ile network ingress ayarlarının neden birleştirebileceğin bağımsız katmanlar olduğu.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Kodu bir Cloud Run service'i olarak çalıştırmakla bir Cloud Run job'ı olarak çalıştırmak arasındaki fark nedir?

A. Bir service, web isteklerine ya da event'lere yanıt veren kodu çalıştırmak için kullanılır ve sürekli çalışır; bir job ise iş yapan ve o iş bittiğinde quit eden kodu çalıştırmak için kullanılır — ikisi de aynı ortamda çalışır ve aynı Google Cloud entegrasyonlarını kullanabilir.
B. Bir service her zaman altında bir Kubernetes cluster'ı gerektirir, bir job ise hiçbir container olmadan doğrudan çıplak bir Compute Engine VM'inde çalışır.
C. Bir job web isteklerine sonsuza kadar yanıt verir, bir service ise tek bir iş birimini gerçekleştirip sonlanır — gerçek amaçlarının tersi.
D. Service ve job, tamamen aynı altındaki kaynağın iki farklı ismidir, ve aralarında seçim yapmanın kodun nasıl çalıştığı üzerinde hiçbir etkisi yoktur.

**2.** Bir Cloud Run service'i HTTPS'i handle etmek için hangi altyapıyı sağlar, ve senin uygulama kodun gerçekte neden sorumludur?

A. Uygulama kodun kendi TLS terminasyonunu ve sertifika yönetimini implement etmelidir; Cloud Run yalnızca hiçbir şeyi decrypt etmeden ham şifreli byte'ları forward eder.
B. Cloud Run yalnızca düz HTTP'yi destekler, hiçbir HTTPS yeteneği yoktur, bu yüzden şifreleme tamamen harici, ayrıca yapılandırılmış bir proxy tarafından handle edilmelidir.
C. Cloud Run, güvenilir bir HTTPS endpoint'i çalıştırmak için gereken altyapıyı sağlar — benzersiz bir `*.run.app` subdomain'inde (ya da özel bir domain'de) geçerli bir TLS sertifikası ve endpoint provizyonlar, ve gelen istekleri handle eder, decrypt eder, ve forward eder, ayrıca WebSockets, HTTP/2, ve gRPC'yi destekler; senin sorumluluğun sadece kodunun bir TCP portunu dinlemesini ve HTTP isteklerini handle etmesini sağlamaktır.
D. HTTPS desteği, her deployment'tan önce kendi sertifika dosyalarını Cloud Storage'a elle yüklemeni gerektirir, otomatik provizyonlama mevcut değildir.

**3.** Cloud Run'a deploy edilen bir uygulamanın hangi programlama dilinde yazılabileceği konusundaki tek gerçek gereksinim nedir?

A. Uygulamalar, birinci sınıf Google Cloud client kütüphane desteğine sahip bir dilde yazılmalıdır, aksi halde Cloud Run için hiç containerized edilemezler.
B. Bir uygulama, 64-bit bir Linux binary'sine compile edilebiliyor ve bir container image'a paketlenebiliyorsa, herhangi bir programlama dilinde geliştirilip Cloud Run'da çalıştırılabilir.
C. Uygulamalar, kullanılan dilden bağımsız olarak hiçbir sistem paketinden ya da kütüphane bağımlılığından kaçınmalıdır.
D. Uygulamalar 32-bit binary'lerle sınırlıdır, çünkü Cloud Run'ın altındaki altyapısı 64-bit çalıştırmayı desteklemez.

**4.** Bir Cloud Run job'ı task'lar ve execution'larla nasıl ilişkilidir, ve bir Array job nedir?

A. Bir job execution, paralel çalışan diğer task'ların durumundan bağımsız olarak, ilk task'ı bittiği anda başarılı sayılır.
B. Bir task aynı anda birden fazla container instance'ı çalıştırabilir, bir job execution ise aynı anda tam olarak bir task çalıştırmakla sınırlıdır.
C. Job'lar, task'lar, ve execution'lar, birbiriyle yapısal bir ilişkisi olmayan, ilgisiz Cloud Run kavramlarıdır, ve Array job'lar artık mevcut olmayan, kullanımdan kaldırılmış bir özelliktir.
D. Bir job, verilen bir job execution'da paralel olarak çalıştırılan bir ya da daha fazla bağımsız task'tan oluşur, her task bir container instance çalıştırır; bir execution'ın başarılı olması için execution'daki tüm task'ların başarıyla tamamlanması gerekir (başarısızlıkları handle etmek için timeout ve retry yapılandırılabilir), ve birden fazla özdeş container instance'ı çalıştıran bir job'a Array job denir — örneğin Cloud Storage'daki birçok dosyayı paralel olarak işlemek için kullanışlıdır.

**5.** Bir ekibin üç ihtiyaç için tetikleme yöntemi seçmesi gerekiyor: yüksek veri yükleriyle yüksek performanslı dahili mikroservisten-mikroservise iletişim, servisler arasında garantili teslimatlı asenkron mesaj teslimatı, ve bir cron job'a benzer şekilde bir servisi tekrarlayan bir zamanlamaya göre güvenle tetiklemek. Her ihtiyaca hangi yöntemler uyar?

A. Üç durum için de HTTPS, çünkü gRPC, Pub/Sub, ve Cloud Scheduler'ın düz HTTPS istekleriyle gereksiz (redundant) olduğu ve belirgin bir avantaj sunmadığı belirtiliyor.
B. Üç durum için de WebSockets, çünkü modülün WebSockets'i Cloud Run'ın ek configuration olmadan gerçekten desteklediği tek tetikleme yöntemi olarak tanımladığı belirtiliyor.
C. İlk ihtiyaç için gRPC (protocol buffer kullandığı için, REST çağrılarından 7 kata kadar hızlı, dahili mikroservisler ve yüksek veri yükleri için uygun), ikinci ihtiyaç için Pub/Sub push subscription'ları (garantili teslimatlı asenkron mesajlar, servis endpoint'ine HTTP istekleri olarak forward edilir), ve üçüncü ihtiyaç için Cloud Scheduler (bir servisi bir zamanlamaya göre güvenle tetikleyen, tam yönetilen bir cron job scheduler'ı).
D. Üç durum için de Eventarc, çünkü Eventarc'ın gRPC, Pub/Sub, ve Cloud Scheduler ihtiyacını tamamen ortadan kaldıran bir üst küme olduğu belirtiliyor.

**6.** Cloud Run'ın ana kaynağı nedir, ve Google Cloud region'ları ve zone'larıyla nasıl ilişkilidir?

A. Cloud Run'ın ana kaynağı bir service'tir. Her service belirli bir Google Cloud region'unda bulunur, ve servisler regional bir kaynak olsa da, container instance'ları o region içindeki herhangi bir zone'da başlayabilir — redundancy için, yüksek trafikli ve çok sayıda instance'a sahip servisler birden fazla zone'a yayılır, böylece bir zone sorun yaşasa bile servis isteklere hizmet vermeye devam eder.
B. Ana kaynak container instance'ının kendisidir, ve herhangi bir belirli region ya da zone ile hiçbir ilişkisi olmadığı belirtiliyor.
C. Ana kaynak bir job'dır, ve servislerin HTTP tetiklemeli kullanım alanları için job'ın etrafında yalnızca isteğe bağlı bir sarmalayıcı olduğu belirtiliyor.
D. Ana kaynak, tek bir ana region'ı olmadan her Google Cloud region'unda aynı anda ve aynı şekilde var olan global bir nesnedir.

**7.** Bir Cloud Run revision'ı neden oluşur, ve modül revision'ların neden immutable olduğunu söylüyor?

A. Bir revision yalnızca bir container image'dan oluşur, herhangi bir türde environment configuration ona paketlenmemiştir.
B. Bir revision, sadece environment variable'lar (container image değil) değiştirilirse, oluşturulduktan sonra yerinde düzenlenebilir.
C. Bir revision yalnızca bir servis ilk kez deploy edildiğinde oluşturulur, ve aynı servise yapılan sonraki deployment'lar hiçbir zaman yeni bir revision oluşturmaz.
D. Bir revision, belirli bir container image ile environment variable'lar, memory limit'leri, ya da concurrency değeri gibi environment ayarlarından oluşur; oluşturulduktan sonra bir revision değiştirilemez — aynı servise farklı bir container image deploy etmek ya da configuration'ı değiştirmek, bunun yerine tamamen yeni bir revision oluşturur, ve istekler otomatik olarak en son sağlıklı revision'a yönlendirilir.

**8.** Belirli bir Cloud Run service revision'ına gelen istekleri ne handle eder, ve bu bir job'ın execution yapısıyla nasıl ilişkilidir?

A. Bir revision'a gelen istekler, hiçbir noktada bir container instance'ı olmadan doğrudan Google Cloud console tarafından handle edilir.
B. Bir container instance, bir service revision'ına gelen istekleri handle eder (ve concurrency ayarına tabi olarak aynı anda birçok istek alabilir); benzer şekilde, job'lar için, bir job execution job'ın tüm task'larını başlatır, ve her task bir container instance çalıştırır.
C. Bir projedeki her servis genelindeki her revision'ı, global olarak paylaşılan, sürekli çalışan tek bir container instance handle eder.
D. Job execution'ları ve container instance'ları tamamen ilgisiz kavramlardır, ve bir job'ın task'larının hiçbir zaman herhangi bir container içinde çalışmadığı belirtiliyor.

**9.** Bir Cloud Run container'ının yaşam döngüsündeki relevant state'ler nelerdir, ve "starting" fazı hangi dört adımı içerir?

A. Yaşam döngüsünün hiç tanımlanmış bir starting fazı yoktur — Cloud Run'ın, herhangi bir koşulda sıfır başlangıç gecikmesiyle, bir container image deploy edildiği anda isteklere hizmet vermeye anında hazır olduğu belirtiliyor.
B. Yaşam döngüsünde yalnızca iki state vardır: "Running" ve "Not Running", starting, idle, ya da shutting-down davranışı arasında hiçbir ayrım yapılmaz.
C. State'ler: Starting, Serving requests, Idle, Shutting down, ve Stopped'dır. Starting, Cloud Run bir container image pull ettiğinde başlar ve container istekleri serve etmeye başladığında biter, ve dört adım içerir: image'dan container'ın root dosya sistemini materialize etmek, entrypoint programını (uygulamanı) çalıştırmak, uygulamanın hazır olup olmadığını kontrol etmek için 8080 portunu sürekli probe etmek (yapılandırılabilir, HTTP/TCP/gRPC startup ve liveness probe desteğiyle), ve uygulama TCP bağlantılarını kabul ettiğinde gelen istekleri forward etmek.
D. Starting'in dört adımı, sırasıyla: shutting down, stopping, restarting, ve re-materializing'dir — herhangi bir istek serve edilmeden önce tekrarlanan bir döngü.

**10.** Cloud Run bir container image pull ettiğinde, bunun gerçekleştiği iki farklı olay nedir, ve her durumda pull ettiği iki farklı kaynak nedir?

A. Cloud Run, hem yeni bir image deploy ederken hem de yeni bir container başlatırken, herhangi bir noktada ayrı bir internal storage katmanı olmadan her zaman doğrudan Artifact Registry'den pull eder.
B. Cloud Run'daki container image'ların, platformun kendisine önceden derlendiği belirtiliyor, bu yüzden bir projedeki herhangi bir Cloud Run servisinin ilk deployment'ından sonra, herhangi bir registry ya da storage'dan hiçbir pull işlemi gerçekleşmez.
C. Cloud Run, proje başına bir container image'ı yalnızca bir kez pull eder, fiilen deploy edilen image'dan bağımsız olarak, gelecekteki her servis için bunu kalıcı olarak cache'ler.
D. Bir container image'ı ilk kez deploy ettiğinde, Cloud Run onu Artifact Registry'den pull edip kendi internal storage'ına kopyalar (büyük image'ların küçükler kadar hızlı yüklenmesi için optimize edilmiştir); Cloud Run daha sonra yeni bir container başlattığında, image'ı bunun yerine o internal storage'dan pull eder — bu ayrıca servisi Artifact Registry başarısızlıklarından ya da yanlışlıkla silinen bir image'dan izole eder.

**11.** Idle bir Cloud Run container'ının tanımlayıcı özellikleri nelerdir, ve bu state hangi sınırlamaları getirir?

A. Idle bir container istekleri serve etmez, hiçbir ücrete yol açmaz, ve CPU'su neredeyse sıfıra throttle edilir (uygulamanın çok yavaş çalışmasına neden olur); her an kapatılabilir (bir lifecycle hook graceful shutdown'a izin verse de), throttle edilmişken arka plan görevleri güvenilir bir şekilde gerçekleştirilemez (bunun yerine Cloud Tasks kullan), ve üçüncü taraflara yapılan network istekleri başarısız olma olasılığı yüksektir.
B. Idle bir container, aktifken olduğu kadar hızlı istekleri serve etmeye devam eder, her zaman tam faturalandırma ücretlerine yol açar, ve bir kez başladıktan sonra hiçbir koşulda hiç kapatılamaz.
C. Idle bir container, her açıdan durdurulmuş bir container ile aynı olarak tanımlanır, iki state arasında anlamlı bir ayrım yoktur.
D. Idle bir container, herhangi bir shutdown sinyali gönderilmeden önce bile, idle olduğu anda bellek içi durumunun tamamını anında kaybeder.

**12.** Idle bir container tekrar bir isteği handle etmeye başladığında, ne kadar hızlı tamamen responsive hale gelir, ve minimum instances ayarı bununla nasıl ilişkilidir?

A. Idle bir container serving requests'e her geri döndüğünde, herhangi bir configuration'dan bağımsız olarak, zorunlu, sabit 30 saniyelik bir gecikme vardır.
B. Bir container idle'dan serving requests'e geçtiğinde, Cloud Run CPU'sunu unthrottle eder ve tam erişimi anında geri verir — uygulama ve kullanıcılar hiçbir gecikme fark etmez; trafik sıçramalarını handle etmek ve cold start'ları minimize etmek için, Cloud Run bazı instance'ları en fazla 15 dakika idle tutabilir, ve minimum instances ayarı Cloud Run'ın her zaman belirli sayıda instance'ı istekleri serve etmeye hazır tutmasını sağlar.
C. Bir container idle olduğunda, bir daha asla serving requests'e geçemez — sonraki her istek için her zaman sıfırdan yepyeni bir container başlatılmalıdır.
D. Minimum instances ayarının idle-to-serving geçişleri üzerinde hiçbir etkisi olmadığı, yalnızca faturalandırmayı etkilediği ve gecikmeyle hiçbir ilgisi olmadığı belirtiliyor.

**13.** Bir container idle'daysa ve Cloud Run onu kapatmaya karar verirse, varsayılan olarak ne olur, ve bir uygulama bunun yerine düzgün bir şekilde kapanmak için ne yapabilir?

A. Bir uygulamanın shutdown davranışını hiçbir şekilde etkilemesinin bir yolu yoktur — Cloud Run, önceden hiçbir sinyal göndermeden idle container'ları her zaman zorla sonlandırır.
B. Cloud Run, uygulama herhangi bir sinyali handle etsin ya da etmesin, herhangi bir idle container'ı kapatmadan önce her zaman tam olarak 60 saniye bekler.
C. Graceful shutdown, uygulamanın yakalayıp süresiz olarak erteleyebileceği bir SIGKILL sinyaliyle tetiklenir, platform tarafından hiçbir zaman sınırı dayatılmaz.
D. Varsayılan olarak, bir container kapatıldığında sadece ortadan kaybolur; ama uygulama SIGTERM sinyalini handle ediyorsa, kaldırılmadan önce temizlik yapması için 10 saniyesi olur — örneğin açık TCP bağlantılarını, dosya tanımlayıcılarını, ve veritabanı bağlantılarını kapatmak, verisi olan buffer'ları flush etmek, ve ileride debug etmeye yardımcı olacak bir log yazmak.

**14.** Cloud Run, aktif olarak istekleri serve eden bir container'ı hangi spesifik koşullarda durdurur, ve bu gerçekleştiğinde devam eden (in-flight) isteklere ne olur?

A. Cloud Run, uygulama davranışından bağımsız olarak her saat sabit bir zamanlamada serving container'ı durdurur, ve devam eden istekler her zaman önceden başarıyla tamamlanır, hiçbir etkisi olmaz.
B. Cloud Run, CPU utilization'ı %60'ı aştığı anda serving container'ı durdurur, tüm devam eden istekler otomatik olarak sıfır kesinti süresiyle önceden ısıtılmış bir yedek container'a yönlendirilir.
C. Normal koşullar altında, Cloud Run istekleri serve ederken bir container'ı asla durdurmaz; iki istisna, uygulamanın çıkması (örneğin, uygulama kodundaki bir hata nedeniyle) ya da container'ın memory limit'ini aşmasıdır (varsayılan 512 MiB, 32 GiB'a kadar yapılandırılabilir) — bir container istekleri handle ederken durursa, tüm devam eden istekler sonlandırılır ve bir hatayla başarısız olur, ve bir yedek container başlarken yeni istekler beklemek zorunda kalabilir.
D. Cloud Run, yeni bir revision deploy edildiğinde her zaman serving container'ı durdurur, herhangi bir eski revision'a bağlı her devam eden isteği anında ve koşulsuz olarak sonlandırır.

**15.** Gelen istekleri bir service revision'ının container instance'ları arasında hangi bileşen dağıtır, ve ham gelen istek oranının ötesinde, hangi faktörler Cloud Run'ın kaç instance tuttuğunu etkiler?

A. İstekler, herhangi bir load balancing bileşeni olmadan tamamen rastgele dağıtılır, ve instance sayısını etkileyen tek faktör projenin kayıtlı kullanıcı sayısıdır.
B. Internal bir Application Load Balancer, istekleri instance havuzu arasında dağıtır, tümü meşgulse instance ekler ve talep azaldıkça bazılarını kapatır; istek oranının ötesinde, instance sayısı ayrıca istekleri işlerken mevcut instance'ların CPU utilization'ından (yaklaşık %60 hedef), maksimum concurrency ayarından, ve minimum/maksimum instance sayısı ayarlarından etkilenir.
C. Tek, elle yapılandırılmış bir DNS round-robin kaydı tüm istekleri dağıtır, ve instance sayısı deployment anında kalıcı olarak sabittir, runtime ayarlaması mümkün değildir.
D. Dağıtım yalnızca her container image'ın içine elle kurulması gereken üçüncü parti bir load balancer tarafından handle edilir, ve Cloud Run'ın kendisinin instance sayısı kararlarında hiçbir rolü yoktur.

**16.** Bir Cloud Run servisi için "scale to zero" ne anlama gelir, ve neden ekonomik olarak çekici olarak tanımlanıyor?

A. Servise gelen istek yoksa, geriye kalan son container instance'ı bile kapatılır — bu scale to zero'dur, ve ekonomik olarak çekicidir çünkü boşta bekleyen container instance'ları için ödeme yapmazsın; yeni bir istek geldiğinde yeni bir instance istek üzerine başlar.
B. Scale to zero, servisin HTTPS endpoint'inin bir hareketsizlik döneminden sonra kalıcı olarak silindiği, onu geri getirmek için tamamen yeni bir deployment gerektirdiği anlamına gelir.
C. Scale to zero, servisin maksimum instance sayısının kalıcı olarak sıfırda sabitlendiği, elle yeniden yapılandırılana kadar bir daha hiçbir isteği serve etmesini önlediği anlamına gelir.
D. Scale to zero, bugün her Cloud Run servisinde zorunlu minimum instance'larla tamamen değiştirilmiş, eski bir özellik olarak tanımlanıyor.

**17.** Bir servis sıfıra ölçeklendikten hemen sonra gelen ilk birkaç isteğe ne olur, ve bu olguya genellikle ne denir?

A. Bu istekler, herhangi bir queuing davranışı olmadan sessizce ve kalıcı olarak düşürülür, client'ın sıfırdan elle yeniden denemesini gerektirir.
B. Bu istekler, ilk container instance'ı başlarken kuyruğa alınır — bu genellikle bir "cold start" olarak bilinir.
C. Bu istekler otomatik olarak bir HTTP 500 hatasıyla reddedilir, ve Cloud Run bu nedenle bir servisin hiçbir zaman sıfıra ölçeklenmesine izin verilmemesini önerir.
D. Bu istekler container'ı tamamen atlar ve hiçbir zaman uygulama koduna ulaşmadan doğrudan internal load balancer tarafından serve edilir.

**18.** Minimum bir instance sayısı ayarlamak, bu instance'lar boşta oturuyor olabileceği göz önüne alındığında, faturalandırma konusunda gerçekte neyi değiştirir?

A. Minimum instance'ların, idle durumundan bağımsız olarak, aktif olarak istekleri serve eden instance'larla aynı şekilde, tam CPU oranında faturalandırıldığı belirtiliyor.
B. Minimum instance'ları ayarlamak, servis sıfırdan ölçeklenirken ortaya çıkan tüm önceki ücretleri, bir faturalandırma kredisi olarak geriye dönük olarak iade eder.
C. Minimum instance'ların, Cloud Run'ın herhangi bir nedenle idle tutulan herhangi bir instance için hiçbir zaman faturalandırma yapmadığı belirtildiği için, her koşulda tamamen ücretsiz olduğu belirtiliyor.
D. Minimum instances özelliği nedeniyle çalışır tutulan idle instance'lar, istekleri serve etmiyor olsalar bile, hâlâ faturalandırma maliyetine yol açar — özellik, bu devam eden maliyeti, cold start'lardan kaçınarak azaltılan gecikmeyle takas eder.

**19.** Bir Cloud Run servisinin scale out edecek şekilde yapılandırıldığı varsayılan maksimum container instance sayısı nedir, ve bu platformun genel instance kotasından nasıl farklıdır?

A. Varsayılan olarak, Cloud Run servisleri maksimum 100 instance'a kadar scale out edecek şekilde yapılandırılmıştır — bu senin servisinin configuration ayarıdır, servis başına platformun varsayılan 1.000 instance'lık kota sınırından (gerekirse bir kota artışı talep edebileceğin) farklıdır.
B. Varsayılan maksimum instance configuration'ı ile platformun instance kotasının, aralarında hiçbir ayrım olmadan, tam olarak aynı sabit sayı olduğu belirtiliyor.
C. Hiç varsayılan bir maksimum instance configuration'ı yoktur — her Cloud Run servisinin, deploy edilebilmesi için bu değerin açıkça ayarlanmış olması gerekir, aksi halde deployment başarısız olur.
D. Platformun instance kotasının, varsayılan maksimum instance configuration'ından daha düşük olduğu, kotanın neredeyse her gerçek deployment'ta bağlayıcı kısıtlama olduğu belirtiliyor.

**20.** Instance başına maksimum eşzamanlı istek ayarının varsayılan ve maksimum değeri nedir, ve bir ekip bunu hangi koşullarda 1'e düşürmeyi düşünmelidir?

A. Varsayılan, instance başına 1 istektir, artırmanın bir yolu yoktur, bu da yüksek throughput'lu servisleri Cloud Run'da mimari olarak imkansız kılar.
B. Varsayılan sınırsızdır, ve ayar yalnızca pratikte anlamlı bir üst sınırı olmayan bir değeri düşürmek için vardır.
C. Varsayılan, instance başına 80 eşzamanlı istektir (maksimum 1.000'e kadar yapılandırılabilir); bir ekip, her istek container'ın mevcut CPU'sunun ya da belleğinin çoğunu kullanıyorsa, uygulama kodu aynı anda birden fazla isteği handle etmek üzere tasarlanmamışsa, ya da kod birden fazla istek arasında paylaşılamayan global state'e dayanıyorsa, bunu 1'e ayarlamayı düşünmelidir.
D. Varsayılan 1.000'dir ve uygulamanın concurrency güvenliğinden bağımsız olarak, hiçbir configuration'da 500'ün altına düşürülemez.

**21.** Modül, Google Cloud'u bir platform olarak nasıl tanımlıyor, ve bir container image'ı Cloud Run'a deploy etmek gerçekte neyin bir örneğidir?

A. Google Cloud, sanal kaynaklar oluşturmanı ve yönetmeni sağlayan bir API koleksiyonu olarak tanımlanıyor — web console, gcloud CLI, Terraform, ya da client kütüphaneleri aracılığıyla erişilir — ve bir container image deploy etmek (örn. `gcloud run deploy` çalıştırmak), bu durumda `run.googleapis.com`'daki Cloud Run Management API'sine yapılan, bir API çağrısının kendisinin bir örneğidir.
B. Google Cloud, hiçbir programlanabilir arayüzü olmayan, önceden build edilmiş sabit bir sanal makine image'ları kümesi olarak tanımlanıyor, ve bir container image deploy etmek tamamen elle yapılan, çevrimdışı bir donanım provizyonlama süreci olarak tanımlanıyor.
C. Google Cloud'un yalnızca Terraform üzerinden erişilebilir olduğu, gcloud CLI, console, ve client kütüphanelerinin artık çalışmayan, kullanımdan kaldırılmış eski araçlar olarak tanımlandığı belirtiliyor.
D. Bir container image deploy etmenin, herhangi bir API katmanı olmadan doğrudan fiziksel sunucu donanımıyla iletişim kurarak tüm Google Cloud API'lerini tamamen atladığı belirtiliyor.

**22.** Bir IAM policy binding'i nedir, ve member, role, ve permission IAM'in modelinde birbiriyle nasıl ilişkilidir?

A. Bir permission, hiçbir role ya da binding olmadan doğrudan erişim verir, bu da role'leri ve policy binding'lerini işlevsel bir etkisi olmayan, tamamen dekoratif kavramlar haline getirir.
B. Bir policy binding, bir member'ı (kimlik) tek bir role'e bağlar, ve bir member'ın bir IAM policy içinde birden fazla policy binding'i (dolayısıyla birden fazla role'ü) olabilir; bir role, member kimliğinin bir kaynak üzerinde belirli aksiyonlar gerçekleştirmesini sağlayan bir izinler kümesi içerir — örneğin, Pub/Sub Publisher role'ü `pubsub.topics.publish` iznini içerir.
C. Bir policy binding, bir kez kullanıldıktan sonra gelecekteki hiçbir yetkilendirme kontrolü için tekrar referans verilemeyen, tek seferlik bir aksiyondur.
D. Role'ler ve permission'lar, IAM altında birbirinin yerine kullanılan, yapısal bir farkı olmayan aynı kavramlardır, ve bir member hiçbir zaman birden fazla role'e aynı anda sahip olamaz.

**23.** Bir Cloud Run servisini kimlik doğrulama olmadan public erişilebilir yapmak için hangi role, hangi member'a verilmelidir — ve bireysel servislere karşı tüm bir projeye erişimi kontrol etmenin iki yolu nedir?

A. Bir servisi public yapmak, proje için IAM'i tamamen devre dışı bırakmayı gerektirir, ve bireysel servislere karşı tüm projeye erişimin, hiçbir ayrım mevcut olmadan aynı şekilde kontrol edildiği belirtiliyor.
B. Bir servisi public yapmak, rastgele oluşturulan bir service account'a Owner role'ünü vermeyi gerektirir, ve erişim yalnızca tüm bir Google Cloud organizasyonu düzeyinde kontrol edilebilir, hiçbir zaman proje ya da servis başına değil.
C. Bir servisi public yapmak, servisi Cloud Run'dan tamamen kaldırıp ayrı, kimlik doğrulanmamış bir platformda yeniden deploy etmek anlamına gelir, çünkü IAM hiçbir koşulda public erişime izin verecek şekilde yapılandırılamaz.
D. Cloud Run Invoker role'ünü (`roles/run.invoker`) `allUsers` member'ına verirsin (örn. `gcloud run services add-iam-policy-binding my-service --member="allUsers" --role="roles/run.invoker"` ile, ya da `--allow-unauthenticated` deploy flag'i ile); bireysel bir servise ya da job'a erişimi kontrol etmek için `gcloud run services/jobs add-iam-policy-binding`/`remove-iam-policy-binding` ile principal'lar ekler/kaldırırsın, projedeki tüm servis ve job'lara erişimi kontrol etmek için ise `gcloud projects add-iam-policy-binding` ile project-level IAM kullanırsın.

**24.** Cloud Run'ın üç network ingress ayarı nedir, ve Serverless VPC Access, ingress'i yapılandırmaktan nasıl farklıdır?

A. Ingress ayarlarının ve Serverless VPC Access'in, tek, aynı komutla yapılandırılan, iki farklı isim altındaki tam olarak aynı özellik olduğu belirtiliyor.
B. Cloud Run'da yalnızca bir ingress ayarı vardır ("All"), ve Serverless VPC Access'in kullanımdan kaldırılmış, işlevsiz bir özellik olduğu belirtiliyor.
C. Üç ingress ayarı şunlardır: "All" (en az kısıtlayıcı — doğrudan internetten dahil, tüm istekleri kabul eder), "Internal" (en kısıtlayıcı — yalnızca internal HTTP(S) load balancer, izin verilen VPC Service Controls perimeter kaynakları, aynı proje/perimeter'daki VPC network'leri, ve Cloud Tasks/Eventarc/Pub/Sub/Workflows gibi aynı proje/perimeter'daki servisler), ve "Internal and Cloud Load Balancing" (Internal'ın izin verdikleri artı external HTTP(S) load balancer, ama yine de doğrudan internetten değil). Bunlar, servisin kendisine gelen (inbound) erişimi kontrol eder; Serverless VPC Access ise bunun yerine bir Cloud Run servisini ya da job'ını, VM instance'ları ya da Memorystore gibi kaynaklara, o trafiği internete maruz bırakmadan erişmek için, (bir connector aracılığıyla internal DNS/internal IP adresleri kullanarak) giden (outbound) yönde bir VPC network'üne bağlar — bu iki mekanizma bağımsızdır ve birlikte kullanılabilir.
D. Ingress ayarlarının yalnızca Cloud Run job'larına uygulandığı, servislere hiç uygulanmadığı, ve Serverless VPC Access'in ingress ayarlarına olan ihtiyacı tamamen ortadan kaldırdığı belirtiliyor.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Cloud Run'da, kod sürekli olarak bir service ya da bir job olarak çalışabilir. Service'ler, web isteklerine ya da event'lere yanıt veren kodu çalıştırmak için kullanılır, job'lar ise iş yapan ve o iş bittiğinde quit eden kodu çalıştırmak için kullanılır. İkisi de aynı ortamda çalışır ve diğer Google Cloud servisleriyle aynı entegrasyonları kullanabilir.

**2. Doğru cevap: C.**
Bir Cloud Run servisi, güvenilir bir HTTPS endpoint'i çalıştırmak için gereken altyapıyı sağlar: `*.run.app`'in benzersiz bir subdomain'inde (özel domain'ler yapılandırılabilir) geçerli bir TLS sertifikası ve HTTPS endpoint'i provizyonlar, ve gelen istekleri handle eder, decrypt eder, ve uygulamaya forward eder, ayrıca WebSockets, HTTP/2, ve gRPC'yi de destekler. Uygulamanın kendi sorumluluğu, sadece bir TCP portunu dinlemek ve HTTP isteklerini handle etmektir.

**3. Doğru cevap: B.**
Container'lar çalıştırmak, Cloud Run'ın büyük bir avantajıdır — uygulamalar, herhangi bir programlama dilinde geliştirilip Cloud Run'da çalıştırılabilir, tek şart bunların 64-bit bir Linux binary'sine compile edilebilir ve bir container image'a paketlenebilir olmasıdır.

**4. Doğru cevap: D.**
Bir job, verilen bir job execution'da paralel olarak çalıştırılan bir ya da daha fazla bağımsız task'tan oluşur, her task bir container instance çalıştırır. Bir job execution'daki tüm task'ların, execution'ın başarılı olması için başarıyla tamamlanması gerekir, ve task başarısızlıklarını handle etmek için timeout ile belirtilen sayıda retry yapılandırılabilir. Birden fazla özdeş container instance'ı çalıştıran job'lara Array job denir — örneğin, Cloud Storage'daki birden fazla görsel dosyasını aynı anda işlemek için.

**5. Doğru cevap: C.**
gRPC, protocol buffer kullandığı için (REST çağrılarından 7 kata kadar hızlı), dahili mikroservisler arasındaki yüksek performanslı, yüksek veri yüklü iletişim için iyi bir seçimdir. Pub/Sub push subscription'ları, servisler arasında garantili teslimatlı asenkron mesajlar göndermeni/almanı sağlar, bunları bir Cloud Run servisinin endpoint'ine HTTP istekleri olarak forward eder. Cloud Scheduler, cron job kullanmaya benzer şekilde bir Cloud Run servisini bir zamanlamaya göre güvenle tetikleyebilen, tam yönetilen bir cron job scheduler'ıdır.

**6. Doğru cevap: A.**
Service, Cloud Run'ın ana kaynağıdır, ve her service, Cloud Run'ın kullanılabilir olduğu belirli bir Google Cloud region'unda bulunur. Servisler regional bir kaynaktır, ama container instance'ları o region içindeki herhangi bir zone'da başlayabilir; redundancy için, yüksek trafikli ve çok sayıda container instance'ı olan servisler birden fazla zone'a yayılır, böylece Cloud Run bir zone'da sorun yaşasa bile servis isteklere hizmet vermeye devam eder.

**7. Doğru cevap: D.**
Bir uygulama container image'ının Cloud Run'a her deploy edilişi, environment variable'lar, memory limit'leri, ya da concurrency değeri gibi environment ayarlarıyla birlikte belirli bir container image'dan oluşan bir service revision oluşturur. Revision'lar immutable'dır — bir kez oluşturulduktan sonra bir revision değiştirilemez; aynı servise farklı bir container image deploy etmek (ya da configuration'ı değiştirmek) bunun yerine yeni bir revision oluşturur, ve istekler mümkün olduğunca hızlı bir şekilde otomatik olarak en son sağlıklı service revision'a yönlendirilir.

**8. Doğru cevap: B.**
İstek alan her service revision, bu istekleri handle etmek için gereken sayıda container instance'ıyla otomatik olarak ölçeklendirilir, ve bir container instance aynı anda birçok istek alabilir (concurrency ayarına tabi olarak). Benzer şekilde, bir job çalıştırıldığında, tüm job task'larının başlatıldığı bir job execution oluşturulur, ve her task bir container instance çalıştırır.

**9. Doğru cevap: C.**
Cloud Run'daki bir container'ın relevant state'leri Starting, Serving requests, Idle, Shutting down, ve Stopped'dır. Starting fazı, Cloud Run bir container image pull ettiğinde başlar ve container istekleri serve etmeye başladığında biter, ve bir container'ı başlatmak dört adım gerektirir: Cloud Run, container image'ı materialize ederek container'ın root dosya sistemini oluşturur, entrypoint programını (uygulamayı) çalıştırır, uygulamanın hazır olup olmadığını kontrol etmek için 8080 portunu sürekli probe eder (yapılandırılabilir, HTTP/TCP/gRPC startup health check ve liveness probe desteğiyle), ve uygulama TCP bağlantılarını kabul etmeye başladığında gelen web isteklerini container'a forward eder.

**10. Doğru cevap: D.**
Cloud Run'ın bir container image pull ettiği, iki farklı kaynaktan gelen iki farklı olay vardır: bir container image'ı ilk kez deploy ettiğinde, Cloud Run onu Artifact Registry'den pull edip kopyalar, sonra kendi internal storage'ında saklar (büyük container image'ların küçükler kadar hızlı yüklenmesi için optimize edilmiştir); bundan sonra Cloud Run her yeni container başlattığında, image'ı bunun yerine o internal storage'dan pull eder. Cloud Run image'ı bu şekilde kopyaladığı için, bu ayrıca servisi Artifact Registry'deki başarısızlıklardan ya da deploy edilmiş bir container image'ının yanlışlıkla Artifact Registry'den kaldırılmasından izole eder.

**11. Doğru cevap: A.**
Idle bir container istekleri serve etmez ve hiçbir ücrete yol açmaz, ama CPU'su neredeyse sıfıra throttle edilir, bu da uygulamanın çok yavaş çalıştığı anlamına gelir. Her an kapatılabilir, ama graceful shutdown için bir lifecycle hook mevcuttur. CPU throttle edildiği için, container üzerinde güvenilir bir şekilde arka plan görevleri gerçekleştirilemez (bunun yerine görevleri zamanlamak için Cloud Tasks kullanılabilir), ve container idle'dayken üçüncü taraflara yapılan network istekleri başarısız olma olasılığı yüksektir.

**12. Doğru cevap: B.**
Bir container idle olduktan sonra bir istek handle ettiğinde, idle state'inden serving requests'e geçer; bu geçiş sırasında, Cloud Run container'ın CPU'sunu unthrottle eder ve tam erişimi anında geri verir, uygulama ya da kullanıcıları tarafından fark edilebilecek hiçbir gecikme olmadan. Trafik sıçramalarını handle etmek ve cold start'ları minimize etmek için, Cloud Run bazı instance'ları en fazla 15 dakika idle tutabilir, ve minimum instances ayarı Cloud Run'ın her zaman belirli sayıda container instance'ını istekleri serve etmeye hazır tutmasını sağlar.

**13. Doğru cevap: D.**
Varsayılan olarak, bir container kapatıldığında sadece ortadan kaybolur. Ancak, bir uygulama, kapanmanın yakında olacağı konusunda onu uyaran ve container kaldırılmadan önce temizlik yapması için 10 saniye veren SIGTERM sinyalini handle edecek şekilde build edilebilir — örneğin veritabanı bağlantılarını kapatmak ya da veri içeren buffer'ları flush etmek, daha genel olarak açık TCP bağlantılarını/dosya tanımlayıcılarını/veritabanı bağlantılarını kapatmak, herhangi bir batch'lenmiş veriyi flush etmek, ve ileride debug etmeye yardımcı olacak bir log yazmak.

**14. Doğru cevap: C.**
Cloud Run, normal koşullar altında istekleri serve ederken bir container'ı asla durdurmaz. Ancak bir container, uygulama çıkarsa (örneğin, uygulama kodundaki bir hata nedeniyle) ya da container memory limit'ini aşarsa (varsayılan 512 MiB, 32 GiB'a kadar yapılandırılabilir) aniden durabilir. Bir container istekleri handle ederken durursa, tüm devam eden istekler sonlandırılır ve bir hatayla başarısız olur, ve Cloud Run bir yedek container başlatırken, yeni istekler beklemek zorunda kalabilir.

**15. Doğru cevap: B.**
Bir service revision'ına gelen istekler, internal bir Application Load Balancer tarafından container instance'ları grubuna dağıtılır — tüm instance'lar meşgulse, Cloud Run ek instance'lar ekler, ve talep azaldıkça, bazı instance'lara trafik göndermeyi durdurup onları kapatır. Gelen istek oranına ek olarak, container instance sayısı; istekleri işlerken mevcut instance'ların CPU utilization'ından (hedef yaklaşık %60), maksimum concurrency ayarından, ve minimum/maksimum container instance sayısı ayarından etkilenir.

**16. Doğru cevap: A.**
Bir servise gelen istek yoksa, geriye kalan son container instance'ı bile kapatılır — buna genellikle scale to zero denir. Bu özellik, boşta bekleyen container instance'ları için ödeme yapmadığın için ekonomik nedenlerle çekicidir; servise yeni bir istek gönderildiğinde yeni bir container instance istek üzerine başlar.

**17. Doğru cevap: B.**
Bir servis sıfıra ölçeklendikten sonra gelen ilk birkaç istek, ilk container instance'ı başlarken kuyruğa alınır — buna bir "cold start" denir.

**18. Doğru cevap: D.**
Minimum instances ayarlandığında, Cloud Run, istekleri serve etmiyor olsalar bile (idle), en az bu kadar instance'ı çalışır durumda tutar. Minimum instances özelliği kullanılarak çalışır tutulan idle instance'lar, yine de faturalandırma maliyetine yol açar — modül, minimum instances'ın, aksi halde hiç instance olmayacağı durumlarda servisin gecikmesini azaltmak için özellikle var olduğunu belirtiyor.

**19. Doğru cevap: A.**
Varsayılan olarak, Cloud Run servisleri maksimum 100 instance'a kadar scale out edecek şekilde yapılandırılmıştır, bunu servisin için ayarlayabilir/güncelleyebilirsin. Ayrı olarak, bir Cloud Run servisindeki container instance sayısı, bir platform kotası olarak varsayılan olarak 1.000 instance ile sınırlıdır — daha fazlasına ihtiyaç duyulursa, bir kota artışı talebi gönderilebilir.

**20. Doğru cevap: C.**
Varsayılan olarak, her Cloud Run container instance'ı aynı anda 80 isteğe kadar alabilir, ve bu maksimum 1.000'e kadar artırılabilir. Maximum concurrency'yi 1'e ayarlamak, her istek mevcut container CPU'sunun ya da belleğinin çoğunu kullanıyorsa, uygulama kodu aynı anda birden fazla isteği handle etmek üzere tasarlanmamışsa, ya da uygulama kodu birden fazla istek tarafından paylaşılamayan global state'lere dayanıyorsa düşünülmelidir.

**21. Doğru cevap: A.**
Google Cloud, sanal kaynaklar oluşturup yönetmeni sağlayan bir API koleksiyonu olarak görülebilir, web console, gcloud CLI, Terraform, ya da uygulama kodundan client kütüphaneleri aracılığıyla erişilir. Bir container image'ı Cloud Run'a deploy etmek, bir API çağrısının kendisinin bir örneğidir: gcloud CLI ile `gcloud run deploy` çalıştırmak, `run.googleapis.com`'daki Cloud Run Management API'sine bir API çağrısı yapar.

**22. Doğru cevap: B.**
Bir IAM policy binding, bir member'ı (bir kimlik) tek bir role'e bağlar, ve bir member'ın bir IAM policy içinde birden fazla policy binding'i olabilir, bu da o member'a birden fazla role verir. Bir role, member kimliğinin Google Cloud kaynakları üzerinde belirli aksiyonlar gerçekleştirmesini sağlayan bir izinler kümesi içerir — örneğin, Pub/Sub Publisher role'ü, bir topic'e mesaj publish etme erişimi sağlayan `pubsub.topics.publish` iznini içerir.

**23. Doğru cevap: D.**
Bir Cloud Run servisini public olarak erişilebilir yapmak için, servis üzerinde `allUsers` member türüne IAM Cloud Run Invoker role'ünü atarsın (`--member="allUsers" --role="roles/run.invoker"` ile `gcloud run services add-iam-policy-binding` üzerinden, ya da `gcloud run deploy`'daki `--allow-unauthenticated` flag'i ile). Bireysel bir servise ya da job'a erişimi kontrol etmek için, istenen role ile `gcloud run services/jobs add-iam-policy-binding`/`remove-iam-policy-binding` kullanarak principal ekler/kaldırırsın; projedeki tüm servis ve job'lara aynı anda erişimi kontrol etmek için ise, `gcloud projects add-iam-policy-binding` ile project-level IAM kullanırsın.

**24. Doğru cevap: C.**
Cloud Run, servis düzeyinde üç ingress ayarı sağlar: "All" (en az kısıtlayıcı varsayılan, doğrudan internetten gönderilenler dahil tüm istekleri kabul eder), "Internal" (en kısıtlayıcı, yalnızca internal HTTP(S) load balancer'dan, servisi içeren bir VPC Service Controls perimeter'ının izin verdiği kaynaklardan, aynı proje/perimeter'daki VPC network'lerinden, ve Cloud Tasks, Eventarc, Pub/Sub, ve Workflows gibi aynı proje/perimeter'daki Google Cloud servislerinden gelen istekleri kabul eder), ve "Internal and Cloud Load Balancing" (Internal'ın izin verdiği her şeye ek olarak external HTTP(S) load balancer'ı da izin verir, ama yine de doğrudan internetten değil). Bunlar, servisin kendisine gelen erişimi kontrol eder. Serverless VPC Access ise bunun yerine bir Cloud Run servisini ya da job'ını — bir connector aracılığıyla internal DNS ve internal IP adresleri kullanarak — VM instance'ları ya da Memorystore instance'ları gibi kaynaklara, o trafiği internete maruz bırakmadan erişmek için giden yönde bir VPC network'üne bağlar; ingress ayarları ve Serverless VPC Access bağımsız mekanizmalardır ve birlikte kullanılabilir.
