# Modül 18 — Introduction to Cloud Run and Google Kubernetes Engine: Pratik Sorular

Bu set, Cloud Run temellerini (ne olduğu, developer workflow, Buildpacks ile container-based vs. source-based deployment, HTTPS ve port 8080 gereksinimi, 64-bit Linux binary kısıtlaması, ve iki pricing modeli), Cloud Run use case'lerini (REST API'ler, karmaşık public site'lar, mikroservis iletişimi, event processing, ve Cloud Scheduler ile zamanlanmış görevler), Cloud Run'da yüksek erişilebilirliği (immutable revision'lar ve traffic splitting, autoscaling faktörleri, region'lar/zone'lar, global load balancing, ve Knative tabanlı portability), Google Kubernetes Engine temellerini (GKE'nin Kubernetes'ten farkı, control plane ve node'larla cluster mimarisi, zonal vs. regional cluster'lar, ve pod'lar), temel Kubernetes kaynaklarını (Deployment'lar, Service'ler, ve Volume'ler), GKE geliştirme/CI-CD workflow'unu, ve Container-Optimized OS'u kapsar. Bu modül, "Developing Containerized Applications on Google Cloud" kursunun Modül 2'sidir, Modül 1'i (Modül 17: Introduction to Containers) takip eder.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: bir container'ın ömrünün neden yalnızca istekleri handle ederken garanti edildiği (ve bunun zamanlanmış işler için ne anlama geldiği), Cloud Run uygulamalarının neden hiçbir zaman TLS'i kendilerinin implement etmesi gerekmediği, autoscaling'in neden istek hacminden fazlasına bağlı olduğu, bir Service'in sabit IP'sinin neden özellikle pod IP'lerinin stabil olmaması yüzünden var olduğu, zonal bir cluster'ın regional bir cluster'dan neden farklı olduğu, ve Container-Optimized OS'un daha küçük bir saldırı yüzeyi karşılığında neden bir package manager'dan ve containerized olmayan workload'lardan vazgeçtiği.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Cloud Run nedir, ve belirli bir programlama dilinde yazılmış bir uygulamanın onun üzerinde deploy edilip edilemeyeceğini ne belirler?

A. Cloud Run, container'ları doğrudan Google'ın ölçeklenebilir altyapısı üzerinde deploy edip çalıştıran tam yönetilen bir compute platformudur; uygulama kodunun — herhangi bir dilde yazılmış olursa olsun — bir container image'ını build edebiliyorsan, onu Cloud Run'da deploy edebilirsin.
B. Cloud Run, yalnızca Go, Node.js, Python, Java, .NET Core, ya da Ruby'de yazılmış uygulamaları kabul eden, ve herhangi bir container image'ı doğrudan reddeden bir kaynak kod barındırma servisidir.
C. Cloud Run, herhangi bir uygulama deploy edilmeden önce bir container runtime'ının elle kurulup yapılandırılması gereken bir sanal makine ürünüdür.
D. Cloud Run, her uygulamanın çalışabilmesi için bir Kubernetes Deployment manifest'i olarak paketlenmesini gerektiren bir Kubernetes dağıtımıdır.

**2.** Bir container'ı Cloud Run'a deploy ettikten sonra ne elde edersin, ve modül Cloud Run'ın "serverless" olduğunu söylerken neyi kastediyor?

A. Bir load balancer'a elle bağlanması gereken statik bir IP adresi elde edersin, ve "serverless", platformun herhangi bir CPU ya da bellek tahsisi olmadan tamamen çalıştığı anlamına gelir.
B. DNS'te elle kaydedilmesi gereken bir on-premises sunucu adresi elde edersin, ve "serverless", container'ın hiçbir zaman gerçekten kod çalıştırmadığı anlamına gelir.
C. Benzersiz bir HTTPS URL elde edersin, ve Cloud Run daha sonra container'ını istekleri handle etmek için istek üzerine başlatır, gelen tüm isteklerin container'ları dinamik olarak ekleyip kaldırarak handle edilmesini sağlar; "serverless", bir geliştirici olarak, uygulamanı güçlendiren altyapıyı inşa edip bakımını yapmak yerine uygulamanı inşa etmeye odaklanabileceğin anlamına gelir.
D. Her istekten sonra elle yeniden başlatılması gereken sabit, tek bir container instance'ı elde edersin, ve "serverless", servis için hiçbir faturalandırma olmadığı anlamına gelir.

**3.** Cloud Run'ın container-based ve source-based workflow'ları arasındaki fark nedir?

A. Container-based workflow yalnızca Go ve Java için, source-based workflow ise yalnızca Python ve Ruby gibi yorumlanan diller için kullanılabilir.
B. Container-based workflow'da container image'ı kendin build edersin, içinde tam olarak neyin paketleneceğine karar verirsin; source-based workflow'da ise bir container image yerine kaynak kodunu deploy edersin, ve Cloud Run senin için Buildpacks kullanarak kaynağını build eder ve uygulamayı bağımlılıklarıyla birlikte bir container image'a paketler.
C. Her iki workflow da bir Dockerfile yazmanı gerektirir — tek fark, build'i submit etmek için hangi komut satırı aracını kullandığındır.
D. Container-based workflow doğrudan Google Kubernetes Engine'e deploy eder, source-based workflow ise yalnızca Compute Engine sanal makinelerine deploy eder.

**4.** Bir Cloud Run uygulamasının web isteklerini handle etmek için hangi portu dinlemesi gerekir, ve uygulamanın HTTPS'i kendisinin implement etmesi gerekir mi?

A. Uygulama kendi HTTPS sunucusunu implement etmeli ve kendi TLS sertifikasını yönetmelidir; Cloud Run, herhangi bir protokol farkındalığı olmadan sadece ham TCP paketlerini forward eder.
B. Uygulama doğrudan 443 portunu dinlemelidir, çünkü Cloud Run şifrelenmiş HTTPS trafiğini herhangi bir proxy olmadan doğrudan container'a geçirir.
C. Varsayılan bir port yoktur — her Cloud Run container'ı deploy anında seçilen rastgele bir port numarasıyla elle yapılandırılmalıdır, ve geliştirici her zaman TLS'i implement etmelidir.
D. Cloud Run, container'ın web isteklerini handle etmek için 8080 portunu (yapılandırılabilir bir varsayılan) düz HTTP üzerinden dinlemesini bekler; geçerli bir TLS sertifikası ve diğer HTTPS configuration'ı Cloud Run tarafından provizyonlanır — Cloud Run, gelen HTTPS isteklerini handle eder, decrypt eder, ve uygulamaya forward eder — bu yüzden uygulamanın kendisinin bir HTTPS sunucusu sağlamasına gerek yoktur.

**5.** Modülün, Cloud Run üzerinde çalışan bir uygulamanın hangi programlama dilinde yazılabileceği konusunda tanımladığı tek gerçek kısıtlama nedir?

A. Uygulama, Cloud Run'ın açıkça isimlendirdiği altı source-based dilden (Go, Node.js, Python, Java, .NET Core, ya da Ruby) biriyle yazılmalıdır, başka diller için istisna yoktur.
B. Uygulama, dilden bağımsız olarak hiçbir kütüphane bağımlılığından kaçınmalıdır, çünkü Cloud Run container'larının bir bağımlılık katmanı içermesine izin verilmez.
C. Uygulama 64-bit bir Linux binary'sine compile edilebiliyor ve bir container image'a paketlenebiliyorsa, herhangi bir programlama dilinde geliştirilip Cloud Run'da çalıştırılabilir.
D. Uygulama, Buildpacks'i doğal olarak destekleyen bir dilde yazılmalıdır, çünkü Cloud Run, farklı bir süreçle build edilmiş herhangi bir container image'ı çalıştırmayı reddeder.

**6.** Cloud Run'ın varsayılan pricing modeli nasıl çalışır, ve modül alternatif pricing modeli hakkında ne söylüyor?

A. Varsayılan modelde, yalnızca bir container istekleri handle ederken, ve başlarken ya da kapanırken kullanılan sistem kaynakları için ödeme yaparsın; Cloud Run ayrıca, hiç istek olmasa bile CPU'nun her zaman tahsis edildiği, tüm container lifecycle'ı için ücretlendiren bir alternatif modeli destekler — bu model, çoğu steady-state workload için daha ekonomik olabilir.
B. Varsayılan modelde, kullanımdan bağımsız olarak sabit bir aylık ücret ödersin, ve alternatif model tamamen gerçekleştirilen deployment sayısına göre ücretlendirir, compute süresine göre değil.
C. Varsayılan modelde, container image build'i başına yalnızca bir kez ödersin, ve alternatif bir pricing modeli yoktur — Cloud Run, container'ı sonrasında fiilen çalıştırmak için hiçbir ücret almaz.
D. Varsayılan modelde, faturalandırma yalnızca servise bağlanan farklı client IP adresi sayısına dayanır, ve alternatif model bunun yerine faturalandırmayı container image boyutuna dayandırır.

**7.** Bir ekip, Cloud Run üzerinde bir e-ticaret sitesi gibi daha karmaşık bir public website inşa etmek istiyor. Modül, tanımladığı backend bağlantılarının yanında, performansı iyileştirmek ve kötü amaçlı trafiği filtrelemek için hangi ek Google Cloud servislerinden bahsediyor?

A. Performans iyileştirmesi için VPC Service Controls ve trafik filtreleme için Binary Authorization, Redis, PostgreSQL, ya da üçüncü parti API bağlantılarından hiç bahsedilmiyor.
B. Performansı iyileştirmek için Cloud Armor ve kötü amaçlı trafiği filtrelemek için Cloud CDN — iki servisin amaçları çoğu insanın varsaydığının tersine çevrilmiş, ve mümkün bir backend veritabanı bağlantısından bahsedilmiyor.
C. Hem performans hem trafik filtreleme için yalnızca Identity-Aware Proxy'den bahsediliyor, ve modül Cloud Run servislerinin herhangi bir harici veritabanına bağlanamayacağını belirtiyor.
D. Performansı iyileştirmek için Cloud CDN ve content-based kurallar kullanarak gelen trafiği filtrelemek için Google Cloud Armor, backend'in bir ilişkisel veritabanına (örn. PostgreSQL), kullanıcı session'ları için bir Redis store'a, ve üçüncü parti API'lere bağlanabilmesiyle birlikte.

**8.** Cloud Run tabanlı bir mikroservis mimarisinde, modül servisler arasında hangi iki iletişim desenini tanımlıyor, ve Pub/Sub bu resme nasıl uyuyor?

A. Servisler yalnızca paylaşılan bir Cloud SQL veritabanı üzerinden iletişim kurabilir, ve Pub/Sub'ın yalnızca logging için kullanıldığı, gerçek servisten servise iletişim için kullanılmadığı belirtiliyor.
B. Cloud Run'daki servisler birbirleriyle doğrudan REST API'ler ya da gRPC kullanarak, ya da asenkron olarak Pub/Sub aracılığıyla iletişim kurabilir — Pub/Sub, mesajları bir servisin endpoint'ine HTTP istekleri olarak forward eden (ve isteğe bağlı olarak kimlik doğrulayan) push subscription'larıyla Cloud Run'a iyi entegre olur.
C. Servisler yalnızca asenkron olarak iletişim kurabilir; iki Cloud Run servisi arasındaki doğrudan request/response iletişimi mimari olarak imkansız olarak tanımlanıyor.
D. Pub/Sub'ın yalnızca bir pull subscription modeli gerektirdiği belirtiliyor — Cloud Run servislerinin, push teslimatı Cloud Run'a desteklenmediği için Pub/Sub'ı sürekli poll etmesi gerekiyor.

**9.** Modülün event processing örneğinde, workflow'u ne tetikler, ve ilk Cloud Run servisi işlemeyi bitirdikten sonra ne olur?

A. Zamanlanmış bir Cloud Scheduler işi, herhangi bir yeni dosya olup olmadığından bağımsız olarak tüm workflow'u her saat tetikler, ve ilk servisin tek çıktısı, daha ileri bir downstream aksiyon olmadan bir log kaydıdır.
B. Bir yönetici, pipeline'ın her adımını Google Cloud console üzerinden elle tetikler, ve hiçbir aşamada Pub/Sub mesajlaşması yer almaz.
C. Bir tıbbi tarama görselinin Cloud Storage'a yüklenmesi, taramayı işleyip dönüştürmek üzere bir Cloud Run servisini tetikleyen bir storage event'i üretir; bu servis daha sonra Pub/Sub'a bir mesaj yayınlar — bu, görseli etiketleyip watermark'lamak için başka bir Cloud Run servisini ve tarama verisindeki anomalileri tespit etmek için ayrı bir VM uygulamasını (GPU'lu) tetikler, her ikisi de çıktısını Cloud Storage'a geri saklar.
D. Workflow, Google Cloud servis entegrasyonu olmayan üçüncü parti bir tedarikçiden gelen bir webhook tarafından tetiklenir, ve modül Cloud Run'ın Cloud Storage event'leriyle hiç entegre olamayacağını belirtiyor.

**10.** Bir ekip, fatura oluşturmak gibi zamanlanmış bir görev çalıştırmak istiyor ve zamanlamayı doğrudan uzun süre çalışan bir Cloud Run container'ının içinde implement etmeyi düşünüyor. Modül bu yaklaşım hakkında ne söylüyor, ve bunun yerine neyi öneriyor?

A. Bu yaklaşım en iyi pratik olarak öneriliyor, çünkü bir Cloud Run container'ının ömrünün, istekleri handle edip etmediğinden bağımsız olarak süresiz garanti edildiği belirtiliyor.
B. Zamanlanmış görevlerin, Cloud Run'da hiçbir biçimde desteklenmediği, Compute Engine'in tek uygulanabilir alternatif olduğu belirtiliyor.
C. Modül, zamanlamayı doğrudan container'da implement etmeyi öneriyor, çünkü container instance'larının bir kez başladıktan sonra asla kapatılmayacağı garanti ediliyor, bu da harici tetiklemeyi gereksiz kılıyor.
D. Bir container'ın ömrü yalnızca istekleri handle ederken garanti edilir, bu yüzden container içinde daha sonra çalışması için zamanlanan bir görev, container'ın o zamana kadar zaten kapatılmış ya da durdurulmuş olması riskini taşır; modül bunun yerine, Cloud Run servisini bir zamanlamaya göre güvenle tetiklemek için tam yönetilen bir cron job scheduler'ı olan Cloud Scheduler'ı kullanmayı öneriyor — servis, görevini yapılandırılmış istek timeout'u içinde tamamlamalıdır.

**11.** Bir Cloud Run servisine her deploy edişinde ona ne olur, ve kötü bir deployment'ın etkisini nasıl azaltabilirsin?

A. Her deployment, container image ve service configuration'dan (environment variable'lar, memory limit'leri, vb.) oluşan, yeni, immutable bir revision oluşturur; yeni ve önceki revision'lar arasında yüzdeyle istek trafiğini bölerek, stabil bir revision'a geri dönmene (rollback) ya da yeni olana kademeli olarak geçiş yapmana izin verebilirsin.
B. Her deployment, önceki revision'ın üzerine sessizce yazar, bu yüzden yeni bir deployment tamamlandıktan sonra önceki bir sürüme geri dönmenin bir yolu yoktur.
C. Her deployment, tüm Cloud Run servisinin elle silinip sıfırdan yeniden oluşturulmasını gerektirir, "revision" kavramı hiç mevcut değildir.
D. Her deployment, trafiğin %100'ünü otomatik olarak hemen yeni revision'a yönlendirir, revision'lar arasında trafik bölmeye yönelik hiçbir mekanizma tanımlanmamıştır.

**12.** Gelen isteklerin ham oranının ötesinde, modül Cloud Run'ın autoscaling'inin bir servis için tuttuğu container instance sayısını hangi faktörlerin etkilediğini söylüyor?

A. Yalnızca uygulamanın kayıtlı kullanıcı sayısı, container instance sayısıyla ilgili başka hiçbir faktör tanımlanmıyor.
B. İstekleri işlerken mevcut instance'ların CPU utilization'ı (yaklaşık %60 utilization hedefi), maksimum concurrency ayarı (belirli bir instance'a paralel olarak kaç istek gönderilebileceği), ve yapılandırılmış minimum ve maksimum container instance sayısı.
C. Yalnızca client ile en yakın Google Cloud region'ı arasındaki coğrafi mesafe, CPU utilization ve concurrency'nin hiçbir etkisi olmadığı belirtiliyor.
D. Artifact Registry'de o anda saklanan container image sayısı, çünkü modül instance sayısını doğrudan registry storage kullanımına bağlıyor.

**13.** Bir region nedir, bir zone nedir, ve Cloud Run bunları yüksek erişilebilirlik için nasıl kullanır?

A. Bir region ve bir zone, modülde aralarında hiçbir ayrım olmayan, aynı, birbirinin yerine kullanılabilir terimler olarak tanımlanıyor.
B. Bir zone'un birden fazla region'ı kapsadığı, ve bir region'ın tek bir zone'un bir alt kümesi olduğu belirtiliyor — modülün gerçek hiyerarşisinin tersi.
C. Bir region, farklı kıtalarda bulunan tam olarak iki zone'dan oluşur, ve Cloud Run'ın her zaman bunlardan yalnızca birine deploy ettiği belirtiliyor.
D. Bir region, Google Cloud kaynaklarının barındırıldığı belirli bir coğrafi konumdur, üç ya da daha fazla zone'dan oluşur (her biri region içinde tek bir hata alanı olarak kabul edilen bir deployment alanı); yüksek erişilebilirlik için, Cloud Run container'ları bir region içindeki birden fazla zone'a dağıtır, bu da uygulamayı tek bir zone'un başarısız olmasına karşı dirençli kılar.

**14.** Global external Application Load Balancer, birden fazla regional Cloud Run servisiyle ne yapmanı sağlar, ve dünya çapındaki client'lara ne fayda sağlar?

A. Birden fazla regional Cloud Run servisini otomatik olarak tek bir region'a birleştirir, birden fazla region ihtiyacını tamamen ortadan kaldırır.
B. Dünya çapındaki her client'ın, fiziksel konumlarından bağımsız olarak tek, sabit bir region üzerinden yönlendirilmesini gerektirir — modül, bunun yük testi amacıyla kasıtlı olarak gecikmeyi artırdığını belirtiyor.
C. Birden fazla regional Cloud Run servisinin önünde tek, global bir IP adresi açığa çıkarır ve her client'ın isteklerini onlara en yakın region'a yönlendirir, uygulama erişilebilirliğini iyileştirir ve dünya çapındaki client'lar için gecikmeyi azaltır.
D. Cloud Run revision'larına olan ihtiyacı tamamen ortadan kaldırır, çünkü modül load balancer'ın revision'lar arasındaki tüm traffic splitting'i de handle ettiğini belirtiyor.

**15.** Modül, Cloud Run uygulamalarını hangi iki şekilde taşınabilir olarak tanımlıyor, ve portability bir geliştirici için neden önemlidir?

A. Portability yalnızca vendor lock-in'den kaçınmak için önemlidir ve modülün kendi örneklerinde hiçbir zaman data sovereignty gereksinimleriyle bağlantılı değildir.
B. Cloud Run uygulamaları, container'ların doğası gereği her yerde çalışabilmesi, ve Cloud Run platformunun (aynı container runtime contract'ını implement eden açık kaynaklı bir proje olan) Knative ile API-uyumlu olması nedeniyle taşınabilirdir — bu da aynı uygulamanın Kubernetes tabanlı ortamlarda çalışmasını sağlar; bu, bir uygulamanın Google Cloud'un fiziksel varlığının olmadığı bir region'da çalışması gerektiğinde (veri egemenliği için) ya da bir geliştirici vendor lock-in'den kaçınmak istediğinde faydalıdır.
C. Cloud Run uygulamaları, yalnızca herhangi bir hypervisor ile uyumlu sanal makine image'ları olarak export edilebildiği için taşınabilirdir, modülde Knative'den hiçbir yerde bahsedilmiyor.
D. Portability, diğer hiçbir container tabanlı platforma — Kubernetes tabanlı olanlar dahil — uygulanmayan, yalnızca Cloud Run'a özgü bir sınırlama olarak tanımlanıyor.

**16.** Bir ekip, çok hızlı bir şekilde çok sayıda instance'a autoscale edebilen bir Cloud Run servisi deploy ediyor. Modül, VM tabanlı workload'ları taşımanın genel konusundan ayrı olarak, bu konuda hangi iki hususu gündeme getiriyor?

A. Servis birçok instance'a ölçeklenirse, o instance'ları çalıştırmak için maliyete katlanırsın (bu, maksimum instance sayısı ayarlanarak sınırlandırılabilir), ve hızlı bir scale-up, downstream sistemlere onların throughput kapasitesinin kaldırabileceğinden daha fazla trafik gönderebilir — bu da servisi yapılandırırken o sistemlerin kapasitesini anlamanı gerektirir.
B. Tanımlanan tek husus, autoscaling'in instance sayısından bağımsız olarak tamamen ücretsiz olduğu, ve downstream sistemlerin herhangi bir yapılandırma olmadan her zaman sınırsız ek trafiği absorbe edebileceğidir.
C. Modül, herhangi bir türde downstream bağımlılığı olan herhangi bir servis için autoscaling'in her zaman devre dışı bırakılması gerektiğini belirtiyor, alternatif olarak mevcut bir maksimum instance ayarı yok.
D. Tanımlanan tek husus, Cloud Run servislerinin hiçbir koşulda tek bir instance'ın ötesine ölçeklenemeyeceği, bu da downstream throughput'u alakasız kıldığıdır.

**17.** Kubernetes nedir, Google Kubernetes Engine (GKE) nedir, ve GKE, kendi başına inşa etmen gerekecek olan hangi gelişmiş cluster yönetim özelliklerini sağlar?

A. Kubernetes, hiçbir açık kaynak bileşeni olmayan tescilli bir Google ürünüdür, ve GKE'nin, herhangi bir ek yönetim özelliği olmadan başka herhangi bir cloud sağlayıcısında Kubernetes çalıştırmakla işlevsel olarak aynı olduğu belirtiliyor.
B. Kubernetes, yazılım deployment'ını, ölçeklendirmesini, ve yönetimini otomatikleştirmek için açık kaynaklı bir container orkestrasyon sistemidir, orijinal olarak Google tarafından tasarlandıktan sonra artık Cloud Native Computing Foundation (CNCF) tarafından bakımı yapılmaktadır; GKE, kolay cluster oluşturma ve yönetimi, load balancing, automatic scaling, otomatik node yazılımı upgrade'leri, otomatik node repair, ve Google Cloud'un operations suite'i aracılığıyla entegre logging/monitoring gibi özellikler ekleyen tam yönetilen bir Kubernetes servisidir.
C. Kubernetes ve GKE, benzer isimleri paylaşan tamamen ilgisiz ürünler olarak tanımlanıyor, GKE'nin altında aslında Kubernetes çalıştırmadığı belirtiliyor.
D. GKE, kullanıcılara daha fazla elle kontrol vermek için cluster yönetim özelliklerini eklemek yerine çıkaran, ham Kubernetes'ten daha düşük seviyeli bir ürün olarak tanımlanıyor.

**18.** Bir GKE cluster'ının control plane'inin rolü nedir, ve zonal bir cluster ile regional bir cluster arasındaki fark nedir?

A. Control plane, cluster'ın node'larında çalışan her şeyi yönetir — container workload'larını zamanlar ve yaşam döngülerini, ölçeklendirmesini, upgrade'lerini, ve network/storage kaynaklarını Kubernetes API server'ı (`kube-apiserver`) aracılığıyla yönetir; zonal bir cluster'ın tek bir zone'da çalışan tek bir control plane'i vardır, regional bir cluster'ın ise bir region içindeki birden fazla zone'da çalışan birden fazla control plane replica'sı vardır, daha yüksek erişilebilirlik için.
B. Zonal bir cluster ile regional bir cluster'ın işlevsel olarak aynı olduğu belirtiliyor, terimler birbirinin yerine kullanılıyor ve aralarında gerçek bir mimari fark yok.
C. Control plane'in tek işlevinin uygulama container'larını doğrudan çalıştırmak olduğu, node'ların ise yalnızca container image'ları için pasif depolama olarak var olduğu belirtiliyor.
D. Control plane'in yalnızca cluster için faturalandırma bilgisini sakladığı ve workload'ları zamanlamada ya da yönetmede hiçbir rolü olmadığı, bunun yerine bağımsız hareket eden ayrı ayrı node'lar tarafından tamamen ele alındığı belirtiliyor.

**19.** Bir GKE node'u nedir, o node üzerinde `kubelet` ne yapar, ve bir pod nedir?

A. Bir node'un Artifact Registry'de saklanan bir container image olduğu, `kubelet`'in yalnızca insan operatörler tarafından kullanılan bir komut satırı aracı olduğu (hiçbir zaman bir agent olarak çalışmadığı), ve bir pod'un node başarısızlığından sonra da hayatta kalan, kalıcı, ephemeral olmayan bir Kubernetes nesnesi olarak tanımlandığı belirtiliyor.
B. Bir node'un sanal makine çalıştıramayan fiziksel bir sunucu olduğu, `kubelet`'in container çalıştırmayla ilgisiz bir faturalandırma agent'ı olarak tanımlandığı, ve bir pod'un tüm bir cluster'ın eş anlamlısı olarak tanımlandığı belirtiliyor.
C. Bir node, containerized workload'ları çalıştıran bir Compute Engine sanal makinesidir; `kubelet`, control plane ile iletişim kuran ve node üzerinde zamanlanan container'ları başlatmaktan ve çalıştırmaktan sorumlu olan Kubernetes node agent'ıdır; bir pod, Kubernetes'teki en küçük deployable compute birimidir — paylaşılan storage ve network kaynaklarına sahip, bir ya da daha fazla container'dan oluşan bir grup ve bunların nasıl çalıştırılacağına dair bir spesifikasyon.
D. Bir node'un, altında yatan herhangi bir compute kaynağıyla hiçbir ilişkisi olmayan bir Kubernetes API nesnesi olduğu, `kubelet`'in node'lar yerine yalnızca control plane üzerinde çalıştığı, ve bir pod'un her zaman tam olarak bir container içerdiği, daha fazlasının mümkün olmadığı belirtiliyor.

**20.** Bir Kubernetes Deployment ne tanımlar, ve bir Deployment ile bir ReplicaSet arasındaki ilişki nedir?

A. Bir Deployment'ın YAML manifest'inin hiçbir zaman bir selector label ya da pod template'i içermediği, yalnızca container image ismini içerdiği belirtiliyor.
B. Bir Deployment'ın, cluster'da hiçbir kalıcı nesne oluşturmayan, tamamen imperatif, tek seferlik bir komut olduğu, ve ReplicaSet'in Deployment'larla hiçbir bağlantısı olmayan, tamamen ilgisiz, bağımsız bir Kubernetes kaynağı olarak tanımlandığı belirtiliyor.
C. Bir Deployment'ın yalnızca aynı anda tam olarak bir pod'u yönetebildiği, ve ReplicaSet kavramının artık kullanılmadığı, Deployment'lar tarafından deprecated olarak tanımlandığı belirtiliyor.
D. Bir Deployment, Kubernetes'te pod'ları ve ReplicaSet'leri deklaratif olarak oluşturup yönetmenin bir yoludur, istenen bir durumu tanımlar (bir ReplicaSet aracılığıyla istenen pod replica sayısı dahil — bu ReplicaSet'in amacı, herhangi bir anda çalışan stabil bir replica pod kümesini korumaktır); Deployment Controller, gerçek durumu kontrollü bir hızda istenen duruma değiştirir.

**21.** Bir Kubernetes Service'in, client'ların doğrudan pod IP adreslerini çağırması yerine neden kendi sabit IP adresine ihtiyacı vardır, ve bir Service ne sağlar?

A. Bir Service, bir selector aracılığıyla seçilen, üye pod'ları arasında load balancing ile birlikte, servisin ömrü boyunca kalan sabit bir IP adresi sağlayan bir network soyutlamasıdır; bu, pod'ların ephemeral olması ve silinip yeniden oluşturuldukça IP adreslerinin değişmesi nedeniyle vardır — bu yüzden pod IP adreslerini doğrudan kullanmak mantıklı değildir; client'lar bunun yerine servis IP'sini çağırır, ve istekler üye pod'lar arasında load-balanced edilir.
B. Bir Service'in yalnızca kozmetik isimlendirme amacıyla var olduğu; pod IP adreslerinin, pod'un tüm ömrü boyunca kalıcı olarak sabit olduğu, bu da bir Service'in IP adresini teknik olarak gereksiz kıldığı belirtiliyor.
C. Bir Service'in pod'lar için storage kalıcılığı sağladığı, IP adresleme ve load balancing'in ise Service'in değil, tek tek pod'ların sorumluluğunda olduğu belirtiliyor.
D. Bir Service'in yalnızca tek, belirli bir isimli pod'a trafik yönlendirebildiği, ve bir label ile seçilen bir pod grubunu temsil edip load-balance edemediği belirtiliyor.

**22.** Ephemeral bir Kubernetes volume türü (ConfigMap ya da Secret gibi) ile durable bir tür (PersistentVolume gibi) arasındaki fark nedir?

A. Ephemeral ve durable volume türlerinin işlevsel olarak aynı olduğu, tek farkın bunları tanımlamak için hangi YAML anahtar kelimesinin kullanıldığı olduğu belirtiliyor.
B. Ephemeral volume türleri, kapsayan pod'larıyla aynı ömre sahiptir — pod oluşturulduğunda oluşturulur ve pod sonlandırıldığında kaldırılır — buna karşın PersistentVolume gibi durable storage'la desteklenen bir volume türü, pod'dan bağımsız bir yaşam döngüsüne sahiptir, ve verisi pod sonlandırıldığında korunur.
C. Durable volume türlerinin, onları oluşturan pod sonlandığı anda otomatik olarak silindiği, ephemeral volume türlerinin ise pod durumundan bağımsız olarak sonsuza kadar kalıcı olduğu belirtiliyor.
D. Her iki volume türünün de, herhangi bir container'ın onlara erişebilmesi için pod'un yeniden oluşturulmasını gerektirdiği, kalıcılık davranışında bir ayrım olmadığı belirtiliyor.

**23.** Modülün tanımladığı GKE geliştirme ve deployment workflow'unda, kaynak repository'ye bir değişiklik commit edildikten sonra Cloud Build otomatik olarak ne yapar, ve bir container production'a terfi ettirilmeden önce ne olur?

A. Cloud Build'in yalnızca kodu geliştiricinin kendi makinesinde local olarak compile ettiği ve workflow'un hiçbir noktasında Artifact Registry'e ya da Cloud Storage'a hiç dokunmadığı belirtiliyor.
B. Bir commit'ten sonra hiçbir şey otomatik olarak gerçekleşmez — image saklama dahil, workflow'da tanımlanan build, test, ve deploy sürecinin her tek adımını bir insanın elle tetiklemesi gerekir.
C. Cloud Build'in yalnızca testleri çalıştırmaktan sorumlu olduğu, container image build'inin ve saklanmasının workflow'da bahsedilmeyen, tamamen ayrı, ilgisiz bir araç tarafından ele alındığı belirtiliyor.
D. Cloud Build, uygulamanın container image'ını yeniden build eder, image'ı Artifact Registry'de saklar, build artifact'lerini bir Cloud Storage bucket'ında saklar, container üzerinde testler çalıştırır, ve container'ı bir staging GKE ortamına deploy etmek için Google Cloud Deploy'u çağırır; build ve testler başarılı olursa, onay sonrasında container'ı production'a terfi ettirmek için Cloud Deploy kullanılır.

**24.** Container-Optimized OS nedir, ve modülde tanımlanan iki dikkat çekici sınırlaması nedir?

A. Container-Optimized OS'un, Compute Engine ile ilgisiz, genel amaçlı bir masaüstü işletim sistemi olduğu, ve tanımlanan ana sınırlamasının Docker container'larını hiç çalıştıramaması olduğu belirtiliyor.
B. Container-Optimized OS'un, genel amaçlı bir Linux dağıtımıyla karşılaştırıldığında hiçbir sınırlaması olmadığı, bu yüzden istisnasız her olası workload için uygun olduğu belirtiliyor.
C. Container-Optimized OS, Google tarafından bakımı yapılan, açık kaynaklı Chromium OS projesine dayanan ve Docker container'ları çalıştırmak için optimize edilmiş, Compute Engine VM'leri için bir işletim sistemi image'ıdır; dikkat çekici iki sınırlaması, bir package manager içermemesi (bu yüzden bir instance'a doğrudan yazılım kurulamaz) ve containerized olmayan uygulamaların çalıştırılmasını desteklememesidir.
D. Container-Optimized OS'un herhangi bir cloud sağlayıcısında ya da on-premises ortamda tam olarak desteklendiği, tek sınırlamasının otomatik güvenlik güncellemelerinin eksikliği olduğu belirtiliyor.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Cloud Run, container'ları doğrudan Google'ın ölçeklenebilir altyapısının üzerinde deploy edip çalıştırmanı sağlayan tam yönetilen bir compute platformudur. Uygulama kodunun — herhangi bir dilde yazılmış olursa olsun — bir container image'ını build edebiliyorsan, o uygulamayı Cloud Run'da deploy edebilirsin.

