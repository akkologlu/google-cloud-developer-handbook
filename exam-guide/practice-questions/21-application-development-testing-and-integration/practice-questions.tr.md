# Modül 21 — Application Development, Testing, and Integration: Pratik Sorular

Bu set, development ve testing'i (bir uygulamanın Cloud Run'a uyması için karşılaması gereken kriterler, service'ler için container runtime contract'ı ile job'lar için olanın farkı, execution environment'lar, dosya sistemi ve data storage seçenekleri, Cloud Code, ve local test araçları), service deployment'larını ve revision'larını yönetmeyi (container'ları build edip deploy etmek, Artifact Registry'nin rolü ve internal image kopyalama, servisleri oluşturmak/güncellemek, sekiz service configuration bileşeni, revision immutability, trafiğin yeni bir revision'a nasıl migrate olduğu, ve trafiği pinlemek/tag'lemek/bölmek), ve Google Cloud servisleriyle entegrasyonu (client kütüphaneleri ve per-service identity, Memorystore'a bağlanmak ve Cloud Run Integrations, Pub/Sub'dan tetiklenmek, ve Cloud SQL'e bağlanmak) kapsar.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: bir job'ın container'ının neden hiçbir zaman bir portu dinlememesi gerektiği (bir service'inkinin ise dinlemesi gerektiği), sadece bir configuration ayarını değiştirmenin neden yeni bir image olmadan bile yeni bir revision oluşturduğu, Cloud Run'ın neden bir container image'ı her seferinde doğrudan Artifact Registry'den pull etmek yerine kendi storage'ına kopyaladığı, pinning, tagging, ve splitting'in neden tek bir araç değil üç farklı araç olduğu, built-in service account'un neden çoğu insanın varsaydığından daha geniş bir risk olduğu, ve Pub/Sub'dan tetiklenen bir Cloud Run endpoint'inin neden public olması gerekmediği.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Bir uygulamanın Cloud Run için iyi bir seçim sayılması için tam olarak hangi kriterleri karşılaması gerekir?

A. HTTP, HTTP/2, WebSockets, ya da gRPC üzerinden teslim edilen istekleri, stream'leri, ya da event'leri serve etmelidir (ya da tamamlanana kadar çalışmalıdır); local persistent bir dosya sistemi gerektirmemelidir; birden fazla eşzamanlı instance'ı handle edecek şekilde build edilmelidir; instance başına 8 CPU ve 32 GiB bellekten fazlasını kullanmamalıdır; ve containerized olmalı, Go/Java/Node.js/Python/.NET'te yazılmış olmalı, ya da başka şekilde containerize edilebilir olmalıdır.
B. Bellek kullanımı, dosya sistemi gereksinimleri, ya da istek handling gibi başka herhangi bir özellikten bağımsız olarak, sadece Python'da yazılmış olması gerekir.
C. Local persistent bir dosya sistemi gerektirmeli ve instance başına 8 CPU'dan fazlasını kullanmalıdır, çünkü Cloud Run'ın özellikle yüksek kaynaklı, stateful workload'lar için tasarlandığı belirtiliyor.
D. Yalnızca tek bir instance olarak çalışacak şekilde tasarlanmalıdır, çünkü modül Cloud Run'ı, birden fazla eşzamanlı instance'ı handle edecek şekilde build edilmiş uygulamalarla uyumsuz olarak tanımlıyor.

**2.** Container'ın yapması gereken şey açısından, container runtime contract'ı bir Cloud Run service'i ile bir Cloud Run job'ı için nasıl farklıdır?

A. Hem service'ler hem job'lar bir portu dinlemeli ve bir HTTP yanıtı döndürmelidir, runtime contract'ta iki container türü arasında hiçbir yerde bir ayrım yapılmamaktadır.
B. Bir job'ın container'ı bir portu dinlemeli ve bir HTTP yanıtı döndürmelidir, bir service'in container'ı ise isteğin başarılı olup olmadığından bağımsız olarak 0 kodla çıkmalıdır.
C. Bir service'in container'ı doğru portta istekleri dinlemeli ve yapılandırılmış timeout içinde (startup dahil 1 saate kadar) yanıt vermelidir, aksi halde istek 504 hatasıyla sonlanır; bir job'ın container'ı bir portu dinlememeli ya da bir web sunucusu başlatmamalıdır, bunun yerine başarıda 0 kodla, başarısızlıkta sıfır olmayan bir kodla çıkmalıdır.
D. Ne service'ler ne de job'lar runtime contract altında spesifik bir şey yapmak zorundadır — contract yalnızca hangi base image'ın kullanılabileceğini yönetir.

**3.** Modül, transport layer security'yi (TLS) doğrudan bir Cloud Run container'ının içinde implement etmek hakkında ne söylüyor?

A. Modül, şifreleme ayarları üzerinde maksimum kontrol için container içinde kendi TLS terminasyonunu implement etmeni öneriyor.
B. Container, transport layer security'yi doğrudan implement etmemelidir, çünkü HTTPS ve gRPC için TLS, Cloud Run tarafından şeffaf bir şekilde terminate edilir, istekler daha sonra container'a HTTP/1 ya da gRPC olarak proxy'lenir (ve HTTP/2 istekleri cleartext formatında handle edilir).
C. TLS yalnızca job'lar için geçerlidir, service'ler için hiç geçerli değildir, bu yüzden service'ler TLS'le ilgili tüm rehberliği tamamen görmezden gelebilir.
D. TLS, HTTPS isteklerinin Cloud Run tarafından şeffaf bir şekilde handle edilmesine rağmen, özellikle gRPC istekleri için container içinde doğrudan implement edilmelidir.

**4.** Cloud Run'ın iki execution environment'ı nedir, ve hangisi gerçekten değiştirilebilir?