**2. Doğru cevap: C.**
Container'ını deploy ettikten sonra, benzersiz bir HTTPS URL elde edersin. Cloud Run daha sonra container'ını istekleri handle etmek için istek üzerine başlatır, ve gelen tüm isteklerin container'ları dinamik olarak ekleyip kaldırarak handle edilmesini sağlar. Cloud Run serverless'tır, bu da bir geliştirici olarak, uygulamanı güçlendiren altyapıyı inşa edip bakımını yapmak yerine uygulamanı inşa etmeye odaklanabileceğin anlamına gelir.

**3. Doğru cevap: B.**
Container image'ı kendin build edersen (container-based workflow), içinde tam olarak hangi dosyaların paketleneceğine sen karar verirsin. Bunun yerine source-based yaklaşımı kullanırsan, bir container image yerine kaynak kodunu deploy edersin; Cloud Run, Buildpacks kullanarak kaynağını build eder ve uygulamayı bağımlılıklarıyla birlikte senin için bir container image'a paketler.

**4. Doğru cevap: D.**
Cloud Run, container'ının web isteklerini handle etmek için 8080 numaralı portu dinlemesini bekler — bu port uygulamana uygun değilse değiştirebileceğin, yapılandırılabilir bir varsayılandır. Cloud Run, HTTPS isteklerini desteklemek için geçerli bir TLS sertifikası ve diğer configuration'ı provizyonlar, ve gelen istekleri handle eder, decrypt eder, ve uygulamana forward eder — kendi HTTPS sunucunu sağlamana gerek yoktur.