A. Cloud Run'ın yalnızca tek bir execution environment'ı vardır, platformda hiçbir yerde "first" ve "second" generation arasında bir ayrım yoktur.
B. Her iki execution environment'ın da yalnızca job'lar için değiştirilebilir olduğu, service'ler için hiç değiştirilemediği belirtiliyor — bu, modülün gerçek rehberliğinin tersidir.
C. Execution environment seçiminin, uygulamanın programlama diline göre Cloud Run tarafından otomatik olarak yapıldığı, ne service'ler ne job'lar için manuel bir configuration'ın mümkün olmadığı belirtiliyor.
D. First generation ve second generation; first generation, service'ler için varsayılandır (ve değiştirilebilir), second generation ise job'lar için varsayılandır ve job'lar için **değiştirilemez** — iki seçenek arasında yalnızca service'ler için, servisinin ihtiyaçlarına göre seçim yapabilirsin.

**5.** Second generation execution environment, onu CPU-yoğun workload'lar ya da bir network dosya sistemi gerektiren workload'lar için uygun kılan hangi yetenekleri sağlar?

A. Second generation execution environment bu yeteneklerden hiçbirini sağlamaz — yalnızca seyrek trafikli servisler için cold start sürelerini azaltmak amacıyla vardır.
B. Second generation execution environment, tüm sistem çağrısı desteğini tamamen kaldırır, first generation'ın tam uyumluluğu yerine yalnızca emülasyona dayanır.
C. Daha hızlı CPU performansı, daha hızlı network performansı (özellikle paket kaybı olduğunda), tüm sistem çağrıları/namespace'ler/cgroup'lar dahil tam Linux uyumluluğu, ve network dosya sistemi desteği.
D. Second generation execution environment'ın, hiçbir ayırt edici yeteneği olmadan, her açıdan first generation ile aynı olduğu belirtiliyor.

**6.** Cloud Run'ın in-memory dosya sisteminin davranışı ve en iyi kullanımı nedir, ve bunun yerine Filestore gibi bir network dosya sistemi kullanmak için ne gerekir?

A. In-memory dosya sistemi yazılabilirdir ve container instance'ının tahsis edilmiş belleğini tüketir, ama yazılan veri instance durdurulduğunda persist etmez — bir cache olarak ya da atılabilir istek-başı veri için uygundur; Filestore ya da self-managed bir network dosya sistemi aracılığıyla standart dosya sistemi semantikleriyle bir instance'ın ömrünün ötesinde veri kalıcı kılmak için, deploy ederken second generation execution environment'ı belirtmelisin.
B. In-memory dosya sistemi salt okunurdur ve yazılan tüm veriyi, container instance durdurulduktan sonra bile, hiçbir ek configuration gerekmeden süresiz olarak otomatik olarak persist eder.
C. Filestore gibi network dosya sistemleri, hiçbir configuration değişikliği gerekmeden first generation execution environment'ta varsayılan olarak mevcuttur.
D. In-memory dosya sistemine yazılan veri, herhangi bir client kütüphanesi ya da açık kod dahil olmadan, otomatik ve kalıcı olarak Cloud Storage'a senkronize edilir.

**7.** Cloud Code, Kubernetes ve Cloud Run uygulamalarıyla çalışan geliştiriciler için ne sağlar?

A. Cloud Code'un, herhangi bir IDE'den tamamen ayrı, yalnızca bağımsız bir masaüstü uygulaması olarak mevcut, ücretli, kapalı kaynaklı bir ürün olduğu belirtiliyor.
B. Cloud Code yalnızca Kubernetes uygulamalarını destekler ve herhangi bir türde Cloud Run uygulaması desteğini açıkça hariç tutar.
C. Cloud Code'un, herhangi bir debug ya da örnek tabanlı destek sağlayabilmesi için önce tamamen deploy edilmiş bir production uygulaması gerektirdiği belirtiliyor.
D. Cloud Code, örnek şablonlardan yeni bir uygulama oluşturup özelleştirmekten, bitmiş uygulamayı çalıştırmaya kadar, Kubernetes ve Cloud Run uygulamalarının tüm geliştirme döngüsü için destek sağlayan, popüler IDE'ler (VS Code, IntelliJ, Cloud Shell) için bir dizi plugin'dir — configuration snippet'leri, özel bir debug deneyimi, ve log streaming/görüntüleme desteğiyle birlikte.

**8.** Bir container'ı Cloud Run'a deploy etmeden önce local'de test etmek için tanımlanan üç yol nedir?

A. Tek desteklenen local test yöntemi, doğrudan canlı, public erişilebilir bir Cloud Run servisine deploy edip production trafiğini gözlemlemektir.
B. Cloud Code (IDE içindeki Cloud Run emulator'ını kullanarak), gcloud CLI (kaynaktan build edebilen, container'ı local'de çalıştırabilen, ve kaynak kod değişikliklerinde otomatik olarak yeniden build edebilen bir local development environment içerir), ve Docker (image URL'i ve dinleme portuyla `docker run` kullanarak, `http://localhost:port/`'da test ederek).
C. Local testing'in, Cloud Run uygulamaları için tamamen desteklenmediği, her tek testin zaten deploy edilmiş bir revision'a karşı yapılması gerektiği belirtiliyor.
D. Local testing yalnızca fiziksel, on-premises bir Kubernetes cluster'ı kullanılarak gerçekleştirilebilir, gcloud CLI ya da Docker tabanlı local testing için herhangi bir destek tanımlanmıyor.

**9.** Modül, bir container image build etmek için hangi üç aracı tanımlıyor, ve Cloud Run'ın kendi source-based deployment'ı buna nasıl uyuyor?

A. Yalnızca Docker'ın bir container image build edebildiği belirtiliyor; Cloud Build ve Cloud Run'ın source-based deployment'ı mevcut olmayan özellikler olarak tanımlanıyor.
B. `gcloud run deploy --source`'un, kaynak koddan bir tane build etmek yerine yalnızca önceden build edilmiş container image'ları deploy edebildiği, bu da onun gerçek tanımlanan amacıyla çeliştiği belirtiliyor.
C. Docker (bir Dockerfile ile local'de build et ve `docker push` ile push et), Cloud Build (Google Cloud'da bir Dockerfile ya da Google Cloud's buildpacks ile `gcloud builds submit` üzerinden build et, buildpacks için `pack` flag'ini ekleyerek), ve Cloud Run'ın kendisi, burada `gcloud run deploy --source`, uygulama kaynağını (varsa) bir Dockerfile'la ya da Google Cloud's buildpacks ile build eder, sonuçtaki image'ı yükler, ve deploy eder.
D. Cloud Build'in fiziksel bir on-premises sunucu gerektirdiği, ve Cloud Run'ın source-based deployment'ının her zaman bir Dockerfile'ın mevcut olmasını gerektirdiği, buildpacks fallback'i olmadığı belirtiliyor.

**10.** Bir container Cloud Run'a deploy edilmeden önce, container image nerede saklanmalıdır, ve normalde image'larını desteklenmeyen bir yerde barındırıyorsan ne yaparsın?

A. Container image'lar, herhangi bir repository'ye hiç yüklenmeden doğrudan herhangi bir local geliştirici makinesinin diskinden deploy edilebilir.
B. Cloud Run'ın, container image'ların bir repository URL'i tarafından referans verilmesi yerine, doğrudan YAML deployment configuration dosyasının kendisine gömülmesini gerektirdiği belirtiliyor.
C. Cloud Run'ın yalnızca, tamamen Google tarafından kontrol edilen, tek, sabit bir public registry'den gelen image'ları desteklediği, private repository'ler ya da remote repository configuration'ı için hiçbir destek olmadığı belirtiliyor.
D. Container image, Cloud Run'ın erişebileceği bir repository'de saklanmalıdır — örneğin Artifact Registry'de (Google'ın önerisi) ya da Docker Hub'da, ya da bir Artifact Registry remote repository'si aracılığıyla bağlanan başka bir public/private registry'de; image'ları desteklenmeyen bir yerde barındırıyorsan, onları önce Artifact Registry'ye push etmen gerekir (örn. `docker push` ile).

**11.** Artifact Registry repository'leri hangi tür yazılım artifact'lerini saklayabilir, ve Cloud Run bir Docker repository'sinden bir container image pull ettikten sonra ne olur?

A. Artifact Registry repository'leri container image'ları (Docker repository), Node.js paketlerini (NPM repository), Java paketlerini (Maven repository), ve Python paketlerini (PyPI), ve diğerlerini saklayabilir; Cloud Run image'ı pull ettikten sonra, onu kendi internal storage'ında local olarak kopyalar ve saklar, büyük image'ların küçükler kadar hızlı yüklenmesini sağlar ve servisi, image'ın daha sonra yanlışlıkla Artifact Registry'den silinmesinden izole eder.
B. Artifact Registry'nin yalnızca container image'ları saklayabildiği ve NPM, Maven, ya da PyPI paketleri gibi başka herhangi bir yazılım paketi türünü açıkça saklayamadığı belirtiliyor.
C. Cloud Run bir image pull ettikten sonra, kendi kopyasını hemen atar ve her tek container başlangıcında doğrudan Artifact Registry'den yeniden pull eder, hiçbir türde local cache'leme olmadan.
D. Artifact Registry repository'lerinin yalnızca depolama için kullanılabildiği, build'lerden üretilen paketleri ve image'ları saklamak için Cloud Build ile hiçbir entegrasyonu olmadığı belirtiliyor.

**12.** Bir container image'ı Cloud Run'a deploy etmek için hangi role'ler gereklidir, ve belirli bir container image'ı yeni bir servise ilk kez deploy ettiğinde ne olur?

A. Cloud Run'a deploy etmek için hiçbir türde IAM role'üne ya da izne ihtiyaç yoktur — API endpoint'ine network erişimi olan herkes, hiçbir yetkilendirme kontrolü olmadan deploy edebilir.
B. Deploy etmek, Owner role'ünden, Editor role'ünden, ya da hem Cloud Run Admin hem Service Account User role'lerinden birine (ya da eşdeğer bir custom role'e) sahip olmayı gerektirir; bir container image'ın ilk deployment'ı hem servisi hem ilk revision'ını oluşturur, ve servis başına yalnızca bir container image vardır.
C. Bir container image'ın ilk deployment'ının yalnızca bir revision oluşturduğu, hiçbir zaman bir servis oluşturmadığı, servisin kendisinin tamamen ayrı, ilgisiz bir süreçle oluşturulması gerektiği belirtiliyor.
D. Bir container image deploy etmenin yalnızca Viewer role'ünü gerektirdiği, başka hiçbir role'ün (Owner ya da Editor dahil) bu amaç için yeterli olarak tanımlanmadığı belirtiliyor.

**13.** Cloud Run'da zaten çalışan bir uygulamayı güncellemenin genel adımları nelerdir?

A. Revision'ların hiçbir türde yeniden deployment'ı desteklemediği belirtildiği için, mevcut Cloud Run servisini tamamen sil ve her configuration ayarını sıfırdan elle yeniden oluştur.
B. Modül, revision'ların yalnızca container image değiştirildiği sürece oluşturulduktan sonra düzenlenebilir olduğunu tanımladığı için, zaten deploy edilmiş, immutable revision'ı doğrudan yerinde düzenle.
C. Modül, kod değişikliklerinin yürürlüğe girmesi için hiçbir zaman yeni bir image gerekmediğini belirttiği için, container image'ı hiç yeniden build etmeden ya da yeniden paketlemeden yalnızca YAML dosyasını değiştir.
D. Uygulama kaynak kodunu değiştir, onu bir container image'a build edip paketle, image'ı Artifact Registry'ye push et, container image'ı Cloud Run servisine yeniden deploy et, ve Cloud Run'ın değişiklikleri deploy etmesini bekle.