**5. Doğru cevap: C.**
Cloud Run'ın büyük bir avantajı, container'lar çalıştırmasıdır — bu, uygulamalarını herhangi bir programlama dilinde geliştirip Cloud Run'da çalıştırabileceğin anlamına gelir, tek şart bunların 64-bit bir Linux binary'sine compile edilebilir ve bir container image'a paketlenebilir olmasıdır.

**6. Doğru cevap: A.**
Cloud Run pricing modeli, yalnızca bir container istekleri handle ederken, ve başlarken ya da kapanırken kullanılan sistem kaynakları için ödeme yapman açısından benzersizdir. Cloud Run ayrıca, uygulamana hiç istek gelmese bile container instance'larına her zaman CPU tahsis edilen, tüm container lifecycle'ı için ücretlendiren bir pricing modelini de destekler — bu model, çoğu steady-state workload için daha ekonomik olabilir.

**7. Doğru cevap: D.**
Cloud Run üzerinde bir e-ticaret sitesi gibi daha karmaşık bir public website için, performansı iyileştirmek üzere Cloud CDN'i etkinleştirebilir, ve content-based politikalarla kötü amaçlı gelen trafiği filtrelemek üzere Google Cloud Armor ekleyebilirsin. Backend'de, ilişkisel bir veritabanına, kullanıcı session'ları için bir Redis store'a, ve üçüncü parti API'lere bağlanabilirsin.