**14.** Bir Cloud Run servisinin configuration'ının sekiz bileşeni nedir, ve container image'ı değiştirmeden bunlardan birini bile değiştirirsen ne olur?

A. Yalnızca iki configuration bileşeni vardır — container image URL'i ve request timeout — ve ikisinden birini değiştirmenin revision'lar üzerinde hiçbir etkisi olmadığı belirtiliyor.
B. Sekiz bileşenin, servis ilk oluşturulduğunda kalıcı olarak sabitlendiği, sonraki deployment'larda hiçbirinin güncellenemediği belirtiliyor.
C. Container image URL'i, container entrypoint'i ve argümanları, secret'lar ve environment variable'lar, request timeout, concurrency, CPU/bellek limitleri, scaling boundaries, ve Google Cloud configuration'ı (service account, connector'lar); bu ayarlardan herhangi birini değiştirmek, container image'ın kendisi değişmese bile, yeni bir revision'ın oluşturulmasına yol açar.
D. Container image'ı değiştirmeden bir configuration ayarını değiştirmenin hiçbir etkisi olmadığı, yalnızca container image değişikliklerinin yeni revision'lar oluşturduğu belirtiliyor.

**15.** Bir Cloud Run service revision'ının immutable olması ne anlama gelir, ve bir revision gerçekte neyin bir kopyasıdır?

A. Immutable, container image'a dokunmadığın sürece, oluşturulduktan sonra bir revision'da sınırlı değişiklikler yapabileceğin anlamına gelir — bu, modüldeki terimin gerçek anlamıyla çelişir.
B. Bir revision, container image'ın ve service configuration'ının immutable bir kopyasıdır; "immutable", bir revision'ı oluşturulduktan sonra değiştiremeyeceğin anlamına gelir — daha fazla güncelleme yapmak için yalnızca yeni revision'lar ekleyebilirsin.
C. Bir revision'ın, service configuration'ının tamamen ayrı ve revision'larla ilgisiz olarak saklandığı, yalnızca container image'ın mutable, sürekli güncellenen bir kopyası olarak tanımlandığı belirtiliyor.
D. Revision'ların, eski revision'ların yanına eklenmek yerine, herhangi bir değişiklik yapıldığında otomatik olarak silinip yerinde değiştirildiği belirtiliyor.

**16.** Yeni bir service revision oluşturulduğunda, yeni revision hazır olarak onaylanmadan önce, geçiş sırasında trafiğe ve container instance'larına ne olur?

A. Cloud Run önce yeni revision'ı mevcut revision'ın kapasitesine kadar scale up eder ve container instance'larının başlamayı bitirmesini bekler; bu gerçekleşirken, mevcut (eski) revision'daki container instance'ları servise gelen istek trafiğini serve etmeye devam eder.
B. Yeni revision oluşturulduğu anda, container instance'larından hiçbiri başlamayı bitirmeden, tüm trafik ona geçer, bu da startup tamamlanana kadar her isteğin başarısız olmasına neden olur.
C. Mevcut revision, hazır olup olmadığından bağımsız olarak, yeni bir revision oluşturulduğu anda hemen kapatılır ve tüm container instance'ları sonlandırılır.
D. Trafik, instance'larının başlayıp başlamadığından bağımsız olarak, yeni revision oluşturulduğu ilk andan itibaren yeni ve mevcut revision'lar arasında eşit olarak %50/%50 bölünür.

**17.** Yeni bir revision'a başlangıçta sıfır trafik göndererek bir uygulama değişikliğinin kademeli rollout'unu gerçekleştirmeni sağlayan deployment seçeneği nedir, ve o revision'ın trafiğini zamanla nasıl artırırsın?

A. `--force-restart` flag'i, hiçbir kademeli seçenek olmadan yeni bir revision'a trafiğin %100'ünü anında yönlendirdiği belirtiliyor.
B. `--no-traffic` seçeneği, yeni bir service revision'ını deploy edildiğinde başlangıçta hiç trafik almayacak şekilde yapılandırır; ardından servisi, yeni revision'ın aldığı trafik miktarını kademeli olarak artırmak için artan bir yüzde değeri belirterek güncellersin.
C. `--max-instances=0` flag'i, bir revision'a trafiğin ulaşmasını kalıcı olarak önlemenin tek yolu olduğu, daha sonra artırmanın bir yolu olmadığı belirtiliyor.
D. Yeni bir revision'a başlangıç trafiğini kontrol etmek için tanımlanan bir deployment seçeneği yoktur — tüm yeni revision'ların her zaman oluşturulduğu anda trafiğin %100'ünü aldığı belirtiliyor.

**18.** Trafiği belirli bir service revision'ına pinlemek neyi başarır, ve nasıl yapılandırılır?

A. Pinning'in bir revision'ı tag'lemekle aynı olduğu, iki terimin birbirinin yerine kullanıldığı ve modülde hiçbir yerde aralarında işlevsel bir fark olmadığı belirtiliyor.
B. Trafiği pinlemek, yapılandırıldığı anda servisin diğer tüm revision'larını kalıcı olarak siler, yalnızca pinlenmiş revision'ı var olarak bırakır.
C. Trafiği pinlemenin, bir revision'ın trafik yüzdesinin 0'a ayarlanmasını gerektirdiği belirtiliyor — bu, özelliğin o revision'dan trafik serve etmeye devam etmesi gereken gerçek amacıyla çelişir.
D. İstek trafiğini pinlemek, yeni bir revision'ın deployment'ını trafik migration'ından ayırır — pinledikten sonra yeni bir revision eklersen, Cloud Run ona otomatik olarak trafik göndermez; bu, rollback ya da migration öncesi test için kullanışlıdır, ve pinlenen revision'ın trafik yüzdesini 100'e ayarlayarak başarılır.