**8. Doğru cevap: B.**
Cloud Run'daki servisler, doğrudan request/response için REST API'ler ya da gRPC kullanarak, ya da Pub/Sub kullanarak garantili teslimatlı asenkron mesajlar göndererek/alarak birbiriyle iletişim kurabilir. Pub/Sub, mesajları bir Cloud Run servisinin endpoint'ine HTTP istekleri olarak forward eden ve isteğe bağlı olarak kimlik doğrulayan push subscription'ları kullanılarak Cloud Run'a iyi entegre olur.

**9. Doğru cevap: C.**
Modülün örneğinde, bir tıbbi tarama görseli Cloud Storage'a yüklendiğinde, taranan görseli işleyip modern bir formata dönüştürmek üzere bir Cloud Run servisi tetiklenir. Bu servis daha sonra Pub/Sub'a bir mesaj gönderir — bu, dönüştürülen görseli etiketleyip watermark'layan başka bir Cloud Run servisini, ve tarama verisindeki anomalileri tespit eden ayrı bir VM uygulamasını (GPU'lu) tetikler — her ikisi de çıktısını Cloud Storage'a geri yazar.

**10. Doğru cevap: D.**
Zamanlanmış bir işi container'ın kendisinde çalıştırmanın sınırlaması, bir container'ın ömrünün yalnızca istekleri handle ederken garanti edilmesidir — görevleri bir container üzerinde daha sonra çalışacak şekilde zamanlarsan, o görevin çalışması gereken zamana kadar container kapatılmış ya da durdurulmuş olabilir. Modül, bir Cloud Run servisini bir zamanlamaya göre güvenle tetiklemek için tam yönetilen bir cron job scheduler'ı olan Cloud Scheduler'ı kullanmayı öneriyor — servisin görevini yapılandırılmış istek timeout'u içinde tamamlaması gerektiğini belirtiyor.