**19.** Tag'lenmiş bir revision nedir, ve benzersiz URL formatı nedir?

A. Tag'lenmiş bir revision'a, hiçbir trafik serve etmeden onu belirli bir URL'de erişilebilir kılan bir tag atanır — trafik serve etmeden önce yeni bir revision'ı test etmek ve doğrulamak için kullanışlıdır; benzersiz URL'i, Cloud Run servisinin URL'idir, önek olarak eklenen tag adıyla birlikte — örneğin, `hello` servisinde `green` tag'i için `https://green---hello-xyz-uc.a.run.app`.
B. Tag'lenmiş bir revision, önce özel olarak test etme seçeneği olmadan, tag atandığı anda otomatik olarak production trafiğinin %100'ünü serve etmeye başlar.
C. Tag'lenmiş revision'ların, hiçbir ayırt edici önek ya da path olmadan, servisin varsayılan endpoint'iyle tam olarak aynı URL'i paylaştığı belirtiliyor.
D. Tag'ler yalnızca bir servisin ilk revision'ına atanabilir ve sonradan oluşturulan hiçbir sonraki revision'a asla atanamaz.

**20.** Trafik splitting birden fazla service revision'ı arasında nasıl çalışır, ve trafik yönlendirme değişiklikleri anlık mıdır?

A. Trafik splitting'in yalnızca aynı anda tam olarak iki revision arasında yapılandırılabildiği, splitting configuration'ları arasındaki herhangi bir geçişin tüm devam eden isteklerin düşürülmesiyle anında tamamlandığı belirtiliyor.
B. Trafik splitting yüzdelerinin, bir revision ilk oluşturulduğunda kalıcı olarak sabitlendiği, hiçbir arayüz üzerinden sonradan hiç ayarlanamadığı belirtiliyor.
C. Trafik splitting, birden fazla revision'ın her birine yapılandırılabilir bir yüzde istek yönlendirir (console, gcloud CLI, YAML, ya da Terraform üzerinden yapılandırılabilir); trafik yönlendirme ayarlamaları anlık değildir — o anda işlenmekte olan istekler tamamlanmaya devam eder ve düşürülmez, geçiş sırasında ya bir revision'a ya diğerine yönlendirilebilirler.
D. Trafik splitting'in önce session affinity'nin etkinleştirilmesini gerektirdiği, onsuz trafik bölmenin bir yolu olmadığı belirtiliyor.

**21.** Client kütüphaneleri Google Cloud API'lerini çağırmak için Cloud Run'ın built-in service account'unu kullandığında, bu hesap varsayılan olarak hangi role'e sahiptir, ve bunun yerine ne önerilir?

A. Built-in service account, geniş Project Editor role'üne sahiptir, bu da tüm Google Cloud API'lerini çağırabileceği ve projedeki neredeyse tüm kaynaklar üzerinde okuma/yazma erişimine sahip olduğu anlamına gelir; per-service identity kullanmak önerilir — minimal bir izinler kümesine sahip bir service account, örneğin servis yalnızca Firestore'dan okuyorsa yalnızca Firestore User role'ü.
B. Built-in service account'un varsayılan olarak hiçbir izne sahip olmadığı, ve onu kullanmanın servisten yapılan her API çağrısının her zaman başarısız olmasına neden olduğu belirtiliyor.
C. Built-in service account'un, servisin kodunun referans verdiği belirli kaynaklarla otomatik olarak sınırlandığı, daha geniş bir erişimin mümkün olmadığı belirtiliyor.
D. Per-service identity kullanmanın Cloud Run servisleri için desteklenmediği, built-in service account'un client kütüphanelerinin kullanabileceği tek hesap türü olduğu belirtiliyor.

**22.** Bir Cloud Run servisini bir Memorystore for Redis instance'ına bağlamanın adımları nelerdir, ve Cloud Run Integrations özelliği bunu nasıl basitleştirir?