**11. Doğru cevap: A.**
Cloud Run'da, container image'ını bir servise her deploy edişinde, container image ve service configuration'dan (environment variable'lar ve memory limit'leri gibi ayarlar) oluşan, yeni, immutable bir revision oluşturulur. Yeni revision'a gönderilecek isteklerin yüzdesini belirterek trafiği yeni ve önceki revision'lar arasında bölerek, istek işleme hatalarının etkisini azaltabilirsin — bu, önceki stabil bir revision'a geri dönmene ya da trafiği kademeli olarak %100 yeni revision'a göndermene olanak tanır.

**12. Doğru cevap: B.**
Gelen istek oranına ek olarak, container instance sayısı; istekleri işlerken mevcut instance'ların CPU utilization'ından (hedef %60 utilization), maksimum concurrency ayarından, ve minimum/maksimum container instance sayısı ayarından etkilenir.

**13. Doğru cevap: D.**
Bir region, Google Cloud kaynaklarının barındırıldığı belirli bir coğrafi konumdur, ve bir region üç ya da daha fazla zone'a sahiptir — her biri, region içinde tek bir hata alanı olarak kabul edilen bir deployment alanıdır. Yüksek erişilebilirlik için, Cloud Run container'ları bir region içindeki birden fazla zone'a dağıtır, bu da uygulamayı bir zone'un başarısız olmasına karşı dirençli kılar.

**14. Doğru cevap: C.**
Cloud Run, birden fazla regional Cloud Run servisinin önünde tek, global bir IP adresi açığa çıkarmanı sağlayan Google Cloud'un global external Application Load Balancer'ıyla entegre olur. Load balancer, bir client'tan gelen istekleri onlara en yakın region'a yönlendirir, uygulama erişilebilirliğini iyileştirir ve dünya çapındaki client'lar için gecikmeyi azaltır.

**15. Doğru cevap: B.**
Cloud Run'daki uygulamalar iki şekilde taşınabilirdir: Cloud Run, her yerde çalışabilen container'lar kullanır, bu da uygulamaları doğası gereği taşınabilir kılar; ve Cloud Run platformu, aynı container runtime contract'ını implement eden açık kaynaklı bir proje olan Knative ile API-uyumludur, bu da serverless uygulamaların Kubernetes tabanlı ortamlarda çalışmasını sağlar. Portability, Google Cloud'un fiziksel varlığının olmadığı bir region'da çalışmak (veri egemenliği için) ya da vendor lock-in'den kaçınmak gibi kullanım alanları için önemlidir.

**16. Doğru cevap: A.**
Birçok container instance'ına kadar ölçeklenen bir servis deploy edersen, o container'ları çalıştırmak için maliyete katlanırsın — bunu, maksimum container instance sayısını ayarlayarak sınırlayabilirsin. Ayrı olarak, servisin kısa bir süre içinde birçok instance'a ölçeklenirse, downstream sistemler ek trafik yükünü handle edemeyebilir, bu yüzden servisi yapılandırırken onların throughput kapasitesini anlaman gerekir.