A. Memorystore bağlantılarının, Redis instance'larının varsayılan olarak her zaman herhangi bir Cloud Run servisinden public erişilebilir olduğu için, hiçbir türde networking configuration'ı gerektirmediği belirtiliyor.
B. Adımlar şunlardır: Redis instance'ının yetkili VPC network'ünü belirle, Cloud Run servisiyle aynı region'da bir Serverless VPC Access connector'ı oluştur, ve connector'ı o VPC network'üne attach et (ardından `--vpc-connector` ve Redis host/port environment variable'larıyla deploy et); Integrations özelliği bunu tamamen otomatikleştirir, tam olarak yapılandırılmış bir Redis cache'i, yeni bir service revision'ı, ve gereken networking/environment variable configuration'ını otomatik olarak oluşturur.
C. Integrations özelliğinin, elle bağlanmaktan daha fazla manuel adım gerektirdiği, bu da onu manuel Serverless VPC Access sürecinden kesinlikle daha az kullanışlı yaptığı belirtiliyor.
D. Memorystore'a bağlanmanın yalnızca bir public IP adresi kullanılarak mümkün olduğu, Serverless VPC Access'in bu spesifik servis için açıkça desteklenmediği belirtiliyor.

**23.** Bir Pub/Sub push subscription'ı bir Cloud Run servisini nasıl tetikler, endpoint'in public olması gerekir mi, ve servis bir mesajı zamanında acknowledge etmezse ne olur?

A. Push subscription'ların, hiçbir IAM koruması mümkün olmadan hedef Cloud Run endpoint'inin her zaman public erişilebilir olmasını gerektirdiği belirtiliyor.
B. Cloud Run'a teslim edilen Pub/Sub mesajları için tanımlanan bir acknowledgement deadline'ı yoktur — mesajların hiçbir koşulda yeniden teslim edilmediği belirtiliyor.
C. Pub/Sub'ın yalnızca Cloud Run job'larını tetikleyebildiği, hiçbir zaman service'leri tetikleyemediği belirtiliyor — bu, modülün bir servis endpoint'ini hedefleyen push subscription'lara dair gerçek tanımıyla çelişiyor.
D. Bir push subscription, mesajları servisin endpoint'ine HTTP istekleri olarak teslim eder; endpoint IAM ile korunabilir ve public olmasına gerek yoktur; servis, 600 saniye içinde (maksimum acknowledgement deadline) bir yanıt döndürerek mesajı acknowledge etmelidir, aksi halde Pub/Sub mesajı yeniden teslim eder, bu da servisin tekrar tetiklenmesine neden olur.

**24.** Bir Cloud Run servisi, public bir IP adresi üzerinden ile private bir IP adresi üzerinden bir Cloud SQL instance'ına tipik olarak nasıl bağlanır, ve modül bağlantıyı yönetmek için hangi best practice'leri öneriyor?

A. Cloud SQL'e public ve private IP bağlantılarının, modülde hiçbir yerde bir ayrım yapılmadan, tam olarak aynı mekanizmayı kullandığı belirtiliyor.
B. Bir Cloud Run servisinin bir Cloud SQL veritabanına bağlanması için tanımlanan bir bağlantı sınırı yoktur, ve bir servisin kaç bağlantı açtığından bağımsız olarak connection pool'ların gereksiz olduğu belirtiliyor.
C. Public bir IP üzerinden, Cloud Run şifreleme sağlayarak Cloud SQL Auth proxy'yi kullanarak (network socket'ları ya da bir Cloud SQL connector'ı kullanarak) bağlanır; private bir IP üzerinden, servis egress trafiğini bir Serverless VPC Access connector'ı üzerinden yönlendirir; best practice'ler, veritabanı credential'larını Secret Manager'da saklamayı (environment variable olarak ya da mount edilmiş bir volume olarak geçirilir) ve otomatik yeniden bağlanmak ve bağlantı sayısını sınırlamak için bir connection pool kullanmayı içerir (Cloud Run servisleri, bir Cloud SQL veritabanına servis başına 100 bağlantıyla sınırlıdır).
D. Modülün, Secret Manager'ın Cloud SQL bağlantılarıyla uyumsuz olduğu belirtildiği için, veritabanı credential'larını Secret Manager kullanmak yerine doğrudan uygulamanın kaynak koduna sabit kodlamayı önerdiği belirtiliyor.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: A.**
Cloud Run için iyi bir seçim olmak için, bir uygulama HTTP, HTTP/2, WebSockets, ya da gRPC üzerinden teslim edilen istekleri, stream'leri, ya da event'leri serve etmelidir (ya da tamamlanana kadar çalışmalıdır), local persistent bir dosya sistemi gerektirmemelidir (ephemeral local ya da network dosya sistemi uygundur), birden fazla instance'ın aynı anda çalışmasını handle edecek şekilde build edilmelidir, instance başına 8 CPU ve 32 GiB bellekten fazlasını gerektirmemelidir, ve containerized olmalı, Go, Java, Node.js, Python, ya da .NET'te yazılmış olmalı, ya da başka şekilde containerize edilebilir olmalıdır.

**2. Doğru cevap: C.**
Bir Cloud Run service'i olarak çalışırken, container doğru portta istekleri dinlemeli ve yapılandırılmış request timeout ayarı içinde (container başlangıç süresi dahil maks. 1 saat) bir yanıt göndermelidir — aksi halde istek sonlanır ve bir 504 hatası döndürülür. Cloud Run job'ları için, container başarıyla tamamlandığında 0 exit code'uyla, başarısız olduğunda sıfır olmayan bir exit code'la çıkmalıdır; job'lar istek serve etmemesi gerektiğinden, container bir portu dinlememeli ya da bir web sunucusu başlatmamalıdır.

**3. Doğru cevap: B.**
Container, transport layer security'yi doğrudan implement etmemelidir, çünkü HTTPS ve gRPC için TLS Cloud Run tarafından terminate edilir — istekler daha sonra container'a HTTP/1 ya da gRPC olarak proxy'lenir. HTTP/2 için, container isteklerini HTTP/2 cleartext formatında handle etmelidir.

**4. Doğru cevap: D.**
Cloud Run'ın iki execution environment'ı vardır: first generation (varsayılan olarak service'ler için kullanılır, ve değiştirilebilir) ve second generation (varsayılan olarak job'lar için kullanılır, ve job'lar için değiştirilemez). Execution environment'ı yalnızca service'ler için değiştirebilirsin, servisinin ihtiyaçlarına göre ikisi arasında seçim yaparak.

**5. Doğru cevap: C.**
Second generation execution environment, daha hızlı CPU performansı, daha hızlı network performansı (özellikle paket kaybı olduğunda), tüm sistem çağrıları, namespace'ler, ve cgroup'lar dahil tam Linux uyumluluğu, ve network dosya sistemi desteği sağlar.

**6. Doğru cevap: A.**
Cloud Run'da, container'ın yazılabilir bir in-memory dosya sistemine erişimi vardır; ona yazmak container instance'ının tahsis edilmiş belleğini kullanır, ve ona yazılan veri container instance'ı durdurulduğunda persist etmez — bir cache olarak ya da atılabilir istek-başı veri ya da configuration saklamak için kullanılabilir. Filestore ya da başka bir self-managed network dosya sistemi aracılığıyla standart dosya sistemi semantikleri kullanarak bir container instance'ın ömrünün ötesinde veri kalıcı kılmak için, servisini deploy ederken second generation execution environment'ı belirtmelisin.

**7. Doğru cevap: D.**
Cloud Code, örnek şablonlardan yeni bir uygulama oluşturup özelleştirmekten, bitmiş uygulamayı çalıştırmaya kadar, Kubernetes ve Cloud Run uygulamalarının tam geliştirme döngüsü için IDE desteği sağlayan, popüler IDE'ler (VS Code, IntelliJ, Cloud Shell) için bir dizi plugin'dir — örnekler, configuration snippet'leri, özel bir debug deneyimi, ve log streaming ve görüntüleme desteği sağlar.

**8. Doğru cevap: B.**
Bir container'ı local'de, Cloud Code (IDE içinde CPU/bellek, environment variable'lar, ve Cloud SQL bağlantılarını yapılandırmana izin veren Cloud Run emulator'ıyla), gcloud CLI (bir container'ı kaynaktan build edebilen, local'de çalıştırabilen, ve kaynak kod değişikliklerinde otomatik olarak yeniden build edebilen, Cloud Run'ı emüle eden bir local development environment içerir — `http://localhost:8080/`'da test et), ya da Docker (container image URL'i ve uygulamanın dinlediği portla `docker run` komutunu kullanarak — `http://localhost:port/`'da test et) kullanarak test edebilirsin.

**9. Doğru cevap: C.**
Bir container image'ı local'de Docker ile bir Dockerfile kullanarak (`docker build`, ardından bir repository'ye yüklemek için `docker push`), Google Cloud'da Cloud Build ile bir Dockerfile ya da Google Cloud's buildpacks kullanarak (`gcloud builds submit`, buildpacks kullanmak için `pack` flag'ini ekleyerek), ya da Cloud Run'ın kendi source-based deployment'ı aracılığıyla build edebilirsin — burada `--source` flag'iyle `gcloud run deploy` komutu, uygulama kaynak kodunu (varsa) bir Dockerfile'la ya da Google Cloud's buildpacks ile build eder, sonuçtaki container image'ı bir image repository'sine yükler, ve Cloud Run'a deploy eder.

**10. Doğru cevap: D.**
Bir container Cloud Run'a deploy edilmeden önce, container image'ın Cloud Run'ın erişebileceği bir repository'de saklanması gerekir — Artifact Registry'de (Google'ın önerdiği) ya da Docker Hub'da saklanan image'ları, ya da bir Artifact Registry remote repository'si kurarak diğer public ya da private registry'lerden gelen image'ları kullanabilirsin. Container image'larını genellikle başka bir yerde, desteklenmeyen bir public ya da private container registry'sinde barındırıyorsan, onları önce Artifact Registry'ye push etmen gerekir, bunu `docker push` komutuyla yapabilirsin.

**11. Doğru cevap: A.**
Artifact Registry, container image'lar (bir Docker repository'sinde) ile Node.js paketleri (NPM repository), Java paketleri (Maven repository), ve Python paketleri (PyPI) dahil, yazılım artifact'lerini private repository'lerde saklamak ve yönetmek için kullanılan universal bir package manager servisidir. Cloud Run bir container image pull ettikten sonra, image'ı internal container storage'ında local olarak kopyalar ve saklar — bu hızlıdır ve image boyutunun container başlangıç süresini etkilememesini sağlar (büyük image'lar küçükler kadar hızlı yüklenir); Cloud Run image'ı kopyaladığı için, deploy edilmiş bir container image'ı yanlışlıkla Artifact Registry'den silsen bile sorun olmaz.

**12. Doğru cevap: B.**
Deploy edebilmek için, Owner ya da Editor role'lerinden birine, ya da hem Cloud Run Admin hem Service Account User role'lerine (ya da gereken izinleri içeren bir custom role'e) sahip olmalısın. Bir container image'ı ilk kez deploy ettiğinde, Cloud Run bir servis ve onun ilk revision'ını oluşturur — servis başına yalnızca bir container image vardır.

**13. Doğru cevap: D.**
Cloud Run'da bir uygulamayı güncellemek için, genellikle: uygulama kaynak kodunu değiştirirsin, uygulamanı bir container image'a build edip paketlersin, container image'ı Artifact Registry'ye push edersin, container image'ı Cloud Run servisine yeniden deploy edersin, ve Cloud Run'ın değişikliklerini deploy etmesini beklersin — container image'ını mevcut bir servise yeniden deploy ettiğinde, otomatik olarak yeni bir revision oluşturulur.

**14. Doğru cevap: C.**
Bir Cloud Run servisinin configuration'ı sekiz bileşenden oluşur: container image URL'i, container entrypoint'i ve argümanları, secret'lar ve environment variable'lar, request timeout, concurrency, CPU/bellek limitleri, scaling boundaries, ve Google Cloud configuration'ı (service account ve connector'lar gibi). Servisin herhangi bir configuration ayarını değiştirmek, container image'ın kendisinde bir değişiklik olmasa bile, yeni bir revision'ın oluşturulmasına yol açar.

**15. Doğru cevap: B.**
Bir revision, container image'ın ve service configuration'ının immutable bir kopyasıdır. "Immutable", bir revision'ı oluşturulduktan sonra değiştiremeyeceğin anlamına gelir — daha fazla güncelleme yapmak için yalnızca yeni revision'lar ekleyebilirsin, ve Cloud Run, service resource'a her deploy ettiğin değişiklikte bu immutable kopyayı oluşturur.

**16. Doğru cevap: A.**
Yeni bir service revision oluşturulduktan sonra, Cloud Run önce yeni revision'ı mevcut revision'ın kapasitesine kadar scale up eder, ve o revision'daki container instance'larının başlamayı bitirmesini bekler. Bu gerçekleşirken, mevcut (önceki) revision'daki container instance'ları servise gelen istek trafiğini serve etmeye devam eder.

**17. Doğru cevap: B.**
Bir uygulama değişikliğinin kademeli rollout'unu gerçekleştirmek için, yeni bir service revision, `--no-traffic` seçeneğiyle deploy edildiğinde başlangıçta hiç trafik almayacak şekilde yapılandırılabilir. Yeni service revision'ın aldığı trafik miktarını kademeli olarak artırmak için, ardından servisi artan bir yüzde değeri belirterek güncelleyebilirsin.

**18. Doğru cevap: D.**
İstek trafiğini belirli bir service revision'ına pinlemek (en son revision yerine), yeni bir revision'ın deployment'ını trafiğin migration'ından ayırır — bu, yeni bir revision eklersen, Cloud Run'ın ona otomatik olarak trafik göndermeyeceği anlamına gelir. Bu, önceki bir revision'a geri dönmek istediğinde, ya da tüm istek trafiğini ona taşımadan önce yeni bir revision'ı test etmek istediğinde kullanışlıdır, ve revision'a giden istek trafiği yüzdesini 100'e ayarlayarak başarılır.

**19. Doğru cevap: A.**
Bir servis deploy ederken, yeni revision'a, o revision'ı trafik serve etmeden belirli bir URL'de erişilebilir kılan bir tag atayabilirsin — trafik serve etmeden önce yeni bir revision'ı test etmek ve doğrulamak için yaygın olarak kullanılır. Tag'lenmiş bir revision'ın kendi benzersiz URL'i vardır: Cloud Run servisinin URL'i, önek olarak eklenen tag adıyla birlikte — örneğin, `hello` servisinde bir revision'ı `green` olarak tag'lemek, `https://green---hello-xyz-uc.a.run.app` test URL'ini verir.

**20. Doğru cevap: C.**
Cloud Run, hangi service revision'larının trafik alacağını ve her revision'ın aldığı trafik yüzdelerini belirlemeni sağlar, bir yüzde değeri belirterek istek trafiğini birden fazla service revision arasında bölmeni sağlar (console, gcloud CLI, bir YAML configuration dosyası, ya da Terraform üzerinden yapılandırılabilir). Trafik yönlendirme ayarlamaları anlık değildir — trafik splitting configuration'ını değiştirdiğinde, o anda işlenmekte olan istekler tamamlanmaya devam eder ve düşürülmez, ve geçiş dönemi sırasında ya yeni ya da önceki bir revision'a yönlendirilebilir.

**21. Doğru cevap: A.**
Client kütüphaneleri, Google Cloud servisleriyle şeffaf bir şekilde kimlik doğrulamak için built-in service account'u kullanır; bu service account, Project Editor role'üne sahiptir, bu da tüm Google Cloud API'lerini çağırabileceği ve projedeki tüm kaynaklar üzerinde okuma ve yazma erişimine sahip olduğu anlamına gelir. Bir Cloud Run servisinin erişebileceği API'leri ve kaynakları kısıtlamak için per-service identity kullanmak önerilir — örneğin, servis yalnızca Firestore'dan veri okuyorsa yalnızca Firestore User IAM role'ünü atamak.

**22. Doğru cevap: B.**
Bir Cloud Run servisinden bir Memorystore for Redis instance'ına bağlanmak için, Redis instance'ının yetkili VPC network'ünü belirlersin, Cloud Run servisiyle aynı region'da bir Serverless VPC Access connector'ı oluşturursun, ve connector'ı Redis instance'ının yetkili VPC network'üne attach edersin — ardından servisi, connector adını ve Redis instance'ının host ve port'u için environment variable'ları belirterek deploy edersin. Integrations özelliği bunu otomatikleştirir: yapılandırılabilir bir bellek boyutuyla tam olarak yapılandırılmış bir Redis cache'i otomatik olarak oluşturur, yeni bir Cloud Run service revision'ı oluşturur, ve servisin Redis cache'ine erişmesi için gereken networking ve environment variable'ları yapılandırır.

**23. Doğru cevap: D.**
Pub/Sub, mesajları bir Cloud Run servisinin endpoint'ine push edebilir, burada mesajlar daha sonra container'lara HTTP istekleri olarak teslim edilir; endpoint IAM ile korunabilir ve public olmasına gerek yoktur. Cloud Run servisi, Pub/Sub mesajını 600 saniye içinde (maksimum acknowledgement deadline) bir yanıt döndürerek acknowledge etmelidir, aksi halde Pub/Sub mesajı yeniden teslim eder, bu da servisin tekrar tetiklenmesine neden olur.

**24. Doğru cevap: C.**
Public bir IP adresi için (Cloud SQL'in varsayılanı), Cloud Run şifreleme sağlar ve network socket'ları ya da bir Cloud SQL connector'ı aracılığıyla Cloud SQL Auth proxy'yi kullanarak bağlanır; private bir IP adresi için, servis tüm egress trafiğini bir Serverless VPC Access connector'ı üzerinden Cloud SQL instance'ına yönlendirebilir. Best practice'ler arasında, hassas veritabanı credential'larını saklamak için Secret Manager kullanmak (uygulamaya environment variable olarak ya da bir volume olarak mount edilerek geçirilir) ve uygulama kodunda, bozulan bağlantıları otomatik olarak yeniden bağlayan ve servisin kullandığı maksimum bağlantı sayısını sınırlamana izin veren connection pool'lar kullanmak yer alır — Cloud Run servisleri, bir Cloud SQL veritabanına servis başına 100 bağlantıyla sınırlıdır.