**17. Doğru cevap: B.**
Kubernetes, yazılım deployment'ını, ölçeklendirmesini, ve yönetimini otomatikleştirmek için açık kaynaklı bir container orkestrasyon sistemidir; orijinal olarak Google tarafından tasarlanmış, artık Cloud Native Computing Foundation (CNCF) tarafından bakımı yapılıyor. GKE, kolay cluster oluşturma ve yönetimi, load balancing, automatic scaling, otomatik node yazılımı upgrade'leri, otomatik node repair, ve Google Cloud'un operations suite'i aracılığıyla logging/monitoring dahil gelişmiş cluster yönetim özellikleri sağlayan tam yönetilen bir Kubernetes servisidir.

**18. Doğru cevap: A.**
Control plane, cluster'ın tüm node'larında çalışan her şeyi yönetir — container workload'larını zamanlar ve yaşam döngülerini, ölçeklendirmesini, ve upgrade'lerini, ayrıca network ve storage kaynaklarını, Kubernetes API server'ı (`kube-apiserver`) aracılığıyla iletişim kurarak yönetir. Zonal bir cluster'ın tek bir zone'da çalışan tek bir control plane'i vardır, regional bir cluster'ın ise, daha yüksek erişilebilirlik için, belirli bir region içindeki birden fazla zone'da çalışan birden fazla control plane replica'sı vardır.

**19. Doğru cevap: C.**
Node'lar, containerized uygulamaları ve diğer workload'ları çalıştıran Compute Engine sanal makineleridir. Bir node, container'larını desteklemek için gereken servisleri çalıştırır — bunlara runtime ve Kubernetes node agent'ı (`kubelet`) dahildir; `kubelet`, control plane ile iletişim kurar ve node üzerinde zamanlanan container'ları başlatmaktan ve çalıştırmaktan sorumludur. Bir pod, Kubernetes'teki en küçük deployable compute birimidir — paylaşılan storage ve network kaynaklarına sahip, bir ya da daha fazla container'dan oluşan bir grup ve bunların nasıl çalıştırılacağına dair bir spesifikasyon.

**20. Doğru cevap: D.**
Bir Deployment, Kubernetes'te pod'ları ve ReplicaSet'leri deklaratif olarak oluşturup yönetmenin bir yoludur, bir cluster'daki pod'lar için istenen bir durumu tanımlar. Herhangi bir anda çalışan stabil bir replica pod kümesini korumak amacıyla, istenen pod replica sayısını belirten bir ReplicaSet tanımlar. Deployment Controller, deployment'ın gerçek durumunu kontrollü bir hızda istenen duruma değiştirir.

**21. Doğru cevap: A.**
Bir Kubernetes Service, bir selector aracılığıyla seçilen bir pod grubu için stabil bir endpoint sağlayan bir network soyutlamasıdır — servisin ömrü boyunca kalan sabit bir IP adresi ve üye pod'ları arasında load balancing sağlar. Pod'lar ephemeral olduğundan, silinip yeniden oluşturuldukça IP adresleri değişir, bu yüzden pod IP adreslerini doğrudan kullanmak mantıklı değildir — client'lar bunun yerine servis IP adresini çağırır, ve istekleri, servisin üyesi olan pod'lar arasında load-balanced edilir.

**22. Doğru cevap: B.**
Ephemeral volume türleri, kapsayan pod'larıyla aynı ömre sahiptir — pod oluşturulduğunda oluşturulur ve pod sonlandırıldığında kaldırılır. PersistentVolume gibi durable storage'la desteklenen volume türleri, pod'dan bağımsız var olur, ve verisi pod sonlandırıldığında korunur.

**23. Doğru cevap: D.**
Kaynak repository'ye bir değişiklik commit edildiğinde, Cloud Build uygulamanın container image'ını yeniden build eder, image'ı Artifact Registry'de saklar, herhangi bir build artifact'ini bir Cloud Storage bucket'ında saklar, container üzerinde testler çalıştırır, ve container'ı bir GKE cluster'ı içeren bir staging ortamına deploy etmek için Google Cloud Deploy'u çağırır. Build ve testler başarılıysa, onay sonrasında container'ı bir production ortamına terfi ettirmek için Cloud Deploy kullanılır.

**24. Doğru cevap: C.**
Container-Optimized OS, Google tarafından bakımı yapılan ve açık kaynaklı Chromium OS projesine dayanan, Docker container'ları çalıştırmak için optimize edilmiş, Compute Engine VM'leri için bir işletim sistemi image'ıdır. Sınırlamaları arasında: bir package manager dahil değildir, bu yüzden bir instance'a doğrudan yazılım paketi kuramazsın, ve containerized olmayan uygulamaların çalıştırılmasını desteklemez.
