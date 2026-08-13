# Sözlük (Türkçe)

Bu sözlük, 12 deep-dive modülünün tamamında geçen terimleri, servisleri ve kavramları hızlı bir referansa dönüştürür. Girdiler alfabetik olarak düzenlenmiştir. Her girdi [glossary/README.md](README.md) dosyasında tanımlanan formatı izler: Tanım, Neden var, İlgili servisler, Yaygın yanlış anlamalar, Gerçek dünya benzetmesi.

Bu bir referans belgesidir, öğretici değil — herhangi bir girdinin arkasındaki tam öğretici bağlam için ilgili [deep-dive](../deep-dive) modülüne bakın.

Bu sözlük, `glossary.en.md` ile **girdi girdi eşleşir** — aynı terimler, aynı sıra, aynı yapı. İçerik bağımsız yazılmamış, İngilizce sürümden çevrilmiştir.

---

### Application Default Credentials (ADC)

**Tanım:** Cloud Client Library'lerin, Google Cloud API'lerini çağırırken hangi kimlik bilgisini kullanacağını, uygulama kodu bunu açıkça belirtmeden otomatik olarak keşfetmesini sağlayan mekanizma.

**Neden var:** Aynı kodun bir dizüstü bilgisayarda, Cloud Run'da ya da GKE'de değişmeden çalışabilmesi için — sadece kimlik bilgisi kaynağı değişir, kod asla değişmez.

**İlgili servisler:** Service Account, Workload Identity, Cloud Client Libraries, Secret Manager.

**Yaygın yanlış anlamalar:** `gcloud auth login` ile `gcloud auth application-default login` sık karıştırılır. Birincisi gcloud CLI'nin kendisini doğrular; ikincisi API çağrısı yapan kod için ADC'yi besler.

**Gerçek dünya benzetmesi:** ADC, seni otomatik olarak tanıyan ve hangi kapıdan girdiğine bakmaksızın doğru kapıları açan bir otel anahtar kartı sistemi gibidir.

---

### AlloyDB

**Tanım:** Hem işlemsel hem analitik iş yüklerinde (HTAP) yüksek performans için compute'u depolamadan ayıran, tam yönetilen, PostgreSQL uyumlu bir veritabanı servisi.

**Neden var:** Klasik PostgreSQL tek bir VM üzerinde, ona bağlı bir diskle çalışır ve ölçeklemesi zordur. AlloyDB, uyumluluktan ödün vermeden PostgreSQL'e Google ölçeğinde performans getirir.

**İlgili servisler:** Cloud SQL, Spanner, BigQuery.

**Yaygın yanlış anlamalar:** AlloyDB genel amaçlı bir ilişkisel veritabanı değildir — Cloud SQL'in aksine yalnızca PostgreSQL motorunu destekler (MySQL veya SQL Server'ı desteklemez). Ayrıca Cloud SQL'in zaten yeterli olduğu küçük, basit uygulamalar için doğru seçim değildir.

**Gerçek dünya benzetmesi:** Cloud SQL güvenilir bir aile arabasıysa, AlloyDB aynı arabanın yarış ayarlı bir motorla yeniden mühendislenmiş hâlidir — aynı kumandalar, çok daha fazla performans.

---

### API Key

**Tanım:** Bir Google Cloud API'sini çağıran uygulamayı tanımlayan, öncelikle faturalandırma ve kota ilişkilendirmesi için kullanılan bir karakter dizisi.

**Neden var:** Tam bir kimlik akışı gerektirmeden, bir API çağrısını bir projeyle ilişkilendirmenin hafif bir yolunu sağlamak için.

**İlgili servisler:** OAuth 2.0, Service Account, IAM.

**Yaygın yanlış anlamalar:** Bir API key, çağıranı doğrulamaz — bir kişiyi ya da workload'ı değil, bir uygulamayı tanımlar. Ele geçirilmiş bir key uzun ömürlü ve büyük ölçüde kısıtlanmamış erişim sağlar; bu yüzden çoğu Google API'si onları kabul bile etmez.

**Gerçek dünya benzetmesi:** API key, vestiyer fişine benzer — hangi askıya ücret yazılacağını kanıtlar ama kim olduğunu kanıtlamaz.

---

### Apigee (API Gateway)

**Tanım:** Google Cloud'un API geliştirme, güvenceye alma ve yönetme platformu; arka uç servislerin önünde bir proxy katmanı olarak durur.

**Neden var:** Tüketici uygulamalara, arka uç işlevselliği üzerinde kontrollü bir cephe sunmak için — arka ucu değiştirmeden güvenlik, oran sınırlama, kota ve analitik eklemek.

**İlgili servisler:** Identity-Aware Proxy, Cloud Run, Cloud Load Balancing.

**Yaygın yanlış anlamalar:** Apigee sadece bir load balancer değildir — yönlendirmenin üzerine bir yönetişim ve analitik katmanı ekler. Özellikle yeniden yazılamayan eski sistemlere modern bir API yüzü koymak için kullanışlıdır.

**Gerçek dünya benzetmesi:** Apigee bir otelin konsiyerj masası gibidir — misafirler mutfak, çamaşırhane ya da bakım ekipleriyle asla doğrudan muhatap olmaz; hepsi tek bir kontrollü ön masadan geçer.

---

### App Engine

**Tanım:** Google Cloud'un Standard ve Flexible ortamlarıyla gelen, orijinal, tam yönetilen Platform as a Service (PaaS) ürünü.

**Neden var:** Konteyner tabanlı serverless platformlar var olmadan önce, geliştiricilerin sunucu yönetmeden uygulama kodu dağıtabilmesini sağlamak için.

**İlgili servisler:** Cloud Run, Platform as a Service (PaaS).

**Yaygın yanlış anlamalar:** App Engine hâlâ var ve destekleniyor, ama yeni servisler için **önerilen** başlangıç noktası **değildir** — Cloud Run daha fazla esneklik, daha hızlı ölçekleme ve daha iyi entegrasyonlar sunar; ayrıca App Engine'i işletmenin öğrettiği derslerin üzerine inşa edilmiştir.

**Gerçek dünya benzetmesi:** App Engine, on yıl önceden kalma, tam donanımlı bir servisli ofis gibidir — konforludur, ama yan taraftaki yeni bina (Cloud Run) daha iyi olanaklara ve daha esnek kiralara sahiptir.

---

### Artifact Analysis

**Tanım:** Artifact Registry'deki build yapıtlarını (temel container imajları, Maven ve Go paketleri) bilinen güvenlik açıkları için hem sürekli hem talep üzerine tarayan bir servis.

**Neden var:** Bugün güvenli olan bir imaj, yarın yeni keşfedilen bir açık içeren bir bağımlılık barındırabilir — sürekli tarama, yalnızca push anında yapılan taramanın kaçıracağı sorunları yakalar.

**İlgili servisler:** Artifact Registry, Binary Authorization, Software Delivery Shield.

**Yaygın yanlış anlamalar:** Artifact Analysis sadece gözlemler ve raporlar — güvenlik açığı olan bir imajın dağıtılmasını engellemez. Zorunlu kılma (enforcement) işi Binary Authorization'a aittir.

**Gerçek dünya benzetmesi:** Artifact Analysis, sadece kamyondan inen ürünleri değil, rafta zaten duran ürünleri de tekrar tekrar test eden bir gıda denetçisidir.

---

### Artifact Registry

**Tanım:** Container imajları ve dil paketleri gibi build yapıtlarını saklamak, güvenceye almak ve yönetmek için yönetilen bir servis.

**Neden var:** CI/CD hatlarının, bir derlemenin çıktısını dağıtılmadan önce saklayacak güvenilir, merkezi bir yere ihtiyacı vardır.

**İlgili servisler:** Cloud Build, Artifact Analysis, Cloud Deploy, Cloud Run, GKE.

**Yaygın yanlış anlamalar:** Sadece Docker imajları için değildir — Maven, npm ve diğer dile özel paketleri de saklar.

**Gerçek dünya benzetmesi:** Artifact Registry, mağazalara (üretime) gönderilmeden önce bitmiş, denetlenmiş ürünlerin beklediği bir depo yükleme rıhtımıdır.

---

### Assured Open Source Software (Assured OSS)

**Tanım:** Google tarafından inşa edilen ve sürekli taranan Java ve Python açık kaynak paketlerini sunan, geliştiricilerin doğrulanmış bağımlılıkları tüketmesini sağlayan bir servis.

**Neden var:** Çoğu uygulama, kimsenin aktif olarak güvenceye almadığı açık kaynak paketlere bağımlıdır; Assured OSS bu sorumluluğu Google'a devreder.

**İlgili servisler:** Artifact Analysis, Cloud Build, Software Delivery Shield.

**Yaygın yanlış anlamalar:** Yalnızca **Java ve Python** paketlerini kapsar — her dil ekosistemini değil.

**Gerçek dünya benzetmesi:** Assured OSS, doğrulanmamış bir semt pazarından değil, kendi gıda güvenliği laboratuvarını işleten bir tedarikçiden market alışverişi yapmaya benzer.

---

### AutoML (Agent Platform AutoML)

**Tanım:** Derin ML uzmanlığı olmadan, kendi verinle (görsel, tablo verisi, video) özel makine öğrenmesi modelleri eğitmenin kodsuz bir yolu.

**Neden var:** Önceden eğitilmiş API'ler genel amaçlıdır; bazen verin, özel bir model gerektirecek kadar spesifiktir ama elinde ML mühendisi yoktur.

**İlgili servisler:** Vision AI, TensorFlow, PyTorch, Generative AI.

**Yaygın yanlış anlamalar:** AutoML, TensorFlow/PyTorch ile model inşa etmekle aynı şey değildir — sıfır ML uzmanlığı karşılığında bir miktar esneklikten ödün verir. Bir spektrumun ortasındadır, uçlarından biri değil.

**Gerçek dünya benzetmesi:** Önceden eğitilmiş bir API mağazadan hazır alınan bir takım elbise, özel bir TensorFlow modeli bir terzide ısmarlama diktirilen takımsa, AutoML tam ölçü alınarak yapılan hazır konfeksiyon gibidir — sana özel ama kendi terzine ihtiyacın olmadan.

---

### Autoscaling

**Tanım:** Yüke göre compute örneklerinin (VM, pod, container) otomatik olarak eklenmesi ya da kaldırılması.

**Neden var:** Trafiğe uyacak şekilde altyapıyı elle yeniden boyutlandırmak yavaş ve hataya açıktır; autoscaling, bulutun elastikliğinin bunu otomatik yapmasını sağlar.

**İlgili servisler:** Managed Instance Group, Cloud Load Balancing, GKE, Cloud Run.

**Yaygın yanlış anlamalar:** Dışa (out) ölçeklenmek (daha fazla makine eklemek, yatay) bulutun doğal eğilimidir ve yukarı (up) ölçeklenmekten (tek makineyi büyütmek, dikey) farklıdır — çoğu iş yükü varsayılan olarak dışa ölçeklenmeye göre tasarlanmalıdır.

**Gerçek dünya benzetmesi:** Autoscaling, akşam yemeği yoğunluğu başladığında otomatik olarak daha fazla personel çağıran ve sakinleşince insanları eve gönderen bir restoran gibidir.

---

### Background Function

**Tanım:** Event-driven bir Cloud Run function'ının daha eski stil implementasyonu; event verisini event türüne göre alır, yalnızca Cloud Run functions (1st gen) tarafından Node.js, Python, Go ve Java runtime'larında desteklenir.

**Neden var:** CloudEvents standardı ve Functions Framework, event-driven function'ları generation'lar ve diller arasında birleştirmeden önce, Cloud Functions 1st generation'ın event trigger'ları ele almasının orijinal yoluydu.

**İlgili servisler:** Cloud Run Functions, CloudEvent Function, Functions Framework, Eventarc.

**Yaygın yanlış anlamalar:** Bir Background function, bugün serbestçe seçilebilecek bir alternatif stil değildir — belirli birkaç runtime'da özellikle Cloud Run functions 1st gen'e bağlıdır. Cloud Run functions'da (2nd gen) yeni event-driven function'lar, desteklenen her dilde bunun yerine CloudEvent functions kullanır.

**Gerçek dünya benzetmesi:** Bir CloudEvent function herkesin anladığı standart, çevrilmiş bir dil konuşuyorsa, bir Background function yalnızca belirli, daha eski bir araç filosunun ayarlamayı bildiği eski bir telsiz sevkiyat sistemi gibidir.

---

### BigQuery

**Tanım:** Devasa veri kümeleri üzerinde SQL analitiği çalıştırmak için tam yönetilen, serverless bir kurumsal veri ambarı.

**Neden var:** İş zekâsı, canlı bir işlem uygulamasından farklı bir araç gerektirir — tek satır aramak için değil, terabayt-petabayt ölçeğinde tarayıp toplu içgörüler üretmek için tasarlanmış bir araç.

**İlgili servisler:** Bigtable, Cloud Storage, AlloyDB.

**Yaygın yanlış anlamalar:** İsim benzerliği yüzünden **Bigtable** ile sık karıştırılır — aslında zıttırlar. BigQuery bir OLAP veri ambarıdır; Bigtable operasyonel, düşük gecikmeli bir NoSQL depodur. BigQuery ayrıca milisaniyelik tekil satır okuma/yazma için tasarlanmamıştır.

**Gerçek dünya benzetmesi:** BigQuery, milyonlarca belgede aynı anda örüntü arayabildiğin bir araştırma kütüphanesidir — açıp tek bir klasörü anında çektiğin bir dosya dolabı değil.

---

### Bigtable

**Tanım:** Devasa, seyrek doldurulmuş tablolar için, muazzam ölçekte sub-10ms key-value aramaları sunan yüksek performanslı bir NoSQL veritabanı.

**Neden var:** Bazı iş yükleri (tıklama akışları, IoT, zaman serisi, reklam gösterimleri) milyarlarca satır ve son derece düşük okuma/yazma gecikmesi gerektirir — ilişkisel veritabanları bu ölçekte boğulur.

**İlgili servisler:** BigQuery, Firestore, Cloud Storage.

**Yaygın yanlış anlamalar:** İsim benzerliği yüzünden BigQuery ile karıştırılır; ikisi de "NoSQL" olduğu için Firestore ile de karıştırılır. Bigtable tek-anahtarlıdır, operasyoneldir, sub-10ms'dir; SQL sorgularını ya da çok satırlı işlemleri desteklemez.

**Gerçek dünya benzetmesi:** Bigtable, tam anahtarıyla herhangi bir satıra anında atlayabildiğin, çoğu boş, devasa bir tablo gibidir — ama satırlar arası ilişkiler hakkında karmaşık sorular soramazsın.

---

### Binary Authorization

**Tanım:** Container imajlarının dağıtılmadan önce belirli attestation'lar (gerekli süreçlerden geçtiğinin kanıtı) taşımasını zorunlu kılan bir politikayı uygulayan servis.

**Neden var:** Bir imajda güvenlik açığı olduğunu bilmek (Artifact Analysis) onun çalışmasını durdurabilmekle aynı şey değildir — Binary Authorization politikayı zorunlu bir kapıya dönüştürür.

**İlgili servisler:** Artifact Analysis, Cloud Deploy, Software Delivery Shield.

**Yaygın yanlış anlamalar:** Binary Authorization zorunluluk katmanıdır, tarama katmanı değil — kendisi güvenlik açığı bulmaz, gerekli attestation'ların var olup olmadığını kontrol eder.

**Gerçek dünya benzetmesi:** Binary Authorization, kapıda kimlik kartlarını kontrol eden güvenlik görevlisidir — arka plan kontrolünü kendisi yapmaz, sadece doğru damgası olmayan kimseyi içeri almaz.

---

### Bucket (Cloud Storage Bucket)

**Tanım:** Cloud Storage'daki nesneleri organize eden kap; her bucket küresel olarak benzersiz bir isme ve belirli bir coğrafi konuma ihtiyaç duyar.

**Neden var:** Nesne depolamanın bir ad alanına ve bir konum sınırına ihtiyacı vardır — bucket'lar ikisini de sağlar.

**İlgili servisler:** Cloud Storage, Cloud Storage Classes.

**Yaygın yanlış anlamalar:** Bir bucket'taki nesneler değişmezdir (immutable) — yerinde düzenlemezsin, yeni bir versiyon oluşturursun (ya da Object Versioning açık değilse üzerine yazarsın).

**Gerçek dünya benzetmesi:** Bir bucket, kendi kendine depolama tesisindeki etiketli bir depolama birimidir — içindeki her şey benzersiz isimle organize edilir ve birimin kendisi sabit bir fiziksel konuma sahiptir.

---

### Circuit Breaker

**Tanım:** Sürekli başarısız olan bir servise trafik göndermeyi, sonsuza kadar yeniden denemek yerine durduran bir dayanıklılık deseni.

**Neden var:** Gerçekten bozuk bir bağımlılığı yeniden denemek hem kendi kaynaklarını boşa harcar hem de başarısız servisi daha da yükler.

**İlgili servisler:** Cloud Client Libraries, Service.

**Yaygın yanlış anlamalar:** Circuit breaker'lar *kalıcı* (uzun süreli) hatalar içindir. *Geçici* (kendiliğinden düzelen) hatalar için doğru desen bunun yerine üstel geri çekilmeyle yeniden denemedir — yanlış hata türü için yanlış stratejiyi kullanmak yaygın bir hatadır.

**Gerçek dünya benzetmesi:** Circuit breaker, ev elektrik sigortası gibidir — bir şey sürekli kısa devre yaptığında, kablolamanın kıvılcım çıkarmaya devam etmesine izin vermek yerine atar ve denemeyi durdurur.

---

### Cloud Build

**Tanım:** Kaynak koddan ya da yapılandırmadan Docker container imajları inşa eden ve bunları Artifact Registry'ye gönderen, tam yönetilen bir servis.

**Neden var:** Ekiplerin kendi CI build sunucularını sağlamak, yamalamak ve ölçeklemek zorunda kalmaması için.

**İlgili servisler:** Artifact Registry, Cloud Deploy, Continuous Integration, Delivery, and Deployment (CI/CD).

**Yaygın yanlış anlamalar:** Cloud Build'deki her build step'i, kendisi bir Docker container'dır — bu isteğe bağlı bir yapılandırma değil, sistemin işleyiş biçimidir. Adımlar veriyi başka bir paylaşılan durum üzerinden değil, `/workspace` dizini üzerinden paylaşır.

**Gerçek dünya benzetmesi:** Cloud Build, her istasyonun (build step) ayrı, kendi içine kapalı bir çalışma alanı olduğu ama her istasyonun parçaları iletmek için aynı konveyör bandını (`/workspace`) paylaştığı bir montaj hattıdır.

---

### Cloud CDN

**Tanım:** Gecikmeyi ve orijin yükünü azaltmak için içeriği kullanıcılara yakın kenar konumlarında önbelleğe alan, Google'ın içerik dağıtım ağı.

**Neden var:** Her isteği tek bir orijin konumundan sunmak, o konumdan uzaktaki kullanıcılar için gereksiz gecikme ekler.

**İlgili servisler:** Cloud Load Balancing, Cloud Storage, Cloud Run.

**Yaygın yanlış anlamalar:** Cloud CDN, ağ kenarında *web/statik içeriği* önbelleğe alır; bu, compute'una yakın bellekte *uygulama verisini* önbelleğe alan Memorystore'dan farklı bir önbellekleme katmanıdır.

**Gerçek dünya benzetmesi:** Cloud CDN, her siparişi tek bir merkezi fabrikadan göndermek yerine, popüler ürünleri müşterilere yakın yerel depolarda stoklayan bir depo zinciridir.

---

### Cloud Client Libraries

**Tanım:** Cloud API'lerini sarmalayan, kimlik doğrulamayı, yeniden denemeleri ve (çoğu zaman) gRPC çağrılarını senin yerine halleden, dile özgü kütüphaneler.

**Neden var:** Ham HTTP/JSON ya da gRPC API'lerini doğrudan çağırmak mümkündür ama sıkıcı ve hataya açıktır; client library'ler, kodundan Google Cloud'la konuşmanın önerilen yoludur.

**İlgili servisler:** Application Default Credentials, gRPC, Service Account.

**Yaygın yanlış anlamalar:** Client library'ler, geçici ağ hatalarını otomatik olarak geri çekilmeyle yeniden dener — bu mantığı genellikle sen yazmak zorunda kalmazsın.

**Gerçek dünya benzetmesi:** Bir Cloud Client Library, sadece senin için yerel dili konuşmakla kalmayıp evrakları da otomatik halleden ve kapı ilk açılışta cevap vermezse yeniden çalan bir tercüman gibidir.

---

### Cloud Code

**Tanım:** Bulut uygulamalarını editörünün içinden inşa etmeyi, dağıtmayı ve hata ayıklamayı kolaylaştıran bir dizi IDE eklentisi (VS Code, JetBrains, Cloud Shell Editor).

**Neden var:** Geliştiricilerin Secret Manager girdilerini yönetmek, Cloud API'lere göz atmak, Kubernetes ile çalışmak ya da Cloud Run'ı yerelde çalıştırmak için IDE'lerinden ayrılmasına gerek kalmasın diye.

**İlgili servisler:** Cloud Shell, Cloud Run, Secret Manager, Kubernetes.

**Yaygın yanlış anlamalar:** Cloud Code, IDE'nin yerine geçen bir araç değildir — zaten kullandığın mevcut IDE'lerle entegre olan bir eklenti setidir.

**Gerçek dünya benzetmesi:** Cloud Code, yeni bir araba almak yerine mevcut arabana kısayollar paneli eklemeye benzer.

---

### Cloud Deploy

**Tanım:** Uygulamaların teslimini, tanımlı bir hedef ortam dizisi (staging, üretim) boyunca Cloud Run ya da GKE'ye otomatikleştiren yönetilen bir servis.

**Neden var:** "Bu derlemeyi bir sonraki ortama dağıt"ı, geri alma özellikli, tekrarlanabilir, tek tıklık (ya da tamamen otomatik) bir sürece dönüştürmek için.

**İlgili servisler:** Cloud Build, Artifact Registry, Continuous Integration, Delivery, and Deployment (CI/CD), Binary Authorization.

**Yaygın yanlış anlamalar:** Cloud Deploy, "dağıtım sistemi" kavramının somut gerçekleştirimidir — sadece bir dağıtım tetikleyicisi değildir, dağıttığı şey hakkında güvenlik içgörüleri de gösterir.

**Gerçek dünya benzetmesi:** Cloud Deploy, doğrulanmış bir ürünü depo, hazırlık alanı ve son olarak mağaza rafı boyunca taşıyan, her aşamada geri çekebilme yeteneğine sahip bir sevkiyat koordinatörüdür.

---

### Cloud DNS

**Tanım:** Google'ın kendi DNS'iyle aynı altyapıda çalışan, yönetilen, programlanabilir DNS servisi.

**Neden var:** Google Cloud'da çalışan uygulamaların, dünyanın hostname'lerini çözebilmesi için güvenilir, düşük gecikmeli, yüksek kullanılabilir bir yola ihtiyacı vardır.

**İlgili servisler:** Cloud Load Balancing, VPC.

**Yaygın yanlış anlamalar:** Cloud DNS, Google'ın genel çözücüsüyle (8.8.8.8) aynı şey değildir — o, ücretsiz genel bir DNS *çözücü* servisidir; Cloud DNS ise kendi zone'larını yayınlamak için kullandığın yönetilen, *yetkili* DNS ürünüdür.

**Gerçek dünya benzetmesi:** Cloud DNS, işletmenin numarasını dünya çapında listeleyip anında güncellenebilir tutan bir telefon rehberi servisidir.

---

### Cloud Identity

**Tanım:** Kuruluşların, mutlaka bir Google Workspace müşterisi olmadan da kullanıcıları ve grupları merkezi olarak yönetmesini sağlayan Google'ın kimlik, erişim, uygulama ve uç nokta yönetim platformu.

**Neden var:** Bireysel Gmail hesaplarıyla başlamak bir süre işe yarar, ama biri ayrıldığında erişimi kaldırmanın merkezi bir yolunu sunmaz — Cloud Identity bunu düzeltir.

**İlgili servisler:** IAM, Principal, Organization Node.

**Yaygın yanlış anlamalar:** Bir Google grubu ya da Google Workspace hesabı gibi, bir Cloud Identity domain'i de kendi başına bir API isteğini doğrulayamaz — sadece Google hesaplarını daha kolay politika yönetimi için gruplar. Ayrıca bir Cloud Identity domain'i, Google Workspace uygulamalarına (Gmail, Docs, Drive) erişimi *olmayan* kuruluşlar içindir; Google Workspace hesapları o erişimi *içerir*.

**Gerçek dünya benzetmesi:** Cloud Identity, bir şirketin İK dizinidir — kuruluşa kimin ait olduğunu izler ve şirket her hizmeti kullanmasa bile, biri ayrıldığı an rozetini iptal eder.

---

### Cloud Load Balancing

**Tanım:** Bir uygulamanın birden çok örneğine trafiği yayan, tam dağıtık, yazılım tanımlı, yönetilen bir servis; Application Load Balancer (HTTP/HTTPS, 7. katman) ve Network Load Balancer'ı (TCP/UDP, 4. katman, proxy ve passthrough varyantlarıyla) kapsar.

**Neden var:** Örnek sayısı artıp azaldıkça, istemcilerin hangi arka ucun hizmet vereceğini bilmeden trafik gönderebileceği tek, kararlı bir yere ihtiyacı vardır.

**İlgili servisler:** Autoscaling, Managed Instance Group, VPC, Cloud CDN.

**Yaygın yanlış anlamalar:** HTTP(S) içerik tabanlı yönlendirme bir Application Load Balancer gerektirir, Network Load Balancer değil. Network Load Balancer'lar içinde, *proxy* varyantı bağlantıyı sonlandırır; *passthrough* varyantı sonlandırmaz ve istemcinin kaynak IP'sini korur — bunlar sık karıştırılır.

**Gerçek dünya benzetmesi:** Cloud Load Balancing, gelen misafirleri, misafirlerin kat planını bilmesine gerek kalmadan en hızlı hizmet verebilecek boş masaya oturtan bir restoran görevlisidir.

---

### Cloud Logging

**Tanım:** Google Cloud kaynaklarından logları otomatik olarak toplayan, depolama, arama, analiz ve uyarı desteğine sahip gerçek zamanlı bir log yönetim sistemi.

**Neden var:** Metrikler sana "ne kadar" der; loglar sana "neden" der — belirli bir hatayı ayıklamak için ayrıntılı izlere ihtiyacın vardır.

**İlgili servisler:** Cloud Monitoring, Error Reporting, Ops Agent, Structured Logging.

**Yaygın yanlış anlamalar:** GKE'de, container ve sistem logları varsayılan olarak **kalıcı değildir** (pod silindiğinde container logu kaybolur; cluster olayları bir saat sonra temizlenir) — kalıcılık için bunları Cloud Logging'e yönlendirmen gerekir.

**Gerçek dünya benzetmesi:** Cloud Logging, bir uçağın kara kutusudur — bir şeyler ters gittikten sonra, tam olarak ne olduğunu saniye saniye yeniden kurmak için kaydedilen ayrıntıyı çekersin.

---

### Cloud Monitoring

**Tanım:** Google Cloud'dan (ve diğer bulutlar/on-premises'ten) metrikleri, olayları ve metadata'yı toplayan, dashboard ve uyarı politikaları kurmanı sağlayan bir servis.

**Neden var:** Güvenilirlik, bir sorunu kullanıcılarından önce fark etmek demektir — Monitoring bunu mümkün kılan erken uyarı sistemidir.

**İlgili servisler:** Cloud Logging, Error Reporting, Four Golden Signals, Prometheus.

**Yaygın yanlış anlamalar:** Monitoring yalnızca Google Cloud'a özel değildir — açıkça tek bir bakış açısından çoklu bulut ve on-premises kaynakları destekler.

**Gerçek dünya benzetmesi:** Cloud Monitoring, bir hastanenin yoğun bakım ünitesindeki monitör bankasıdır — hayati bulgular sürekli izlenir ve durum kritik hale gelmeden bir alarm çalar.

---

### Cloud Profiler

**Tanım:** Üretim uygulama örneklerinden CPU ve bellek tahsisi verisini sürekli toplayan ve bunu kaynak koduna atfeden istatistiksel, düşük ek yüklü bir profil çıkarıcı.

**Neden var:** Test ortamları nadiren gerçek üretim yükünü, eşzamanlılığını ve veri hacmini yeniden yaratır — performansı gerçekten önemli olduğu yerde gözlemlemen gerekir.

**İlgili servisler:** Cloud Trace, Cloud Monitoring.

**Yaygın yanlış anlamalar:** Cloud Profiler, Cloud Trace ile kolayca karıştırılır. Trace "hangi istek adımı yavaştı" sorusuna cevap verir (istek/zaman ekseni); Profiler "hangi fonksiyon CPU/bellek tüketti" sorusuna cevap verir (kod/kaynak ekseni).

**Gerçek dünya benzetmesi:** Cloud Profiler, gün boyunca her makinenin rastgele anlık fotoğraflarını çeken bir fabrika gözlemcisi gibidir — hiçbir makineyi sürekli izlemez ama hiçbir şeyi yavaşlatmadan zamanın nerede harcandığına dair istatistiksel olarak doğru bir resim oluşturur.

---

### Cloud Run

**Tanım:** Web isteklerine ya da Pub/Sub olaylarına yanıt olarak durumsuz container'ları çalıştıran, sıfıra kadar otomatik ölçeklenen, tam yönetilen, serverless bir platform.

**Neden var:** Geliştiricilerin, herhangi bir sunucu sağlamadan, yapılandırmadan ya da yönetmeden konteynerize kod dağıtabilmesi için.

**İlgili servisler:** Cloud Run Functions, Cloud Run Jobs, Container, Artifact Registry, GKE.

**Yaygın yanlış anlamalar:** Cloud Run, HTTP(S)/gRPC dışındaki TCP protokollerini desteklemez — bunun için Compute Engine ya da GKE gerekir. Ayrıca bir "mini VM" değildir; Kubernetes tabanlı açık bir API ve çalışma zamanı olan Knative üzerine kuruludur.

**Gerçek dünya benzetmesi:** Cloud Run, oda servisi tam olan bir otel odasıdır — sen sadece valizini (kodunu) getirirsin; kat hizmetleri, faturalar ve çıkış işlemleri tamamen otele aittir ve odada olmadığın süre için ödeme yapmazsın.

---

### Cloud Run Functions

**Tanım:** Arka planda Cloud Run servisleri olarak dağıtılan, Cloud Storage/Pub/Sub olaylarıyla (asenkron) ya da HTTP çağrılarıyla (senkron) tetiklenen, hafif, olay güdümlü, tek amaçlı fonksiyonlar.

**Neden var:** Yüklenen bir resmi yeniden boyutlandırmak gibi küçük, tek amaçlı görevler için sürekli açık bir servis çalıştırmak israftır — sadece tetiklendiğinde çalışan bir fonksiyon daha uygundur.

**İlgili servisler:** Cloud Run, Pub/Sub, Eventarc, Cloud Storage.

**Yaygın yanlış anlamalar:** Cloud Run Functions, Cloud Run'dan ayrı bir ürün değildir — fonksiyonlar farklı bir paketleme modeliyle Cloud Run servisleri *olarak* dağıtılır.

**Gerçek dünya benzetmesi:** Bir Cloud Run Function, tam zamanlı bir masa tutmak yerine sadece belirli bir görev için çağrıldığında ortaya çıkan nöbetçi bir uzman gibidir.

---

### Cloud Run Jobs

**Tanım:** HTTP isteklerini dinlemek yerine tamamlanana kadar çalışıp çıkan, tek seferlik ya da zamanlanmış görevler için bir Cloud Run çalıştırma modu.

**Neden var:** Her iş yükü bir istek/yanıt servisi değildir — toplu işleme bunun yerine "bir kez çalış, bitir, çık" modeline ihtiyaç duyar.

**İlgili servisler:** Cloud Run, Cloud Scheduler.

**Yaygın yanlış anlamalar:** Bir Cloud Run job'ı bir portu **dinlemez** ve HTTP trafiği kabul etmez — bir Cloud Run *servisi*nden temel farkı budur. Bir job içindeki görevler paralel çalışabilir ve başarısız olan görevler otomatik olarak yeniden denenebilir.

**Gerçek dünya benzetmesi:** Cloud Run servisi açık ve müşteri bekleyen bir dükkânsa, Cloud Run job'ı tek bir rotayı tamamlayıp işi bitince paydos eden bir kurye gibidir.

---

### Cloud Scheduler

**Tanım:** Tek bir dashboard'dan yönetilen, tamamen yönetilen, kurumsal düzeyde bir cron job scheduler; tekrarlayan bir programa göre bir Pub/Sub topic'ini, bir App Engine uygulamasını ya da herkese açık bir HTTP endpoint'ini tetikleyebilir.

**Neden var:** Tekrarlayan job'lar (gecelik batch işleri, saatlik senkronizasyonlar) garantili execution ve başarısızlıkta otomatik retry'a ihtiyaç duyar — bir VM üzerinde kendi cron daemon'ını çalıştırmak, onun için patching, uptime ve monitoring'i de sahiplenmen anlamına gelir.

**İlgili servisler:** Cloud Tasks, Pub/Sub, App Engine.

**Yaygın yanlış anlamalar:** Cloud Scheduler'ın varsayılan time zone'u UTC'dir, ve UTC'de kalmak önerilir — daylight saving time uygulayan time zone'lar, saatler ileri alındığında bir job'ın atlanmasına ya da saatler geri alındığında iki kez çalışmasına neden olabilir.

**Gerçek dünya benzetmesi:** Cloud Scheduler, bir binanın otomatik alarm sistemi saati gibidir — etraftaki kimin olduğundan bağımsız olarak tam olarak zamanlanmış saatlerde çalar, ve her şube için tek bir standart saat (UTC) kullanmak, her lokasyonun daylight saving'i farklı şekilde uygulamasının yarattığı kaosu önler.

---

### Cloud Shell

**Tanım:** Google Cloud SDK'sı önceden kurulu ve önceden kimlik doğrulanmış, 5 GB kalıcı home dizinli, ücretsiz, tarayıcı tabanlı, geçici bir Debian VM'i.

**Neden var:** Geliştiricilere, herhangi bir tarayıcıdan, sıfır kurulumla, Google Cloud'a anında komut satırı erişimi vermek için.

**İlgili servisler:** gcloud CLI, Cloud Workstations, Cloud Code.

**Yaygın yanlış anlamalar:** Cloud Shell örnekleri bir saatlik hareketsizlikten sonra sonlanır — ama 5 GB'lık persistent disk hayatta kalır, yani dosyaların kaybolmaz, sadece çalışan makine geri dönüştürülür.

**Gerçek dünya benzetmesi:** Cloud Shell, bir otelin iş merkezindeki bilgisayar gibidir — her zaman hazır, önceden yapılandırılmış ve ücretsizdir, ama uzaklaştığında sıfırlanır; sadece kişisel dolabına (persistent disk) kaydettiğini saklar.

---

### Cloud SQL

**Tanım:** MySQL, PostgreSQL ve SQL Server'ı destekleyen, Google'ın replikasyon, yük devretme, yedekleme ve yamalamayı yönettiği, tam yönetilen ilişkisel veritabanı servisi.

**Neden var:** Üretimde bir ilişkisel veritabanını kendin çalıştırmak; replikasyon, yük devretme ve yedeklemeyi yönetmek demektir — Cloud SQL bu operasyonel yükü kaldırır.

**İlgili servisler:** AlloyDB, Spanner, OLTP and OLAP.

**Yaygın yanlış anlamalar:** Cloud SQL tek bölgeli, dikey ölçeklenen bir ilişkisel veritabanıdır — Spanner'ın yatay, küresel ölçeğini sunmaz. VPC'sinin dışından erişim, IP izin listelerini ve SSL sertifikalarını elle yönetmekten kaçınan **Cloud SQL Auth Proxy** ile en iyi şekilde yapılır.

**Gerçek dünya benzetmesi:** Cloud SQL, bakımı dahil güvenilir bir kiralık araba gibidir — hâlâ sen sürersin (sorguları yazarsın) ama motora (replikasyon, yamalama, yedekleme) asla dokunmazsın.

---

### Cloud Storage

**Tanım:** Görüntü, video, yedek ve arşiv gibi yapılandırılmamış veriler için Google Cloud'un dayanıklı, yüksek erişilebilir nesne depolama servisi.

**Neden var:** Büyük, yapılandırılmamış dosyalar bir veritabanında satır olarak durmamalı; nesne depolama bunları her ölçekte verimli işler, web kullanımına uygun URL-dostu bir ad alanıyla.

**İlgili servisler:** Bucket, Cloud Storage Classes, Cloud CDN, BigQuery.

**Yaygın yanlış anlamalar:** Cloud Storage bir key-value nesne deposudur, sorgulanabilir bir veritabanı değil — üzerinde JOIN, indeks ya da SQL yoktur. Nesneler değişmezdir; değişiklikler yerinde düzenleme değil, yeni versiyonlar oluşturur.

**Gerçek dünya benzetmesi:** Cloud Storage, öğeleri tam etiketiyle (nesne adı) geri aldığın bir kendi kendine depolama deposudur — "bana mavi olan her şeyi bul" diye sormadığın bir yerdir.

---

### Cloud Storage Classes

**Tanım:** Maliyet ve beklenen erişim sıklığı açısından farklılaşan dört depolama katmanı — Standard, Nearline, Coldline ve Archive (sürekli erişimden yılda birden aza kadar).

**Neden var:** Nadiren erişilen veriyi "sıcak" fiyatlarla saklamak para israfıdır; sınıflar, daha az eriştiğin veri için daha az ödemeni sağlar.

**İlgili servisler:** Cloud Storage, Bucket.

**Yaygın yanlış anlamalar:** Archive sınıfının depolama maliyeti en düşüktür ama erişim maliyeti en yüksektir ve 365 günlük minimum saklama süresi vardır — saklamak ucuzdur, geri almak ucuz değildir. Autoclass, nesneleri gerçek erişim desenlerine göre sınıflar arasında taşımayı otomatikleştirir.

**Gerçek dünya benzetmesi:** Bu sınıflar, kendi kendine depolama tesisinin fiyatlandırma katmanları gibidir: her gün ziyaret ettiğin bir birim, yılda bir ziyaret ettiğin uzak bir birimden aylık daha pahalıdır — ama uzak birimden bir şey çıkarmak ekstra ücrete tabidir.

---

### Cloud Tasks

**Tanım:** Her biri belirli bir HTTP servisine dispatch edilen, çok sayıda dağıtık task'ın execution'ını, dispatch'ini ve teslimatını yapılandırılabilir rate limit'ler, retry'lar ve zamanlanmış dispatch saatleriyle yöneten bir servis.

**Neden var:** Bir uygulamanın genelde ana request'i bloklamadan, belirli, bilinen bir yavaş işi (bir rapor üretmek, bir third-party API'yi çağırmak) offload etmesi gerekir, ama bunu hangi servisin ve ne zaman ele alacağı üzerinde doğrudan kontrolü elinde tutarak yapmak ister.

**İlgili servisler:** Pub/Sub, Cloud Scheduler, Eventarc.

**Yaygın yanlış anlamalar:** Cloud Tasks ve Pub/Sub kavramsal olarak benzerdir (ikisi de asenkron message passing yapar) ama birbirinin yerine geçmez: Cloud Tasks **explicit invocation** kullanır — creator, execution ve destination üzerinde tam kontrolü elinde tutar — Pub/Sub ise **implicit invocation** kullanır, burada bir mesaj publish etmek, mevcut olan hangi subscriber varsa onu tetikler, kimin alacağı üzerinde hiçbir kontrol yoktur.

**Gerçek dünya benzetmesi:** Cloud Tasks, belirli bir mektubu belirli bir kuryeye, belirli teslimat talimatları ve bir teslimat saatiyle vermek gibidir; Pub/Sub ise bir panoya herkese açık bir duyuru asmak gibidir — şu anda ilgilenen herkes bu duyuru üzerinde harekete geçebilir, ve kimin ilgileneceği üzerinde hiçbir söz hakkın yoktur.

---

### Cloud Trace

**Tanım:** İstek başına gecikme verisini toplayan ve bir isteğin uygulamanda nasıl yayıldığını gösteren dağıtık bir izleme sistemi.

**Neden var:** Tek bir istek birden çok servise, veritabanına ve API'ye dokunabilir — zamanın gerçekte nereye gittiğini görmek için tek bir zaman çizelgesi görünümüne ihtiyacın vardır.

**İlgili servisler:** Cloud Profiler, Cloud Monitoring.

**Yaygın yanlış anlamalar:** Cloud Run'daki otomatik izleme yalnızca gelen/giden HTTP isteğini yakalar — servisin **içindeki** gecikmeyi göstermez. İç span'leri görmek enstrümantasyon gerektirir (önerilen yaklaşım OpenTelemetry + Cloud Trace Exporter'dır).

**Gerçek dünya benzetmesi:** Bir trace, bir yemek siparişinin verilmesinden servis edilmesine kadar olan tüm yolculuğudur; her span o yolculuğun bir bacağıdır — mutfak hazırlığı, pişirme, servis — ayrı ayrı zamanlanır.

---

### Cloud Workstations

**Tanım:** VPC'nin içindeki geçici Compute Engine VM'lerinde çalışan, tekrar üretilebilir bir workstation yapılandırmasıyla tanımlanan, tam yönetilen, güvenli, bulut tabanlı geliştirme ortamları.

**Neden var:** "Bende çalışıyordu" sorunlarını ortadan kaldırmak ve BT'nin bir ekibin tamamı için tutarlı geliştirme ortamlarını sağlamasına, ölçeklemesine ve güvenceye almasına izin vermek için.

**İlgili servisler:** Cloud Shell, Compute Engine, VPC.

**Yaygın yanlış anlamalar:** Cloud Workstations, Cloud Shell ile aynı şey değildir — Cloud Shell hızlı, geçici, tek kullanıcılı bir yönetim kabuğudur; Cloud Workstations standartlaştırılmış, kalıcı, ekip çapında bir geliştirme ortamıdır.

**Gerçek dünya benzetmesi:** Cloud Shell bir otelin iş merkeziyse, Cloud Workstations her çalışana verilen, şirket tarafından sağlanan, aynı şekilde yapılandırılmış bir dizüstü bilgisayardır.

---

### CloudEvent Function

**Tanım:** Event-driven Cloud Run functions için güncel implementasyon stili; CloudEvents endüstri standardı spesifikasyonu üzerine kuruludur ve Functions Framework'e kayıtlıdır; tüm Cloud Run functions dil runtime'larında (ve Cloud Run functions 1st gen tarafından .NET, Ruby ve PHP için) desteklenir.

**Neden var:** Event-driven function'lara, daha eski, generation'a ve dile özgü Background function modelinin yerine, event verisini almanın standartlaştırılmış, taşınabilir bir yolunu vermek için.

**İlgili servisler:** Cloud Run Functions, Background Function, Functions Framework, CloudEvents, Eventarc.

**Yaygın yanlış anlamalar:** Bir CloudEvent function, bir HTTP function ile aynı şey değildir — doğrudan bir HTTP isteği yerine bir event trigger tarafından tetiklenen, iki event-driven function stilinden biridir (güncel olanı).

**Gerçek dünya benzetmesi:** Bir CloudEvent function, evrensel bir paket etiketleme standardında eğitilmiş, taşıyıcıya özgü talimatlara ihtiyaç duymadan herhangi bir taşıyıcıdan paket alabilen bir kurye gibidir.

---

### CloudEvents

**Tanım:** Event verisini tanımlamak için ortak bir metadata formatı sağlayan bir CNCF standart spesifikasyonu; Eventarc, kaynaktan bağımsız olarak event'leri tutarlı biçimde teslim etmek için bunu kullanır.

**Neden var:** Event publisher'ları tarihsel olarak her biri kendi özel formatını kullandı, bu da consumer'ları kaynağa özel parsing kodu yazmaya zorluyordu — paylaşılan bir format, aynı event-handling mantığının, event nereden gelirse gelsin çalışmasını sağlar.

**İlgili servisler:** Eventarc, Event, Event-Driven Architecture.

**Yaygın yanlış anlamalar:** CloudEvents Google'a özgü değildir — birçok dil için SDK'ları olan (Python, JavaScript, Java, Go, C#, Ruby, PHP) bir CNCF (Cloud Native Computing Foundation) standardıdır, ve değeri özellikle herhangi bir tek kaynağa ya da vendor'a **bağlı olmamasında** yatar.

**Gerçek dünya benzetmesi:** CloudEvents, her taşıyıcının kullandığı standartlaştırılmış bir kargo etiketi formatı gibidir — paketi hangi şirket teslim ederse etsin, etiket her zaman aynı yerde aynı alanlara sahiptir, bu yüzden teslim alan depo her taşıyıcı için farklı bir kabul süreci gerektirmez.

---

### Compute Engine

**Tanım:** Google Cloud'un IaaS teklifi — geleneksel bir veri merkezinde çalıştıracağın sunucuları taklit eden, işletim sistemi, disk ve ağ üzerinde tam kontrol sunan VM'ler.

**Neden var:** Maksimum kontrol, özel işletim sistemi/donanım ya da lift-and-shift göçü gerektiren iş yükleri için geleneksel sunucu deneyimini bulutta yeniden yaratmak için.

**İlgili servisler:** Managed Instance Group, Autoscaling, VPC, GKE.

**Yaygın yanlış anlamalar:** Daha fazla kontrol "her zaman en iyi seçim" anlamına gelmez — hedef minimum operasyonel efor ise, Compute Engine nadiren doğru cevaptır. Özel makine tipleri, sadece önceden tanımlı şekiller yerine vCPU/belleği ince ayar yapmanı sağlar.

**Gerçek dünya benzetmesi:** Compute Engine, boş bir daire kiralamaktır — bina (donanım, elektrik, soğutma) senin için inşa edilmiş ve bakımı yapılmıştır, ama içindeki her şey (işletim sistemi, yazılım, mobilya) tamamen senin sorumluluğundadır.

---

### Consistency (Strong vs. Eventual)

**Tanım:** Güçlü tutarlılık, bir yazmadan hemen sonraki bir okumanın her zaman ve her yerde en güncel değeri döndürmesi demektir. Nihai tutarlılık, bazı okuyucuların değişiklik yayılana kadar kısa süreliğine eski veriyi görebilmesi demektir.

**Neden var:** Dağıtık veritabanları gecikme, kullanılabilirlik ve tutarlılık arasında ödünleşim yapmak zorundadır; bir servisin hangi modeli sunduğunu bilmek, doğruluk ihtiyaçlarına uyup uymadığını belirler.

**İlgili servisler:** Spanner, Firestore, Bigtable.

**Yaygın yanlış anlamalar:** "Güçlü tutarlılık + yatay ölçek + ilişkisel + küresel" birlikte geçiyorsa neredeyse kesinlikle Spanner'a işaret eder — çok az servis dördünü aynı anda sunar.

**Gerçek dünya benzetmesi:** Güçlü tutarlılık, herkesin doğrudan okuduğu tek bir paylaşılan yazı tahtası gibidir. Nihai tutarlılık, o tahtanın birer birer güncellenen birkaç fotokopisi gibidir — bir an için bazı kopyalar güncel değildir.

---

### Container

**Tanım:** Donanım sanallaştırması yerine işletim sistemi seviyesinde sanallaştırma (process namespace'leri ve kaynak limitleri) üzerine kurulu, bir uygulama ve bağımlılıkları için hafif, izole bir çalıştırma ortamı.

**Neden var:** VM'ler yavaş açılır ve kaynak açısından ağırdır çünkü her biri tam bir OS kopyası taşır; container'lar bunun yerine OS seviyesinde izole eder ve saniyenin bir kısmında başlar.

**İlgili servisler:** Container Image, Kubernetes, Cloud Run, GKE.

**Yaygın yanlış anlamalar:** Bir container **"mini bir VM" değildir.** VM donanımı sanallaştırır ve kendi OS kopyasını çalıştırır; container işletim sistemini sanallaştırır ve process izolasyonu ve namespace'ler aracılığıyla host kernel'ini paylaşır.

**Gerçek dünya benzetmesi:** VM, her kiracının kendi evini sıfırdan, kendi tesisatıyla inşa etmesi gibidir. Container, ortak altyapıyı (kernel) paylaşan ama her dairenin kilitli ve bağımsız kaldığı bir apartman gibidir.

---

### Container Image

**Tanım:** Bir uygulamanın binary'sinin ve çalışması için gereken her şeyin eksiksiz, kendi içine kapalı paketi; dev, test ve üretimde aynı davranışı garanti eder.

**Neden var:** Ortam farkı ("bende çalışıyordu") dağıtım hatalarının klasik nedenidir — gerekli her şeyi paketlemek bu hata sınıfını ortadan kaldırır.

**İlgili servisler:** Container, Cloud Build, Artifact Registry.

**Yaygın yanlış anlamalar:** Staging'de test edilen imaj ile üretime dağıtılan imaj *tam olarak aynı* imaj olmalıdır — aralarında yeniden derlenmemeli ya da yeniden paketlenmemelidir.

**Gerçek dünya benzetmesi:** Bir Docker imajı bir tariftir; çalışan bir container o tariften pişmiş yemektir. Aynı tarif, nerede pişirirsen pişir aynı yemeği üretir.

---

### Continuous Integration, Delivery, and Deployment (CI/CD)

**Tanım:** Üç aşamalı otomatik bir hat: **Continuous Integration (CI)**, bir feature branch'e her commit'te kodu derler ve test eder; **Continuous Delivery**, main branch'te testler geçtikten sonra bir release-candidate yapıtı inşa eder ve üretime geçmeden önce **manuel onay** bekler; **Continuous Deployment** aynıdır ama **manuel onay yoktur** — geçen bir derleme otomatik olarak dağıtılır.

**Neden var:** Manuel yayınlar ekip boyutuyla ya da değişiklik hacmiyle ölçeklenmez — otomasyon, yayınları yavaş, riskli, büyük patlamalı olaylardan küçük, sık, düşük riskli değişikliklere dönüştürür.

**İlgili servisler:** Cloud Build, Artifact Registry, Cloud Deploy, Progressive Delivery.

**Yaygın yanlış anlamalar:** Delivery ve Deployment bu alanda en çok karıştırılan çifttir. Delivery, üretime hazır bir yapıt üretir ama **bir insan** ne zaman yayınlanacağına karar verir. Deployment, testler geçer geçmez otomatik olarak yayınlar — tek fark bu manuel kapıdır.

**Gerçek dünya benzetmesi:** CI, hattan çıkan her parça için bir kalite kontrol noktasıdır. Delivery, bitmiş ürünü paketleyip bir yöneticinin onayını beklemektir. Deployment, denetimden geçtiği an ürünü otomatik olarak göndermektir.

---

### Deny Policy

**Tanım:** Belirli principal'ların, kendilerine verilen rollerden bağımsız olarak belirli izinleri kullanmasını açıkça engelleyen bir IAM kuralı.

**Neden var:** Bazen, hiyerarşinin başka bir yerinde verilen geniş bir rolle atlatılamayacak zorunlu bir geçersiz kılmaya ihtiyacın olur.

**İlgili servisler:** IAM, Role, Principal.

**Yaygın yanlış anlamalar:** IAM her zaman deny politikalarını allow politikalarından **önce** değerlendirir — bir deny varsa, bir allow politikası da aynı izni verse dahi deny kazanır.

**Gerçek dünya benzetmesi:** Deny politikası, bir kulübün "ömür boyu yasaklı" listesidir — hiçbir VIP kart bunu geçersiz kılamaz.

---

### Deployment (Kubernetes)

**Tanım:** Bir dizi özdeş Pod replikasını yöneten, altta yatan node'lar çökse bile istenen sayının çalışır kalmasını sağlayan bir Kubernetes kaynağı; durumsuz (stateless) bileşenler için kullanılır.

**Neden var:** Pod'lar gelip gider ve IP'leri sabit değildir; bir Deployment, her zaman tutarlı sayıda sağlıklı replikanın hazır olmasını sağlar.

**İlgili servisler:** Pod, Service, GKE, Kubernetes.

**Yaygın yanlış anlamalar:** Bir Deployment, herhangi bir replikanın herhangi bir isteğe cevap verebildiği **durumsuz** bileşenler içindir. Kalıcı depolamaya ve kararlı bir ağ kimliğine ihtiyaç duyan uygulamalar (bir veritabanı gibi) bunun yerine bir **StatefulSet** gerektirir — bunları karıştırmak klasik bir sınav tuzağıdır.

**Gerçek dünya benzetmesi:** Bir Deployment, her zaman tam olarak 10 temsilciyle çalışan bir çağrı merkezi gibidir — biri hasta olup eve giderse, yerine biri çağrılır ve arayanlar hangi temsilcinin cevap verdiğini umursamaz.

---

### Document AI

**Tanım:** Yapılandırılmamış belge içeriğini (taranmış faturalar, sözleşmeler, formlar) yapılandırılmış, sorgulanabilir veriye dönüştüren bir servis.

**Neden var:** Tutarsız düzen ve yazı tiplerindeki binlerce belge, alanları (tarih, tutar, isim) tutarlı biçimde çıkarılana kadar analiz için kullanılamaz.

**İlgili servisler:** Vision AI, Natural Language AI.

**Yaygın yanlış anlamalar:** Document AI, OCR'den (metin okuma) fazlasını yapar — sadece ham metin değil, anlamı eklenmiş *yapılandırılmış alanlar* çıkarır.

**Gerçek dünya benzetmesi:** Document AI, farklı biçimlerdeki bir yığın faturayı okuyup her seferinde tutarı, tarihi ve satıcıyı aynı elektronik tablo sütunlarına giren titiz bir görevli gibidir.

---

### Enterprise Service Bus (ESB)

**Tanım:** Service-Oriented Architecture (SOA) içinde kullanılan, servisler arası bağlantı, güvenlik ve mesaj routing/dönüşümünü yöneten merkezi bir messaging middleware bileşeni.

**Neden var:** SOA'nın, her servisin — hatta organizasyon dışındaki uygulamaların bile — protokol ve veri dönüşümlerini birbirlerine karşı elle kodlamadan entegre olabilmesine ihtiyacı vardı; ESB bu işi merkezileştirir.

**İlgili servisler:** Service-Oriented Architecture (SOA), Microservices.

**Yaygın yanlış anlamalar:** ESB entegrasyon karmaşıklığını ortadan kaldırmaz, sadece taşır — SOA, tekil servislerdeki karmaşıklığı azalttı, ama bu karmaşıklık genelde tek bir merkezi takımın sahip olduğu ESB entegrasyon işi olarak yeniden ortaya çıktı ve her uygulama genelinde değişiklik göndermenin darboğazı haline geldi.

**Gerçek dünya benzetmesi:** Bir ESB, büyük bir ofisteki her telefon çağrısının üzerinden geçmesi gereken, merkezi olarak yönetilen tek bir santral gibidir — teoride verimlidir, ama her kablolama değişikliği santral operatöründen geçmeyi gerektirir ve oradaki herhangi bir hata herkes için çağrıları düşürebilir.

---

### Error Reporting

**Tanım:** Çalışan uygulamalardaki hataları, açık API raporları ya da loglardan stack trace desen çıkarımı kullanarak otomatik olarak tespit eden, gruplandıran ve sayan bir servis.

**Neden var:** "Hangi hata en sık oluyor ve yeni mi" sorusunu binlerce ham log satırı arasında elle çözmek yavaştır — Error Reporting bu değerlendirmeyi otomatikleştirir.

**İlgili servisler:** Cloud Logging, Cloud Monitoring.

**Yaygın yanlış anlamalar:** Error Reporting'i etkinleştirmek ortama göre değişir: Cloud Run'da otomatik, GKE'de `cloud-platform` erişim kapsamı gerektirir, Compute Engine VM'inin servis hesabında **Error Reporting Writer** IAM rolü gerektirir.

**Gerçek dünya benzetmesi:** Error Reporting, yüzlerce şikâyeti her bir bilet olarak tek tek göstermek yerine, onları gerçekte kaç kök nedenin yönlendirdiğine göre bir avuç gruba otomatik olarak ayıran bir müşteri şikâyet masasıdır.

---

### Event

**Tanım:** Gerçekleşmiş bir şeyin kaydı (bir login, sepete eklenen bir ürün); immutable bir gerçek olarak ele alınır, hiç consume edilmeden üretilebilir, ve farklı servisler tarafından paralel olarak birden fazla kez persist edilip consume edilebilir.

**Neden var:** Request/response çağrıları anında bir yanıt bekler ve işlendikten sonra ortadan kalkar; uygulamaların ayrıca, kimsenin hemen tepki vermeye hazır olmasına bağlı olmayan, "ne oldu"nun kalıcı ve replayable bir kaydına da ihtiyacı vardır.

**İlgili servisler:** Event-Driven Architecture, Microservices.

**Yaygın yanlış anlamalar:** Şu anda sıfır consumer'ı olan bir event otomatik olarak bir bug değildir — birçok producer, event'lerini bir şeyin consume edip etmediğini bilmez, bilmesine de gerek yoktur. Event'ler ayrıca sonradan düzenlenmemeli ya da silinmemelidir; bir düzeltme, eskisini değiştirmek yerine yeni bir event olarak ifade edilmelidir.

**Gerçek dünya benzetmesi:** Bir event, bir günlük (diary) kaydıdır — bir kez yazıldıktan sonra, o anda ne olduğunun kalıcı bir kaydıdır. Dünkü kaydı, sonradan öğrendiklerini yansıtacak şekilde geri dönüp düzenlemezsin; bunun yerine yeni bir kayıt yazarsın.

---

### Event-Driven Architecture

**Tanım:** Servislerin birbirini doğrudan çağırmak yerine, bir event intermediary üzerinden event üreterek ve consume ederek iletişim kurduğu bir mimari desen.

**Neden var:** Microservices arasındaki point-to-point çağrılar, her servisin bağımlı olduğu her downstream servise nasıl ulaşacağını bilmesini zorunlu kılar, bu da coupling yaratır ve servis sayısı arttıkça yönetilemez bir iletişim "örümcek ağına" dönüşebilir.

**İlgili servisler:** Event, Microservices, Enterprise Service Bus (ESB).

**Yaygın yanlış anlamalar:** Bir event intermediary, bir Enterprise Service Bus (ESB) ile aynı şey değildir — ESB, her entegrasyon değişikliğinin kendisinden ve onu yöneten takımdan geçmesi gerektiği için bir darboğaza dönüşen, merkezi, SOA döneminden kalma bir routing/transformation katmanıydı; event intermediary ise özellikle producer'ları consumer'lardan (hiçbiri diğeri hakkında bir şey bilmek zorunda değildir, yalnızca event'in formatını bilirler) bu tür merkezi bir darboğazı yeniden yaratmadan ayırmak için var olur. Ayrıca, event-driven (asenkron) işlem, senkron request/response zincirlerinden yalnızca "farklı" değil, daha resilient'tır — event-driven bir sistemde çöken bir consumer sadece geride kalıp replay/redelivery ile yakalanabilirken, senkron bir zincirde çöken bir servis, çağrı yığınında yukarı doğru hataları cascade edebilir.

**Gerçek dünya benzetmesi:** Senkron request/response çağrıları, bir dizi domino taşı gibidir — biri düşerse (başarısız olursa), arkasındaki her şey de düşer. Event-driven architecture ise daha çok bir posta kutuları setine benzer: gönderen bir mektubu bırakır ve yoluna devam eder; alıcı ortada değilse, mektup sadece geri gelene kadar bekler, tüm posta sistemi durmak yerine.

---

### Eventarc

**Tanım:** Birçok GCP ve third-party kaynaktan gelen event'leri, kural tabanlı event trigger'ları kullanarak target'lara yönlendiren, standart CloudEvents formatında teslim eden, Google Cloud'un tamamen yönetilen eventing sistemi.

**Neden var:** Mümkün olan her kaynak için event ingestion'ı elle kurmak (Cloud Audit Logs'u parse etmek, topic'leri/subscription'ları yönetmek, formatları normalize etmek) tekrarlayan, hataya açık bir iştir ve Eventarc bunu otomatikleştirir.

**İlgili servisler:** Pub/Sub, CloudEvents, Event-Driven Architecture.

**Yaygın yanlış anlamalar:** Eventarc bir Pub/Sub rakibi değildir — güvenilirlik ve observability için transport katmanı olarak Pub/Sub'ı kullanır, ama altındaki topic'leri ve subscription'ları otomatik olarak yönetir, bu yüzden uygulama yalnızca Eventarc'ın gönderdiği HTTP isteklerini kabul etmesi gerekir. Doğrudan event desteği olmayan kaynaklar için Eventarc, özel polling kodu gerektirmek yerine Cloud Audit Logs kayıtlarından event üretebilir.

**Gerçek dünya benzetmesi:** Eventarc, her kaynağın yerel dilini zaten konuşan ve sana vermeden önce her şeyi tek bir standart formata çeviren evrensel bir event dispatcher'ı gibidir — düzinelerce farklı sistemi dinlemek için yalnızca tek bir dil öğrenmen yeterlidir.

---

### Firebase Authentication

**Tanım:** Parolaları, telefon numaralarını ve federe kimlik sağlayıcılarını (Google, Apple, GitHub) destekleyen, hazır UI bileşenleriyle gelen, mobil ve web uygulamaları için hazır (drop-in) bir kimlik doğrulama servisi.

**Neden var:** Kendi giriş, parola saklama ve hesap kurtarma akışını yazmak, doğru ve güvenli yapılması aldatıcı derecede zordur — Firebase Auth bunu senin için halleder.

**İlgili servisler:** Identity Platform, OAuth 2.0.

**Yaygın yanlış anlamalar:** Firebase Authentication sık sık Identity Platform ile karıştırılır. Firebase Authentication mobil/web geliştiricileri hedefler; **Identity Platform aynı temel üzerine kurumsal özellikler ekler** (SAML, OpenID Connect, MFA, IAP entegrasyonu).

**Gerçek dünya benzetmesi:** Firebase Authentication, küçük bir dükkân için hazır bir ön kapı kilit sistemidir; Identity Platform, kurumsal bir bina için rozet okuyucular ve güvenlik kameralarıyla yükseltilmiş aynı sistemdir.

---

### Firestore

**Tanım:** Hiyerarşik bir veri modeliyle (koleksiyonlardaki dokümanlar), gerçek zamanlı senkronizasyon ve çevrimdışı desteğiyle, serverless, yatay ölçeklenen bir NoSQL doküman veritabanı.

**Neden var:** Mobil ve web uygulamalarının, çevrimdışıyken bile istemcilere gerçek zamanlı senkronize olan esnek, hızlı değişen, hiyerarşik veriye ihtiyacı vardır — katı bir tablo şeması bu şekle uymaz.

**İlgili servisler:** Cloud Storage, Bigtable, Cloud SQL.

**Yaygın yanlış anlamalar:** İkisi de "NoSQL" olduğu için Bigtable ile sık karıştırılır. Firestore doküman tabanlıdır, gerçek zamanlı senkronizasyon ve çevrimdışı destek gerektiren mobil/web uygulamaları için idealdir; Bigtable tek-anahtarlıdır ve sub-10ms gecikmede milyarlarca satır için inşa edilmiştir.

**Gerçek dünya benzetmesi:** Firestore, biri yazdığı an herkesin kopyasında otomatik güncellenen, hâlâ çevrimdışıyken not almana izin veren ve sonra senkronize eden paylaşılan bir defter gibidir.

---

### Folder

**Tanım:** Projeleri (ve diğer klasörleri) gruplamak için kullanılan, ihtiyacın olan ayrıntı düzeyinde politika atamanı sağlayan kaynak hiyerarşisinin üçüncü katmanı.

**Neden var:** Aynı politikayı iki ilişkili projeye ayrı ayrı uygulamak sıkıcı ve hataya açıktır; bir klasör bunu ikisine bir kerede uygulamanı sağlar.

**İlgili servisler:** Resource Hierarchy, Organization Node, Project, IAM.

**Yaygın yanlış anlamalar:** Klasörler var olabilmek için bir organizasyon düğümüne ihtiyaç duyar — biri olmadan klasör kullanamazsın. Ayrıca özel IAM rolleri klasör düzeyine **uygulanamaz** (sadece proje ya da organizasyon).

**Gerçek dünya benzetmesi:** Bir klasör, bir şirket içindeki bir departmandır — departman düzeyinde belirlenen politikalar içindeki her ekibe otomatik olarak uygulanır.

---

### Four Golden Signals

**Tanım:** Her servis dashboard'unun izlemesi gereken dört temel metrik: gecikme, trafik, hatalar ve doygunluk.

**Neden var:** Bir servisin ne yaptığından bağımsız olarak sağlığını anlamak için evrensel bir başlangıç noktası sağlarlar.

**İlgili servisler:** Cloud Monitoring.

**Yaygın yanlış anlamalar:** Gecikme, başarılı ve başarısız istekler için ayrı ölçülmelidir (hızlı başarısız olan bir 500 hatası, ortalama gecikmeyi yapay olarak düşürebilir). "Hatalar" HTTP 5xx'ten fazlasını kapsar — yanlış içerikli bir 200 yanıtı ya da bir SLA'yı ihlal eden bir yanıt da sayılır. Doygunluk, %100 kullanıma ulaşmadan **önce** bozulmaya başlayabilir.

**Gerçek dünya benzetmesi:** Dört altın sinyal, bir doktorun temel vital bulgu kontrolü gibidir — nabız, tansiyon, ateş, oksijen — daha derine inmeden önceki evrensel bir başlangıç noktası.

---

### Functions Framework

**Tanım:** Kullanıcı function kodunu kalıcı bir HTTP uygulamasında saran, Cloud Run functions için hem HTTP functions'ı hem CloudEvent functions'ı register etmek amacıyla kullanılan açık kaynaklı bir kütüphane.

**Neden var:** Cloud Run functions altta hâlâ Cloud Run'ın HTTP tabanlı container modeli üzerinde çalışır; Functions Framework, bir geliştiricinin tam bir HTTP server yerine küçük, tek amaçlı bir function yazmasına izin veren şeydir — geri kalanını Cloud Run functions halleder.

**İlgili servisler:** Cloud Run Functions, CloudEvent Function, Cloud Build, Artifact Registry.

**Yaygın yanlış anlamalar:** Functions Framework, Cloud Run functions'a özgü tescilli bir format değildir — dil başına implementasyonları olan (Node.js, Python, Go ve diğerleri) açık kaynaklı bir kütüphanedir; her biri o dilin standart paket manifest'ine (örn. `package.json`, `requirements.txt`, `go.mod`) bir dependency olarak dahil edilir.

**Gerçek dünya benzetmesi:** Functions Framework, her prize yerleşik, standartlaştırılmış bir elektrik adaptörü gibidir — duvarın arkasındaki kablolamayla (HTTP server) uğraşmadan kendi özel cihazınızı (function kodunuzu) takarsınız.

---

### Gemini

**Tanım:** Birçok Google Cloud ürününe gömülü, gcloud komutları üretme ya da kod yazmaya yardım gibi görevler için her zaman açık bir iş birlikçi olarak kullanılan Google Cloud'un üretken yapay zekâ modeli.

**Neden var:** Geliştiricilere, veri bilimcilere ve operatörlere zaten kullandıkları araçların içinde genel amaçlı bir yapay zekâ modeline doğrudan erişim vermek için.

**İlgili servisler:** Generative AI, Large Language Model, Prompt Engineering.

**Yaygın yanlış anlamalar:** Gemini'nin kullanışlılığı, prompt'unu ne kadar iyi ifade ettiğine büyük ölçüde bağlıdır — modelin altta yatan yeteneğinden bağımsız olarak, belirsiz prompt'lar belirsiz cevaplar getirir.

**Gerçek dünya benzetmesi:** Gemini, yanında oturan ve her soruya cevap verebilen uzman bir meslektaşın olması gibidir — ama sadece ona nasıl sorduğun kadar iyi.

---

### Generative AI

**Tanım:** Dar bir sınıflandırma sorusuna cevap vermek yerine, var olan içerikten örüntüleri öğrenerek yeni içerik (metin, görsel, ses, kod) üreten bir yapay zekâ türü.

**Neden var:** Dar ML modelleri "bu bir kedi mi, evet ya da hayır" sorusuna cevap verir; generative AI "kediler hakkında bildiğin her şeyi anlat" sorusuna cevap verir — temelde daha açık uçlu bir yetenek.

**İlgili servisler:** Large Language Model, Gemini, Prompt Engineering, Hallucination.

**Yaygın yanlış anlamalar:** Geleneksel programlama, dar makine öğrenmesi ve generative AI birbirinin yerine geçen alternatifler değil, bir **evrimdir** — her biri bir öncekinin bir sınırlamasını çözer.

**Gerçek dünya benzetmesi:** Geleneksel programlama, sabit bir tarif kartını takip etmektir. Dar ML, çok sayıda örnek tadarak bir yemeği öğrenen bir aşçıdır. Generative AI, genel mutfak bilgisini özümsemiş ve istek üzerine tamamen yeni bir yemek doğaçlayabilen bir şeftir.

---

### GKE (Google Kubernetes Engine)

**Tanım:** Google'ın control plane'i çalıştırdığı, yönetilen bir Kubernetes servisi; iki modda gelir — node pool'ları maksimum esneklik için kendin yönettiğin **Standard**, ve Google'ın node'ları da yönettiği, operasyonel efor'u en aza indiren **Autopilot**.

**Neden var:** Kubernetes'i kendin çalıştırmak, control plane'ini de yamalı, ölçekli ve yüksek kullanılabilir tutmak demektir — GKE bu yükü kaldırır.

**İlgili servisler:** Kubernetes, Pod, Deployment (Kubernetes), Service (Kubernetes), Container.

**Yaygın yanlış anlamalar:** GKE **Standard**'da Google control plane'i yönetir, ama **node'ları varsayılan olarak sen yönetirsin** — yaygın bir tuzak, Standard'ın da node yönetimini üstlendiğini varsaymaktır; bunu gerçekte yapan **Autopilot**'tur.

**Gerçek dünya benzetmesi:** GKE Standard, istediğin gibi düzenleyebileceğin ama alışverişini ve temizliğini yine kendin yaptığın bir mutfaklı ev gibidir. GKE Autopilot, bir otelin oda servisidir — sadece ne istediğini söylersin.

---

### Hallucination (LLM)

**Tanım:** Büyük bir dil modelinin, kendinden emin ama gerçeklere aykırı ya da anlamsız bir cevap ürettiği durum.

**Neden var (bir olgu olarak):** LLM'ler yalnızca eğitildikleri şeyi bilir, gerçek zamanlı bilgi erişimine sahip değildir, açıklayıcı sorular soramaz ve prompt'un doğru olduğunu varsayar — eğitim verisindeki, bağlamdaki ya da kısıtlardaki boşluklar, yanlış-ama-kendinden-emin bir cevap ihtimalini artırır.

**İlgili servisler:** Large Language Model, Prompt Engineering, Generative AI.

**Yaygın yanlış anlamalar:** Halüsinasyon nadir bir hata değildir — LLM'lerin çalışma biçiminin doğasında olan bir risktir; iyi prompt mühendisliğiyle (net talimatlar, yeterli bağlam, örnekler, tanımlı bir persona) azaltılır (asla tamamen ortadan kaldırılmaz).

**Gerçek dünya benzetmesi:** Halüsinasyon gören bir LLM, bir bilgi yarışmasında her soruya anında ve kendinden emin cevap veren, uydursa bile emin görünen bir misafir gibidir.

---

### Hybrid Connectivity (Cloud VPN, Interconnect, Peering)

**Tanım:** Bir Google VPC'sini on-premises ya da başka bulut ağlarına bağlamak için bir seçenekler ailesi: **Cloud VPN** (internet üzerinden tünel, Cloud Router ile dinamik yönlendirme), **Direct Peering**/**Carrier Peering** (bir Google varlık noktasında trafik değişimi, SLA'ya tabi değil), **Dedicated Interconnect**/**Partner Interconnect** (özel fiziksel bağlantılar, %99,99'a varan SLA) ve **Cross-Cloud Interconnect** (başka bir bulut sağlayıcıya özel, yüksek bant genişlikli bağlantı, 10 Gbps ya da 100 Gbps).

**Neden var:** Çoğu kuruluş %100 bulutta değildir — Google Cloud ile mevcut ağları arasında güvenilir, bazen SLA'ya tabi bağlantıya ihtiyaç duyarlar.

**İlgili servisler:** VPC, Cloud Router.

**Yaygın yanlış anlamalar:** Peering (direct ya da carrier) **bir Google SLA'sı kapsamında değildir** — garantili çalışma süresi önemliyse bunun yerine Dedicated ya da Partner Interconnect gerekir.

**Gerçek dünya benzetmesi:** Cloud VPN, genel internet üzerinden yapılan bir telefon görüşmesidir — çoğu konuşma için yeterince güvenilirdir. Dedicated Interconnect, doğrudan binaya çekilen özel, kiralık bir telefon hattıdır — bir bedel karşılığında garantili kalite.

---

### IAM (Identity and Access Management)

**Tanım:** *Kimin* (principal) *neyi* (rol) *hangi kaynak üzerinde* yapabileceğini tanımlayan sistem — Google Cloud'daki yetkilendirmenin omurgası.

**Neden var:** Bir kuruluş çok sayıda klasöre, projeye ve kaynağa sahip olduğunda, "kim neye erişebilir" merkezi olarak yönetilmelidir, rastgele değil.

**İlgili servisler:** Principal, IAM Roles, Resource Hierarchy, Service Account.

**Yaygın yanlış anlamalar:** Bir izni asla doğrudan bir kullanıcıya vermezsin — her zaman `service.resource.verb` biçiminde izinlerin bir paketi olan bir **rol** verirsin. Politikalar, kaynak hiyerarşisi boyunca **aşağı doğru** miras alınır.

**Gerçek dünya benzetmesi:** IAM, bir binanın erişim rozeti sistemidir: rozetler (roller) insanlara (principal'lara) atanır ve belirli kapıları (kaynakları) açar — kimse doğrudan tek bir kapının anahtarını almaz.

---

### Identity-Aware Proxy (IAP)

**Tanım:** Geliştirici herhangi bir erişim kontrolü kodu yazmadan, bir kullanıcının kimliğini doğrulayan ve belirli bir bulut uygulamasına erişip erişemeyeceğine karar veren bir servis.

**Neden var:** VPN'ler ve ağ güvenlik duvarları "doğru ağdasın" garantisi verir; bu, "kim olduğunu söylediğin kişisin ve buraya izinlisin"den daha zayıf bir garantidir — IAP, erişimi bunun yerine uygulama düzeyinde zorunlu kılar.

**İlgili servisler:** IAM, Identity Platform, OAuth 2.0.

**Yaygın yanlış anlamalar:** IAP, uygulama içinde özel yetkilendirme kodu yazma ihtiyacının **yerine geçer** — böyle bir kodun yanında ek bir katman değil, onu ortadan kaldırmayı hedefler.

**Gerçek dünya benzetmesi:** Bir VPN, "bu binaya girebilirsin" der. IAP, "bu binaya, *ve özellikle* şu odaya girebilirsin, o odaya değil" der — elle kontrol eden bir resepsiyonist olmadan.

---

### Identity Platform

**Tanım:** Firebase Authentication artı kurumsal düzey yetenekler: OpenID Connect ve SAML ile giriş, çok faktörlü kimlik doğrulama ve Identity-Aware Proxy entegrasyonu.

**Neden var:** Kurumsal müşterilerin, basit bir mobil/web giriş akışının sağlamadığı SSO standartlarına, MFA'ya ve merkezi erişim kontrolüne ihtiyacı vardır.

**İlgili servisler:** Firebase Authentication, Identity-Aware Proxy, OAuth 2.0.

**Yaygın yanlış anlamalar:** "MFA", "SAML" ya da "IAP entegrasyonu" gibi gereksinimler sade Firebase Authentication'ı değil Identity Platform'u işaret eder — onu "Firebase Authentication + kurumsal özellikler" olarak düşün.

**Gerçek dünya benzetmesi:** Firebase Authentication bir dükkân kapısındaki tuşlu kilit sistemiyse, Identity Platform kurumsal bir rozet ve ziyaretçi yönetim sistemidir.

---

### Infrastructure as a Service (IaaS)

**Tanım:** Ham işlem, depolama ve ağı sanal kaynaklar olarak sunan bir bulut servis modeli; işletim sistemini ve yukarısını sen seçer ve yönetirsin; ayrılan (rezerve edilen) kaynaklar için ödersin.

**Neden var:** Bazı iş yükleri, daha yüksek soyutlama düzeylerinin (PaaS, serverless) sunmadığı, işletim sistemi ve yazılım yığını üzerinde tam kontrole ihtiyaç duyar.

**İlgili servisler:** Compute Engine, Platform as a Service (PaaS), Serverless.

**Yaygın yanlış anlamalar:** IaaS, kullansan da kullanmasan da **ayırdığın** kaynaklar için seni faturalandırır — bu, PaaS'tan (kullandığın kadar öde) ve serverless'tan (istek/ms başına öde) temelde farklıdır.

**Gerçek dünya benzetmesi:** IaaS, boş bir daire kiralamaktır — bina hazırdır ama içini döşemek, dekore etmek ve her şeyi bakımını yapmak sana kalmıştır.

---

### Kubernetes

**Tanım:** Konteynerize iş yüklerini dağıtmak, ölçeklemek ve işletmek için, ilk olarak Google'da geliştirilmiş ve şimdi CNCF tarafından bakımı yapılan, önde gelen açık kaynak platform.

**Neden var:** Hangi container'ın nerede çalıştığını, nasıl ağlandığını, biri çökünce ne olacağını ve kaç replika olması gerektiğini elle yönetmek, bir orkestrasyon çerçevesi olmadan ölçekte imkânsız hale gelir.

**İlgili servisler:** GKE, Pod, Deployment (Kubernetes), Service (Kubernetes), Node.

**Yaygın yanlış anlamalar:** Kubernetes'teki bir "node", cluster'daki bir compute örneğidir (makine) — bu terimi Google Cloud'un başka yerlerindeki ilgisiz "node" kullanımlarıyla (kaynak hiyerarşisindeki "organizasyon düğümü" gibi) karıştırma.

**Gerçek dünya benzetmesi:** Kubernetes bir orkestra şefidir — şef kimin ne zaman çalacağına karar verir ve bir müzisyen hastalanırsa yerine birini bulur, ama sesi fiilen üreten müzisyenlerdir (node'lar).

---

### Large Language Model (LLM)

**Tanım:** Devasa metin veri kümeleriyle eğitilmiş, milyarlarca-trilyonlarca parametreye sahip, belirli görevler için ince ayar yapılabilen, önceden eğitilmiş, genel amaçlı bir dil modeli.

**Neden var:** "Büyük", hem eğitim verisinin ölçeğini (bazen petabayt) hem de parametre sayısını ifade eder — tek bir modelin geniş bir dil görevleri yelpazesini makul biçimde ele almasını sağlayan bu ölçektir.

**İlgili servisler:** Generative AI, Hallucination, Prompt Engineering, Gemini.

**Yaygın yanlış anlamalar:** LLM, generative AI'ın dil odaklı bir alt kümesidir, onunla eş anlamlı değildir — diğer foundation model türleri metin yerine görüntü ya da kod üzerinde eğitilir.

**Gerçek dünya benzetmesi:** Bir LLM, devasa bir kütüphaneyi özümsemiş, hemen hemen her konuda bilgiyle konuşabilen, çok okumuş bir genel kültür sahibi gibidir — ama tek bir dar alanda uzman olmak için özel eğitime (fine-tuning) gönderilebilir.

---

### Least Privilege (Principle of Least Privilege)

**Tanım:** Her principal'a işini yapmak için gerçekten gereken erişimin — ne fazla ne eksik — verilmesi gerektiği ilkesi.

**Neden var:** Bir principal'ın sahip olduğu her ekstra izin saldırı yüzeyini büyütür — o kimlik ele geçirilirse, gereksiz izin sayısı azken hasar da daha küçük olur.

**İlgili servisler:** IAM Roles, Service Account, Secret Manager.

**Yaygın yanlış anlamalar:** Temel (geniş) IAM rolleri en az ayrıcalığın tam tersidir ve üretimde genellikle önerilmez — önceden tanımlı ya da özel roller çoğu gerçek iş yükü için daha uygundur.

**Gerçek dünya benzetmesi:** En az ayrıcalık, bir çalışana "her ihtimale karşı" tüm binanın ana anahtarı yerine, yalnızca çalıştığı odaları açan bir anahtar vermektir.

---

### Log-Based Alert and Log-Based Metric

**Tanım:** İki Cloud Logging mekanizması: bir **log tabanlı uyarı**, bir log girdisinde belirli bir desen göründüğü anda seni bilgilendirir; bir **log tabanlı metrik**, oluşumları zaman içinde sayar (trendler ya da eşik tabanlı uyarılar için).

**Neden var:** Bazı durumlar gerçekleştiği an anlık bir bildirim gerektirir; diğerleri zaman içinde trend takibi gerektirir — tek bir mekanizma ikisine de uymaz.

**İlgili servisler:** Cloud Logging, Cloud Monitoring.

**Yaygın yanlış anlamalar:** "Bu olduğu anda beni bilgilendir" bir log tabanlı **uyarı** gerektirir; "bunun kaç kez olduğunu say ve bir eşiği aşınca bilgilendir" bir log tabanlı **metrik** gerektirir — bunlar sık sık yer değiştirir.

**Gerçek dünya benzetmesi:** Log tabanlı bir uyarı, dumanı algıladığı an bip sesi çıkaran bir duman dedektörüdür. Log tabanlı bir metrik, duman dedektörünün bu ay kaç kez çaldığını izleyen ve artış trendindeyse seni uyaran bir sayaçtır.

---

### Managed Instance Group (MIG)

**Tanım:** Bir instance template'ten oluşturulan, autoscaling'i, otomatik değiştirmeli sağlık kontrollerini ve global load balancing'i destekleyen bir özdeş Compute Engine VM'i grubu.

**Neden var:** Tek tek VM'leri elle yönetmek ölçeklenmez; MIG'ler bir VM filosunu tek, yönetilebilir, kendi kendini onaran bir birim olarak ele almanı sağlar.

**İlgili servisler:** Compute Engine, Autoscaling, Cloud Load Balancing.

**Yaygın yanlış anlamalar:** Bu kendi kendini onarma ve ölçekleme davranışları, sadece Compute Engine kullandığın için otomatik olarak gerçekleşmez — MIG'i kendin yapılandırmalısın; buna karşın GKE ya da Cloud Run'da benzer davranış varsayılan olarak yerleşiktir.

**Gerçek dünya benzetmesi:** MIG, aynı plandan özdeş mağazalar açan, düşük performanslı birini otomatik olarak kapatıp yerine yenisini açan bir franchise operatörüdür.

---

### Memorystore

**Tanım:** Sık erişilen ya da hesaplaması pahalı uygulama verisini önbelleğe almak için Redis ya da Memcached kullanan, tam yönetilen, bellek içi bir veri deposu.

**Neden var:** Kendi Redis/Memcached kümeni sağlamak ve işletmek — provizyonlama, replikasyon, yük devretme, yamalama — Memorystore'un senden aldığı önemli, sürekli bir iştir.

**İlgili servisler:** Cloud CDN, VPC, IAM.

**Yaygın yanlış anlamalar:** Memorystore, bellekte **uygulama verisini** önbelleğe alır; bu, ağ kenarında **web/statik içeriği** önbelleğe alan Cloud CDN'den farklı bir katmandır. Memorystore, birincil doğruluk kaynağın olması amaçlanmamıştır — kalıcı veri, kalıcı bir veritabanında yaşamalıdır.

**Gerçek dünya benzetmesi:** Memorystore, anında erişim için tezgahın arkasında tutulan küçük mal stoğudur; tam depo (kalıcı veritabanın) ise kalıcı envanterin yaşadığı yer olmaya devam eder.

---

### Microservices

**Tanım:** Bir uygulamayı, her biri kendi veritabanına sahip ve diğer servisler tarafından kullanılan bir API sunan, ayrı, kapsamı sınırlı servislere ayrıştıran decentralized (merkeziyetsiz) bir mimari stil.

**Neden var:** Monolithler büyüdükçe sıkı bağlı ve bakımı zor hale gelir, SOA'nın paylaşılan Enterprise Service Bus (ESB)'i ise merkezi bir darboğazı yeniden yarattı; microservices, paylaşılan middleware'i tamamen kaldırarak servislerin bağımsız olarak inşa edilmesini, deploy edilmesini ve ölçeklenmesini sağlar.

**İlgili servisler:** Monolith (Monolithic Application), Service-Oriented Architecture (SOA), Cloud Run, GKE.

**Yaygın yanlış anlamalar:** Microservices otomatik olarak "daha iyi" ya da varsayılan başlangıç noktası değildir — domain uzmanlığı olmadan servis sınırlarını tasarlamak yeni bir projenin en zor kısımlarından biridir, bu yüzden modüler bir monolith ile başlayıp sonra geçiş yapmak genelde daha güvenli bir yoldur. Microservices ayrıca compute latency'sini artırır (in-process çağrılar ağ çağrılarına dönüşür) ve bağımsız deploy edilebilirlik, teknoloji özgürlüğü ve bağımsız ölçeklenme karşılığında gerçek bir operasyonel yük ekler (otomatik build/test/deploy, tutarlı logging ve güvenlik, dağıtık loglar üzerinde daha zor debugging).

**Gerçek dünya benzetmesi:** Bir microservices mimarisi, her biri kendi kasasını ve kendi deposunu işleten, kendi sokak adresinden (API) ulaşılabilen bağımsız dükkanlardan oluşan bir şehir gibidir — her kasanın aynı merkezi arka ofis üzerinden çalıştığı tek bir dev mağaza yerine.

---

### Monolith (Monolithic Application)

**Tanım:** UI, business logic ve data access'in hepsinin tek bir deploy edilebilir birimde toplandığı, genelde tek büyük bir ilişkisel veritabanının üzerinde çalışan, tek, kendi kendine yeten bir kod tabanı olarak inşa edilmiş uygulama.

**Neden var:** Çoğu uygulama için doğal başlangıç noktasıdır: tek kod tabanı, tek deployment, dağıtık sistemlerin getirdiği ek yük yok — basit kalır, ta ki kalmayana kadar.

**İlgili servisler:** Microservices, Service-Oriented Architecture (SOA).

**Yaygın yanlış anlamalar:** Bir monolith doğası gereği "kötü bir mimari" değildir — problem domain'ini iyi servis sınırları çizecek kadar henüz anlamıyorsan, kasıtlı olarak (modüler) bir monolith ile başlayıp daha fazlasını öğrendikten sonra microservices'e geçmek meşru ve genelde önerilen bir stratejidir.

**Gerçek dünya benzetmesi:** Bir monolith, kasanın, envanterin ve müşteri hizmetleri masalarının hepsinin aynı arka ofisi paylaştığı tek bir büyük mağazadır — açılması hızlıdır, ama bir köşedeki bir tadilat tüm binanın elektrik tesisatını devre dışı bırakabilir.

---

### Multi-Region

**Tanım:** Kaynakları, bir bölgedeki birden çok zon yerine, **birden çok bölgedeki** birden çok zona kopyalayan bir yapılandırma.

**Neden var:** Çoklu zon tek bir zonun çökmesine karşı korur; çoklu bölge, bir bölgenin tamamını devre dışı bırakan bir şeye (doğal afet gibi) karşı korur ve veriyi farklı coğrafyalardaki kullanıcılara yaklaştırır.

**İlgili servisler:** Region, Zone, Spanner, Cloud Storage Classes.

**Yaygın yanlış anlamalar:** Çoklu zon ve çoklu bölge farklı sorunları çözer — zonlar tek bir coğrafya içinde yedeklilik verir, bölgeler hem kullanıcılara coğrafi yakınlık hem de bölge çapında kesintilerden korunma sağlar.

**Gerçek dünya benzetmesi:** Çoklu zon, aynı binada yedek jeneratörlere sahip olmak gibidir. Çoklu bölge, başka bir şehirde ikinci, tamamen bağımsız bir binaya sahip olmak gibidir.

---

### Natural Language AI

**Tanım:** Belgelerden, makalelerden ya da sosyal medya gönderilerinden yapılandırılmış anlam — varlıklar, duygu ve niyet — çıkaran önceden eğitilmiş bir API.

**Neden var:** Duyguyu ya da niyeti ölçmek için binlerce müşteri mesajını elle okumak ölçeklenmez; bu API bu çıkarımı otomatikleştirir.

**İlgili servisler:** Vision AI, Document AI, Translation AI.

**Yaygın yanlış anlamalar:** Natural Language AI, anahtar kelime aramasından fazlasını yapar — sadece kelimeleri eşleştirmek değil, bağlamı anlamayı gerektiren duygu (olumlu/olumsuz) ve niyet (satın alma vs. şikâyet) çıkarır.

**Gerçek dünya benzetmesi:** Natural Language AI, bir yığın müşteri yorumunu göz gezdiren ve hangilerinin mutlu, hangilerinin kızgın olduğunu, her kişinin gerçekte ne istediğini anında söyleyen bir okuyucu gibidir.

---

### Node (Kubernetes)

**Tanım:** Bir Kubernetes cluster'ında, control plane tarafından yönetilen ve container'larını fiilen çalıştıran bir makine (sanal ya da fiziksel).

**Neden var:** Control plane zamanlama kararları verir, ama işin kendisi — container'ları çalıştırmak — bir yerde gerçekleşmelidir; node'lar o "bir yer"dir.

**İlgili servisler:** Kubernetes, Pod, GKE.

**Yaygın yanlış anlamalar:** Kubernetes "node"u, kaynak hiyerarşisinin tepesindeki "organizasyon düğümü"yle ilgisizdir — aynı kelime, tamamen farklı bir kavram.

**Gerçek dünya benzetmesi:** Control plane bir orkestra şefiyse, node'lar müziği fiilen üreten bireysel orkestra üyeleridir.

---

### OAuth 2.0

**Tanım:** Bir kullanıcı, bir yetkilendirme sunucusu üzerinden açık onay verdikten sonra, bir uygulamanın o kullanıcı adına kaynaklara erişmesine izin veren bir protokol.

**Neden var:** Uygulamaların bazen bir kullanıcının parolasını asla görmeden onun adına hareket etmesi (örneğin kullanıcının kendi BigQuery veri kümelerini okumak) gerekir.

**İlgili servisler:** Identity-Aware Proxy, Firebase Authentication, Identity Platform.

**Yaygın yanlış anlamalar:** OAuth 2.0 erişimi, kullanıcının tam olarak onayladığı şeyle sınırlıdır — bir uygulama onaylanandan daha geniş erişimi sessizce kazanamaz.

**Gerçek dünya benzetmesi:** OAuth 2.0, bir vale görevlisine sadece arabayı çalıştırıp bagajı açan sınırlı erişimli bir anahtar vermeye benzer — evini de açan bir anahtar değil.

---

### OLTP and OLAP

**Tanım:** **OLTP** (Online Transaction Processing), çok sayıda küçük, hızlı işlem demektir — sipariş oluştur, bakiye güncelle. **OLAP** (Online Analytical Processing), büyük ölçekli analitik sorgular demektir — toplama, trend analizi, raporlama.

**Neden var:** Bu iki erişim deseni zıt performans profillerine sahiptir, bu yüzden temelde farklı sistemler tarafından hizmet edilir.

**İlgili servisler:** Cloud SQL, AlloyDB, Spanner, BigQuery.

**Yaygın yanlış anlamalar:** Tek bir veritabanı nadiren ikisini de iyi yapar — OLTP veritabanları (Cloud SQL, AlloyDB, Spanner) analitik ambar değildir ve BigQuery milisaniyelik tekil satır işlemleri için inşa edilmemiştir. AlloyDB'nin Columnar Engine'i, transactional performanstan ödün vermeden ikisini birleştiren (HTAP) dikkate değer bir istisnadır.

**Gerçek dünya benzetmesi:** OLTP, bir seferde tek bir satışı hızla çarpan bir yazar kasadır. OLAP, şimdiye kadar yapılan her satışı analiz eden çeyrek sonu raporudur.

---

### Ops Agent

**Tanım:** Compute Engine VM'lerine kurulan, logları (Fluent Bit ile) ve metrikleri (OpenTelemetry Collector ile) toplayıp Cloud Logging ve Cloud Monitoring'e gönderen ajan.

**Neden var:** Bir ajan olmadan, bir VM'de çalışan üçüncü taraf yazılımların (NGINX ya da Tomcat gibi) logları ve metrikleri makineden asla dışarı çıkmaz.

**İlgili servisler:** Cloud Logging, Cloud Monitoring, Compute Engine.

**Yaygın yanlış anlamalar:** Ops Agent tek, monolitik bir araç değildir — birlikte paketlenmiş iki özelleşmiş açık kaynak bileşendir (loglar için Fluent Bit, metrikler için OpenTelemetry Collector).

**Gerçek dünya benzetmesi:** Ops Agent, kiralık bir daireye hem duman dedektörü hem termostat kurmak gibidir — ayrı, özelleşmiş cihazlar, aynı merkezi izleme servisine rapor verir.

---

### Organization Node

**Tanım:** Bir kuruluşun hesabına bağlı her klasörü, projeyi ve kaynağı kapsayan, kaynak hiyerarşisinin en üst katmanı.

**Neden var:** Tepede birinin tüm hiyerarşi üzerinde yetkisi olması gerekir — kuruluş çapında politika için, ve hatta kimin yeni bir proje oluşturabileceğini kontrol etmek için.

**İlgili servisler:** Resource Hierarchy, Folder, Project, Cloud Identity.

**Yaygın yanlış anlamalar:** Bir Google Workspace domain'in varsa organizasyon düğümü otomatik olarak oluşur; yoksa Cloud Identity aracılığıyla bir tane üretirsin — kendiliğinden ortaya çıkmaz.

**Gerçek dünya benzetmesi:** Organizasyon düğümü, bir şirketin genel merkezidir — her departman (klasör) ve ekip (proje) sonunda ona bağlanır.

---

### Platform as a Service (PaaS)

**Tanım:** Kodunun, altyapıya erişim sağlayan kütüphanelere bağlandığı bir bulut servis modeli — altyapıyla değil uygulama mantığıyla ilgilenirsin; gerçekten kullandığın kaynaklar için ödersin.

**Neden var:** "Her şeyi kendin yönet" (IaaS) ile "hiçbir şey yönetme" (serverless) arasında, PaaS altyapı endişelerini atlamana izin verir ama yine de geleneksel bir uygulama dağıtırsın.

**İlgili servisler:** App Engine, Infrastructure as a Service (IaaS), Serverless.

**Yaygın yanlış anlamalar:** PaaS hâlâ yönettiğin bir uygulamayı içerir (serverless fonksiyonların aksine) — IaaS'tan soyutlama merdiveninde bir adım yukarıdır, en tepesi değil.

**Gerçek dünya benzetmesi:** PaaS, tam donanımlı, servisli bir ofise taşınmaktır — elektrik, internet ve mobilya hazırdır; sadece gelip çalışmaya başlarsın.

---

### Pod (Kubernetes)

**Tanım:** Kubernetes'te oluşturulabilecek en küçük birim — ağ ve depolamayı paylaşan bir ya da daha fazla container'ın etrafındaki sarmalayıcı.

**Neden var:** Kubernetes'in atomik bir zamanlama ve ağ birimine ihtiyacı vardır; sıkı bağlı container'ları tek bir Pod'da gruplamak onlara paylaşılan bir IP ve yaşam döngüsü verir.

**İlgili servisler:** Deployment (Kubernetes), Service (Kubernetes), Node, GKE.

**Yaygın yanlış anlamalar:** Çoğu Pod tam olarak bir container içerir — çok container'lı Pod'lar istisnadır, yalnızca container'lar sıkı bağlı olduğunda ve kaynak paylaşması gerektiğinde kullanılır.

**Gerçek dünya benzetmesi:** Bir Pod, tek bir öğe ya da birlikte taşınıp teslim edilmesi gereken birkaç sıkı ilişkili öğe tutabilen bir sevkiyat kasasıdır.

---

### Predefined, Basic, and Custom Roles (IAM Roles)

**Tanım:** Farklı kapsam ve sahiplikte üç IAM rol türü: **Basic** roller (Viewer, Editor, Owner, Billing Administrator) geniştir ve Google tarafından yönetilir; **Predefined** roller Google tarafından yönetilir ama belirli bir servise/işe göre kapsamlandırılmıştır; **Custom** roller kullanıcı tanımlıdır, en az ayrıcalığın gerektirdiği tam izin seti içindir.

**Neden var:** İzinleri tek tek vermek sıkıcı ve hataya açık olurdu — ilgili izinleri farklı ayrıntı düzeylerinde adlandırılmış rollerde paketlemek farklı gerçek dünya ihtiyaçlarına uyar.

**İlgili servisler:** IAM, Principal, Least Privilege.

**Yaygın yanlış anlamalar:** Basic roller geniştir ve üretimde genellikle önerilmez. Custom roller **yalnızca** proje ya da organizasyon düzeyinde uygulanabilir — asla klasör düzeyinde, ki bu sık görülen bir sınav tuzağıdır.

**Gerçek dünya benzetmesi:** Basic roller bir ana anahtardır. Predefined roller belirli bir iş için etiketli bir anahtarlıktır (örn. "hademe anahtarları"). Custom roller kendin kestiğin, tam olarak belirttiğin kapıları açan bir anahtardır.

---

### Preemptible VM / Spot VM

**Tanım:** Google'ın kapasiteyi geri almak istemesi halinde sonlandırabildiği, indirimli (yüzde 90'a varan) Compute Engine VM'leri; büyük, kesintiye toleranslı toplu iş yükleri için idealdir.

**Neden var:** Google'ın veri merkezlerinde her an atıl kapasite vardır; bunu ucuza satmak, geri alınabilir olması koşuluyla, hem Google'a hem maliyet bilincindeki müşterilere fayda sağlar.

**İlgili servisler:** Compute Engine, Managed Instance Group.

**Yaygın yanlış anlamalar:** Spot VM'lerin (eski Preemptible modelinin modern halefi) **maksimum çalışma süresi yoktur** — eski modelin aksine, kapasite mevcut olduğu sürece süresiz çalışabilirler; yine de her an sonlandırılabilirler.

**Gerçek dünya benzetmesi:** Bir Spot VM, yedek bir uçak koltuğudur — harika bir indirimdir, ama tam ücretli bir yolcunun koltuğa ihtiyacı olursa koltuğundan çıkarılabilirsin.

---

### Principal (IAM)

**Tanım:** Bir IAM politikasındaki "kim". Google hesapları ve servis hesapları *kimlik doğrulayabilir* (kimliğini kanıtlayabilir); Google grupları, Google Workspace hesapları ve Cloud Identity domain'leri **kimlik doğrulayamaz** — sadece toplu izin yönetimini kolaylaştırırlar.

**Neden var:** Erişim kontrolünün, o kişi, bir uygulama ya da ikisinin toplu bir koleksiyonu olsun, "kim"in erişim istediğini tutarlı biçimde belirlemesi gerekir.

**İlgili servisler:** IAM, Service Account, Cloud Identity.

**Yaygın yanlış anlamalar:** Bu, en çok sınavda sorulan ayrımlardan biridir: bir Google grubu, bir Google Workspace hesabı ve bir Cloud Identity domain'i **bir API isteğini imzalayamaz** — yalnızca bir Google hesabı (bir insan) ya da bir servis hesabı (bir workload) yapabilir.

**Gerçek dünya benzetmesi:** Bir Google hesabı ya da servis hesabı, kendi kimlik kartını taşıyan bir kişidir. Bir grup bir posta listesidir — tüm listeye birden hitap edebilirsin, ama listenin kendisi kapıda kimlik gösteremez.

---

### Progressive Delivery (Canary and Blue-Green)

**Tanım:** İki güvenli yayın stratejisi: **Canary**, yeni bir sürümü önce trafiğin küçük bir yüzdesine açar ve kademeli olarak artırır; **Blue-Green**, iki özdeş ortam tutar ve tüm trafiği aralarında anında değiştirir, anında geri dönüşle.

**Neden var:** Yeni bir derlemeyi bir anda kullanıcıların %100'üne yaymak, bir hatanın herkesi aynı anda etkilemesi demektir — bu stratejiler kötü bir yayının hasar alanını sınırlar.

**İlgili servisler:** Cloud Deploy, Continuous Integration, Delivery, and Deployment (CI/CD).

**Yaygın yanlış anlamalar:** Canary *kademeli* bir trafik kaydırmasıdır; blue-green *anlık, tek seferlik* bir geçiştir, aynı derecede anlık bir geri dönüş yoluyla — aynı sorunu farklı biçimlerde çözerler, birbirinin yerine geçmezler.

**Gerçek dünya benzetmesi:** Canary release, bir şefin yeni bir yemeği tüm menüye koymadan önce birkaç masaya sunması gibidir. Blue-green, bir restoranın tüm menüsünü gece boyunca değiştirmesi gibidir — bir şeyler ters giderse eski menüyü anında geri getirmeye hazır olarak.

---

### Project

**Tanım:** Kaynak hiyerarşisinin ikinci katmanı ve Google Cloud servislerini etkinleştirmenin ve kullanmanın temel birimi — her kaynak tam olarak bir projeye aittir.

**Neden var:** Faturalandırma, API yönetimi ve iş birlikçi erişiminin hepsi bağlanacak bir sınıra ihtiyaç duyar — proje bu sınırdır.

**İlgili servisler:** Resource Hierarchy, Folder, Organization Node.

**Yaygın yanlış anlamalar:** Bir **Project ID** küresel olarak benzersizdir ve bir kere ayarlandığında **değişmezdir (immutable)**. Bir **Project name** kullanıcı tarafından atanır ve istediğin zaman **değiştirilebilir**. Bu ikisini karıştırmak klasik bir sınav tuzağıdır.

**Gerçek dünya benzetmesi:** Bir proje, aynı büyük kuruluşa ait başka hesaplarla aynı kuruluşa ait olsa bile, kendi kimlik numarasına, kendi işlemlerine ve kendi yetkili kullanıcı setine sahip bireysel bir banka hesabı gibidir.

---

### Prometheus (Google Cloud Managed Service for Prometheus)

**Tanım:** Popüler açık kaynak Prometheus izleme araç setinin, Prometheus'u kendin işletme ve ölçekleme yükünü ortadan kaldıran, PromQL'i ve mevcut ekosistemi koruyan Google Cloud'un tam yönetilen versiyonu.

**Neden var:** PromQL ve Prometheus dashboard'larını zaten bilen ekipler, sadece yönetilen bir deneyim elde etmek için bu bilgiyi çöpe atmak zorunda kalmamalıdır.

**İlgili servisler:** Cloud Monitoring, GKE.

**Yaygın yanlış anlamalar:** Herhangi bir Kubernetes ortamı için (GKE dahil), kendi kendine dağıtılanlar yerine **managed collector'lar önerilir** — bir Kubernetes operatörü operasyonel yükü senin için üstlenir.

**Gerçek dünya benzetmesi:** Favori tarif kitabını (PromQL) elinde tutmak ama alışverişi, hazırlığı ve temizliği kendin yapmak yerine bunları halledecek bir mutfak ekibi (managed collector'lar) tutmak gibidir.

---

### Prompt Engineering

**Tanım:** Bir preamble (bağlam/talimatlar) ve input (asıl istek) kullanarak, büyük bir dil modelinden mümkün olan en iyi yanıtı almak için prompt'ları (zero-shot, one-shot, few-shot ya da role tabanlı) ifade etme pratiği.

**Neden var:** Bir model yalnızca eğitim verisindeki ve prompt'ta ona verdiğin şeyi bilir — iyi yapılandırılmış prompt'lar belirsizliği ve halüsinasyonu doğrudan azaltır.

**İlgili servisler:** Large Language Model, Gemini, Hallucination.

**Yaygın yanlış anlamalar:** Prompt'ta daha fazla örnek her zaman gerekli değildir — zero-shot basit sorular için iyi çalışır, ama teknik görevler genellikle en az bir örnekten (one-shot) ya da birkaçından (few-shot) fayda görür.

**Gerçek dünya benzetmesi:** Prompt mühendisliği, yeni bir danışmanı bilgilendirmeye benzer: talimatların, bağlamın ve örneklerin ne kadar netse, cevapları o kadar iyi ve ilgili olur.

---

### Pub/Sub

**Tanım:** Bağımsız servislerin ya da uygulamaların mesaj göndermesini ve almasını sağlayan, tamamen yönetilen, gerçek zamanlı bir messaging servisi: bir publisher bir topic'e bir mesaj gönderir, ve mesaj her subscriber'ın kuyruğuna teslim edilir.

**Neden var:** Event-driven ya da choreographed bir mimarideki servislerin, publisher'ın kimin (varsa) dinlediğini bilmesine ya da umursamasına gerek kalmadan mesaj geçirmenin kalıcı, decoupled bir yoluna ihtiyacı vardır.

**İlgili servisler:** Eventarc, Event-Driven Architecture, Cloud Tasks.

**Yaygın yanlış anlamalar:** Bir mesaj, ancak acknowledge edildikten sonra bir subscriber'ın kuyruğundan kaldırılır — bu, **at-least-once** teslimatı garanti eder (bir mesaj yeniden teslim edilebilir), exactly-once değil. Ayrıca, bir **pull subscription**'da subscriber mesajlar için poll eder, bir **push subscription**'da ise Pub/Sub mesajları yapılandırılmış bir endpoint'e otomatik olarak gönderir — bunlar aynı mekanizma değildir.

**Gerçek dünya benzetmesi:** Pub/Sub, bir dergi aboneliği gibidir: publisher bir sayıyı bir kez yazar ve basar, ve mevcut her subscriber otomatik olarak kendi kopyasını alır — publisher kaç subscriber olduğunu ya da kim olduklarını takip etmez, umursamaz.

---

### Push-Based Messaging vs. Polling

**Tanım:** Bir consumer'ın yeni işin mevcut olduğunu öğrenmesinin iki yolu: polling, bir kaynağa yeni bir şey olup olmadığını tekrar tekrar sorar; push-based messaging ise consume edilecek bir event olduğunda consumer'ı otomatik olarak bilgilendirir.

**Neden var:** "Yeni bir şey var mı henüz?" diye sürekli sormak network I/O'sunu boşa harcar ve işin mevcut olduğu an ile gerçekten alındığı an arasına gecikme ekler — push-based teslimat her iki maliyetten de kaçınır.

**İlgili servisler:** Event, Event-Driven Architecture.

**Yaygın yanlış anlamalar:** Polling, push'un gerçek bir dezavantajı olmayan, sadece "daha az gelişmiş" bir versiyonu değildir — modül, polling'in genelde network I/O'yu artırdığını ve gereksiz işleme gecikmesi yarattığını açıkça belirtir; bu yüzden event consumer'ları için tercih edilen model push-based messaging'dir.

**Gerçek dünya benzetmesi:** Polling, bir restoranı tekrar tekrar arayıp "siparişim hazır mı?" diye sormaktır. Push-based messaging ise restoranın sipariş hazır olduğu anda sana mesaj atmasıdır — kontrol etmek için çaba harcamazsın ve harekete geçilecek bir şey olduğu anda haberin olur.

---

### Quotas (Rate and Allocation)

**Tanım:** Kaçak kaynak tüketimini önleyen proje düzeyinde limitler. **Oran kotaları** sabit bir zaman penceresinden sonra sıfırlanır (örn. 100 saniyede 3.000 API çağrısı); **tahsis kotaları**, aynı anda tutabileceğin bir kaynak sayısını sınırlar (örn. proje başına 15 VPC ağı).

**Neden var:** Limitler olmadan, bir hata ya da kötü niyetli aktör paylaşılan kaynakları tüketebilir; bu hem hesap sahibine hem daha geniş Google Cloud topluluğuna zarar verir.

**İlgili servisler:** Resource Hierarchy, VPC, GKE.

**Yaygın yanlış anlamalar:** "Zamanla sıfırlanır" bir **oran (rate)** kotasıdır; "kaç tane tutabilirsin" bir **tahsis (allocation)** kotasıdır — sınav sorularında sık sık yer değiştirir.

**Gerçek dünya benzetmesi:** Bir oran kotası, her ay sıfırlanan bir veri paketi gibidir. Bir tahsis kotası, sabit sayıda rafı olan bir depolama dolabı gibidir — yeniden dolmaz, sadece aynı anda ne kadar tutabileceğini sınırlar.

---

### Region

**Tanım:** Zonlardan oluşan bağımsız bir coğrafi alan (örn. `europe-west2` / Londra).

**Neden var:** Uygulamanı nereye koyduğun, kullanıcıların için kullanılabilirliği, dayanıklılığı ve gecikmeyi doğrudan etkiler — bölgeler sana coğrafi seçim sunar.

**İlgili servisler:** Zone, Multi-Region.

**Yaygın yanlış anlamalar:** Bir bölge, bir zonla aynı ayrıntı düzeyinde değildir — kaynaklar fiilen bir bölgenin *içindeki* bir zonda çalışır; bu iki terim genellikle gevşek kullanılır ama farklı şeyler ifade eder.

**Gerçek dünya benzetmesi:** Bir bölge bir şehirdir; bir zon o şehrin mahallelerinden biridir.

---

### Resource Hierarchy

**Tanım:** IAM politikalarının aşağı doğru miras alındığı Google Cloud'un dört katmanlı yapısı — Organization Node, Folder, Project, Resource.

**Neden var:** Bir hiyerarşi olmadan, büyük bir kuruluşta erişim politikalarını uygulamak ve denetlemek, aynı yapılandırmayı her yerde tekrarlamayı gerektirirdi.

**İlgili servisler:** Organization Node, Folder, Project, IAM.

**Yaygın yanlış anlamalar:** Politikalar **aşağı doğru** akar, yukarı doğru değil — bir klasöre uygulanan bir politika, altındaki her proje ve kaynağa otomatik olarak uygulanır, ama tersi geçerli değildir.

**Gerçek dünya benzetmesi:** Kaynak hiyerarşisi bir şirketin organizasyon şemasıdır — genel merkezde belirlenen şirket çapında bir politika, otomatik olarak her departmana ve çalışana kadar iner.

---

### Secret Manager

**Tanım:** Hassas veriyi — API anahtarları, parolalar, sertifikalar — ikili blob'lar ya da metin dizeleri olarak güvenle saklamak, versiyonlamak ve erişimi kontrol etmek için bir servis.

**Neden var:** Sırları düz dosyalarda ya da ortam yapılandırmalarında saklamak, onları sızdırmayı kolaylaştırır, merkezi olarak denetlemeyi ve döndürmeyi zorlaştırır.

**İlgili servisler:** IAM, Least Privilege, Cloud KMS.

**Yaygın yanlış anlamalar:** Bir sırrın **adı** küreseldir, ama **verisi** isteğe bağlı olarak bölgesel olarak saklanabilir. Sır versiyonları **değişmezdir ama silinebilir** — bir versiyonu asla yerinde düzenlemezsin, yenisini oluşturursun. Varsayılan olarak yalnızca proje **sahipleri** sırlara erişebilir; başka herkes açık bir IAM izni gerektirir.

**Gerçek dünya benzetmesi:** Secret Manager, bir banka kasa sistemi gibidir — her erişim kaydedilir, sadece yetkili kişiler bir anahtar alır ve içindekini asla fiziksel olarak değiştirmezsin, zamanla sadece yeni öğeler eklersin.

---

### Serverless

**Tanım:** Geliştiricinin hiç altyapı yönetmediği, sadece kodla ilgilendiği, faturalandırmanın genellikle milisaniyeye kadar gerçek kullanıma dayandığı bir bulut servis modeli.

**Neden var:** Soyutlama merdiveninin (IaaS → PaaS → Serverless → SaaS) son adımıdır — ne kadar az şey yönetirsen, uygulamana o kadar odaklanabilirsin.

**İlgili servisler:** Cloud Run, Cloud Run Functions, Platform as a Service (PaaS).

**Yaygın yanlış anlamalar:** "Serverless", sunucu olmadığı anlamına gelmez — onlar hakkında hiç düşünmen gerekmediği anlamına gelir.

**Gerçek dünya benzetmesi:** Serverless bir taksi yolculuğudur — arabaya sahip olmaz, bakımını yapmaz ya da park etmezsin; sadece fiilen kat ettiğin mesafe için ödersin.

---

### Serverless VPC Access

**Tanım:** Cloud Run functions'ı (ve diğer serverless ürünlerini) doğrudan bir VPC network'e bağlayan, böylece Compute Engine VM instance'ları ve Memorystore gibi yalnızca internal IP address'i olan kaynaklara internal DNS ve internal IP address'ler üzerinden erişilmesini sağlayan bir mekanizma.

**Neden var:** Serverless ürünler varsayılan olarak VPC network'ünüzün dışında çalışır; Serverless VPC Access olmadan, yalnızca internal IP'si olan bir backend'e erişmenin tek yolu onu public olarak açığa çıkarmak olurdu, ki bu da onu internal tutmanın amacını ortadan kaldırır.

**İlgili servisler:** Cloud Run Functions, VPC (Virtual Private Cloud), VPC Peering and Shared VPC, Compute Engine, Memorystore.

**Yaygın yanlış anlamalar:** Bir connector, oluşturulur oluşturulmaz otomatik olarak kullanılabilir hale gelmez — ona ihtiyaç duyan her function'ın onu kullanacak şekilde ayrı ayrı yapılandırılması gerekir, ve connector'ın region'ı function'ın deployment region'ıyla eşleşmelidir, aksi halde bağlantı sessizce başarısız olur. Bir connector'ın ayrıca yalnızca kendi kullanımına ayrılmış bir subnet'e ya da CIDR range'e ihtiyacı vardır.

**Gerçek dünya benzetmesi:** Serverless VPC Access, kontrol etmediğiniz bir bina (serverless ortam) ile kontrol ettiğiniz güvenli bir bina (VPC network'ünüz) arasında inşa edilmiş, özel, ayrılmış bir koridordur — trafik, sokağa çıkıp yan kapıdan tekrar girmek yerine bu koridordan yürür.

---

### Service (Kubernetes)

**Tanım:** Bir grup Pod'a kararlı bir ağ uç noktası (sabit IP) veren Kubernetes soyutlaması; böylece istemcilerin tek tek Pod'ların değişen IP'lerini takip etmesi gerekmez.

**Neden var:** Pod'lar geçicidir ve IP'leri değişir; diğer bileşenlerin onlara güvenilir biçimde ulaşabilmesi için kalıcı bir şeye ihtiyaç vardır.

**İlgili servisler:** Pod, Deployment (Kubernetes), GKE.

**Yaygın yanlış anlamalar:** Bir Service, harici bir load balancer'la aynı şey değildir; ancak GKE, bir Service'in harici erişime ihtiyacı olduğunda genellikle otomatik olarak bir tane (bir Network Load Balancer) provizyonlar.

**Gerçek dünya benzetmesi:** Bir Service, bir şirketin tek bir yayınlanmış telefon numarası gibidir — cevap veren kişi vardiyadan vardiyaya değişebilir, ama arayanların hatırlaması gereken tek şey bir numaradır.

---

### Service Account

**Tanım:** Bir kişiyi değil bir uygulamayı ya da workload'ı temsil eden, parola yerine bir RSA anahtar çifti kullanılarak doğrulanan ve bir kaynak olarak kendisine de IAM rolleri uygulanabilen bir kimlik.

**Neden var:** Gözetimsiz çalışan bir programın (örn. Cloud Storage'a yazan bir VM) her seferinde bir insanın erişimi manuel onaylamasına gerek kalmadan kendi kimliğine ihtiyacı vardır.

**İlgili servisler:** IAM, Application Default Credentials, Workload Identity.

**Yaygın yanlış anlamalar:** İndirilmiş servis hesabı anahtarları önemli bir risktir (kimlik bilgisi sızıntısı, ayrıcalık yükseltme, kimlik gizleme) ve **son çare** sayılır — attached Service Account, Workload Identity ya da Workload Identity Federation, tercih edilen, anahtarsız alternatiflerdir.

**Gerçek dünya benzetmesi:** Bir servis hesabı, robot bir çalışandır — kendi rozetine ve kendi sınırlı kapı erişimine sahiptir ve asla hasta izni almaz ya da bir insanın onu kapıdan geçirmesine ihtiyaç duymaz.

---

### Service Choreography and Service Orchestration

**Tanım:** Microservices'i koordine etmek için iki desen. **Choreography**'de her servis bağımsız çalışır, merkezi bir source of truth olmadan event'lere tepki verir. **Orchestration**'da merkezi bir orchestrator, servisler arasındaki tüm etkileşimleri kontrol eder.

**Neden var:** Microservices arasındaki iletişimi koordine etmek, bir microservices mimarisinin en zor kısımlarından biridir — bir zamanlar bir monolith'in içinde yaşayan karmaşıklığın bir kısmı, servislerin birbiriyle nasıl konuştuğuna kayar, ve bunlar bunu yönetmenin iki temel yoludur.

**İlgili servisler:** Event-Driven Architecture, Workflows, Eventarc.

**Yaygın yanlış anlamalar:** Hiçbir desen kesin olarak "daha güvenli" bir varsayılan değildir. Orchestration, sürecin üst düzey bir görünümünü ve daha kolay troubleshooting'i verir, ama orchestrator bir single point of failure'dır. Choreography bu tek arıza noktasından kaçınır ve decentralized takımlara/organizasyonlara iyi uyar, ama merkezi bir source of truth'u yoktur, bu da genel akışı anlamayı zorlaştırır ve visibility, error handling ve retry'ları doğru yapmayı zorlaştırır.

**Gerçek dünya benzetmesi:** Choreography, koreografi yapılmış bir dans gibidir — her dansçı kendi bölümünü bilir ve performans sırasında kimse aktif olarak yönlendirmeden bağımsız olarak icra eder. Orchestration ise bir orkestra gibidir — bir şef, her müzisyeni gerçek zamanlı olarak aktif olarak senkronize eder.

---

### Service-Oriented Architecture (SOA)

**Tanım:** Bir uygulamayı, her biri ayrı bir iş fonksiyonunu yürüten, merkezi bir Enterprise Service Bus (ESB) üzerinden routing yapılan messaging ile tanımlı arayüzler üzerinden iletişim kuran, yeniden kullanılabilir servislere ayrıştıran bir mimari stil.

**Neden var:** Monolithleri düzeltme girişimiydi — büyük, sıkı bağlı bir kod tabanını, daha küçük takımların sahip olabileceği daha küçük, daha gevşek bağlı, yeniden kullanılabilir servislere bölmek.

**İlgili servisler:** Enterprise Service Bus (ESB), Monolith (Monolithic Application), Microservices.

**Yaygın yanlış anlamalar:** SOA'nın entegrasyon karmaşıklığını kesin olarak çözdüğü sıkça varsayılır — pratikte genelde karışık sonuçlar üretti: servisler daha küçük ve daha gevşek bağlı hale geldi, ama onları bağlamanın karmaşıklığı kaybolmadı, merkezi bir takımın sahip olduğu ESB entegrasyonlarına taşındı ve bu yeni darboğaz haline geldi (microservices'in daha sonra paylaşılan ESB'yi tamamen kaldırmasının nedeni de budur).

**Gerçek dünya benzetmesi:** SOA, bir şirketteki birkaç bağımsız departmanın yalnızca tek bir merkezi posta odası üzerinden iletişim kurması gibidir — departmanların kendisi kendi başına gayet iyi çalışır, ama artık departmanlar arası her istek o tek posta odasının kapasitesine bağımlıdır.

---

### Software as a Service (SaaS)

**Tanım:** İnternet üzerinden tam, kullanıma hazır bir uygulama sunan bir bulut servis modeli — hiçbir şey kurmadan ya da yönetmeden doğrudan tüketirsin (örn. Gmail, Docs, Drive).

**Neden var:** Soyutlamanın en yüksek düzeyini temsil eder — sıfır altyapı ve sıfır uygulama yönetimi, sadece kullanım.

**İlgili servisler:** Platform as a Service (PaaS), Serverless.

**Yaygın yanlış anlamalar:** SaaS, IaaS/PaaS/Serverless'tan tamamen farklı bir eksendedir — "serverless'tan daha serverless" değildir; üzerine inşa ettiğin bir şey değil, doğrudan tükettiğin bitmiş bir üründür.

**Gerçek dünya benzetmesi:** SaaS, kendi taksini çağırmak yerine bir taksi *uygulamasını* kullanmak gibidir — arayüz dahil tüm deneyim sana hazır teslim edilir.

---

### Software Delivery Shield

**Tanım:** CI/CD hattı boyunca uçtan uca yazılım tedarik zinciri güvenliği için Google Cloud'un şemsiye terimi; Assured OSS, Cloud Build'in doğrulanabilir metadata'sı, Artifact Analysis, Cloud Deploy, Binary Authorization ve GKE/Cloud Run'ın çalışma zamanı güvenlik panellerini birleştirir.

**Neden var:** Bir CI/CD hattının birçok bileşeni vardır (kaynak, bağımlılıklar, build altyapısı, imaj deposu, dağıtım adımı) ve her biri ayrı bir saldırı yüzeyidir — Software Delivery Shield hepsini parça parça değil bir arada ele alır.

**İlgili servisler:** Assured OSS, Cloud Build, Artifact Analysis, Binary Authorization, Cloud Deploy.

**Yaygın yanlış anlamalar:** Software Delivery Shield tek başına bağımsız bir araç değildir — birkaç ayrı servisin birlikte nasıl çalıştığını tanımlayan bir şemsiye kavramdır.

**Gerçek dünya benzetmesi:** Software Delivery Shield, girişte ham maddeleri denetleyen, üretim kayıtlarını izleyen, depoda bitmiş ürünleri tarayan ve her şey sevk edilmeden önce kimlik bilgilerini doğrulayan, bir fabrikanın uçtan uca kalite programıdır.

---

### Spanner

**Tanım:** %99,999'a varan kullanılabilirlik sunan, tam yönetilen, yatay ölçeklenen, güçlü tutarlı bir ilişkisel veritabanı; kritik görev, küresel OLTP için tasarlanmıştır.

**Neden var:** Geleneksel olarak bir ilişkisel veritabanında ya güçlü tutarlılığa *ya da* yatay ölçeğe sahip olabilirdin, ikisine birden değil — Spanner bu ödünleşimi kırdı.

**İlgili servisler:** Cloud SQL, AlloyDB, Consistency (Strong vs. Eventual).

**Yaygın yanlış anlamalar:** Spanner, küçük, tek bölgeli bir uygulama için aşırı mühendisliktir — gerçekten küresel ölçeğe ve güçlü tutarlılığa birlikte ihtiyaç duymuyorsan Cloud SQL daha iyi ve daha ucuz bir seçimdir.

**Gerçek dünya benzetmesi:** Spanner, dünya çapındaki her şubeye anında ve özdeş biçimde kopyalanan bir banka defteridir — hangi şubeye baksan bakiye her zaman tam olarak doğru ve günceldir.

---

### Speech-to-Text and Text-to-Speech

**Tanım:** Sesi metne ve metni sese çeviren, 110 dil ve varyantı destekleyen bir çift önceden eğitilmiş API.

**Neden var:** Sesli arayüzler, dikte ya da yazıya dökme özelliklerini sıfırdan inşa etmek, çoğu ekibin sahip olmadığı derin bir konuşma tanıma uzmanlığı gerektirir.

**İlgili servisler:** Natural Language AI, Translation AI.

**Yaygın yanlış anlamalar:** Birbirlerini tekrarlamazlar, tamamlarlar — ikisini birleştirmek, uygulamanın dinlediği, anladığı ve geri konuştuğu tam bir sesli arayüz kurmanı sağlar.

**Gerçek dünya benzetmesi:** Speech-to-Text ve Text-to-Speech, bir çift kapı gibidir — biri sesi kelime olarak içeri alır, diğeri kelimeleri ses olarak dışarı verir.

---

### Strangler Pattern

**Tanım:** Bir eski (legacy) uygulamanın küçük parçalarının aşamalı olarak yeni servislerle değiştirildiği, bir cephenin (facade) istekleri eski sisteme ya da yenilere yönlendirdiği, legacy sistem tamamen değiştirilene kadar süren bir göç stratejisi.

**Neden var:** Büyük bir legacy sistemin "big bang" yeniden yazımı yüksek risklidir; aşamalı değiştirme, ilerledikçe öğrenmene ve riski azaltmana izin verir.

**İlgili servisler:** Continuous Integration, Delivery, and Deployment (CI/CD).

**Yaygın yanlış anlamalar:** Bu desen, zamanla bir ev sahibi ağacı yavaşça saran boğucu asma bitkilerinden adını alır — isim, aniden bir değiştirmeyi değil, kademeli devralmayı tarif eder.

**Gerçek dünya benzetmesi:** Strangler deseni, bir evi yıkıp yeniden başlamak yerine, hâlâ içinde yaşarken oda oda tadilat yapmak gibidir.

---

### Structured Logging

**Tanım:** Düz metin (`textPayload`) yerine, `severity` (log seviyesi) ve `message` (ana görüntü metni) gibi özel alanlarla, tek satırlık serileştirilmiş bir JSON nesnesini (bir log girdisinin `jsonPayload` alanında saklanan) loglamak.

**Neden var:** Düz metin logların bir log seviyesi yoktur ve aranması zordur; yapılandırılmış alanlar, logları büyük ölçekte filtrelenebilir ve sorgulanabilir hale getirir.

**İlgili servisler:** Cloud Logging, Log-Based Alert and Log-Based Metric.

**Yaygın yanlış anlamalar:** Düz metin loglama yanlış değildir, sadece sınırlıdır — hızlı/basit durumlar için işe yarar, ama yapılandırılmış (JSON) loglama, üretim log analizi için genel olarak önerilen yaklaşımdır.

**Gerçek dünya benzetmesi:** Düz metin bir log, bir şey bulmak için baştan sona okuman gereken bir günlük girdisidir. Yapılandırılmış bir log, anında arayıp sıralayabileceğin, etiketli alanları olan doldurulmuş bir formdur.

---

### Translation AI

**Tanım:** Desteklenen diller arasında keyfi metni gerçek zamanlı çeviren, önceden eğitilmiş, son derece hızlı yanıt veren bir API.

**Neden var:** Web siteleri ve uygulamalar genellikle statik, önceden hazırlanmış çeviri dosyaları yerine dinamik, anlık çeviriye ihtiyaç duyar.

**İlgili servisler:** Natural Language AI, Speech-to-Text and Text-to-Speech.

**Yaygın yanlış anlamalar:** "Dinamik" burada anahtar kelimedir — bu, sabit bir sayfa setini önceden çevirmekle ilgili değildir, içeriği kullanıcı istediği an çevirmekle ilgilidir.

**Gerçek dünya benzetmesi:** Translation AI, önceden danıştığın bir konuşma kılavuzu yerine yanında duran, gerçek zamanlı çeviri yapan canlı bir tercümandır.

---

### Trigger (Cloud Run Functions)

**Tanım:** Deployment anında belirtilen, bir Cloud Run function'ının nasıl ve ne zaman invoke edileceğini belirleyen yapılandırma — ya bir HTTP trigger (HTTP(S) isteklerine tepki verir) ya da bir event trigger (Pub/Sub, Cloud Storage, Firestore ya da Firebase gibi desteklenen bir kaynaktan gelen, hepsi Eventarc aracılığıyla teslim edilen bir event'e tepki verir).

**Neden var:** Bir function'ın ne zaman çalışması gerektiğini bilmesinin iyi tanımlanmış bir yola ihtiyacı vardır; trigger'lar, "function'ı ne invoke eder" sorusunu function'ın kendi kodundan ayırır, böylece aynı function mantığı prensipte farklı invocation kaynaklarına bağlanabilir.

**İlgili servisler:** Eventarc, CloudEvents, Cloud Run Functions, Pub/Sub, Firestore.

**Yaygın yanlış anlamalar:** Tek bir function aynı anda yalnızca bir trigger'a bağlanabilir — ona birden fazla trigger bağlayamazsınız. Tek bir event'i birkaç function'a fan-out etmek farklı şekilde başarılır: aynı trigger source ayarlarını paylaşan birden fazla function deploy ederek, bir function'a birkaç trigger vererek değil.

**Gerçek dünya benzetmesi:** Bir trigger, tek bir spesifik düğmeye bağlı tek bir kapı zilidir — aynı düğmeye birkaç kapı zili bağlayabilirsiniz (bir trigger source, birden fazla function), ama tek bir kapı zili aynı anda iki farklı düğmeden çalacak şekilde bağlanamaz (bir function, bir trigger).

---

### Video AI

**Tanım:** Saklanan videodaki varlıkları çekim, kare ya da tüm video düzeyinde tespit eden ve etiketleyen, ne zaman göründüklerini de içeren, önceden eğitilmiş bir API (Video Intelligence API).

**Neden var:** Vision AI tek bir sabit görüntüyü analiz eder; bazı problemler (örn. "bu görüntüde arabanın göründüğü her anı bul") zaman boyutunun eklenmesini gerektirir.

**İlgili servisler:** Vision AI, Document AI.

**Yaygın yanlış anlamalar:** Video AI, sadece Vision AI'ın kareler üzerinde tekrar tekrar çalıştırılması değildir — özellikle *zamanlama* bilgisi döndürür (bir varlığın ne zaman göründüğü), ki eklenen karmaşıklığın tüm amacı budur.

**Gerçek dünya benzetmesi:** Video AI, sana sadece "bu kitap köpeklerden bahsediyor" değil, köpeklerin bahsedildiği tam sayfayı ve paragrafı söyleyebilen bir kütüphaneci gibidir.

---

### Vision AI

**Tanım:** Etiketleme, OCR, nirengi noktası/logo tespiti, yüz tespiti (duyguyla birlikte) ve müstehcen içerik tespiti sunan, görüntü tespiti için önceden eğitilmiş bir API.

**Neden var:** Sıfırdan bir görüntü tanıma modeli eğitmek devasa etiketli veri kümeleri ve compute gerektirir; Vision AI sana Google'ın önceden eğitilmiş yeteneğini basit bir API çağrısıyla verir.

**İlgili servisler:** Video AI, Document AI, AutoML.

**Yaygın yanlış anlamalar:** Vision AI'ın anlayışı şaşırtıcı derecede ayrıntılı olabilir — örneğin, Las Vegas Sfenks replikasını gerçek Mısır Sfenksi'nden ayırt edebilir, sadece "bir sfenks" tanımakla kalmaz.

**Gerçek dünya benzetmesi:** Vision AI, bir fotoğrafı elden geçirip her yüzün ifadesini, her nirengi noktasının adını ve görüntüdeki her yazılı kelimeyi anında söyleyen bir uzmana fotoğraf teslim etmek gibidir.

---

### VPC (Virtual Private Cloud)

**Tanım:** Genel bir bulut içinde (Google Cloud) barındırılan, güvenli, izole, özel bir bulut bilişim ortamı; Google Cloud'daki VPC ağları **küreseldir**, bir bölgenin zonlarını kapsayabilen bölgesel alt ağlarla.

**Neden var:** Kuruluşların, genel bir bulutun ölçeklenebilirliğinden ve kolaylığından vazgeçmeden özel bulut tarzı izolasyona ihtiyacı vardır.

**İlgili servisler:** Subnet, Cloud Load Balancing, VPC Peering and Shared VPC, Hybrid Connectivity.

**Yaygın yanlış anlamalar:** Yeni kullanıcılar, VPC ağlarının **küresel** olmasına genellikle şaşırır — aynı alt ağdaki kaynaklar farklı zonlarda (hatta bir bölgeyi kapsayan tek bir küresel ağın alt ağları içinde farklı bölgelerin zonları bile alışılmadık değildir) durabilir ve yine de "komşu" sayılabilir.

**Gerçek dünya benzetmesi:** Bir VPC bir şehirdir; alt ağlar onun mahalleleridir; güvenlik duvarı kuralları her mahalleye giren ve çıkanları kontrol eden güvenlik kapılarıdır.

---

### VPC Peering and Shared VPC

**Tanım:** **VPC Peering**, iki ayrı VPC ağını trafik değişebilecek şekilde birbirine bağlar. **Shared VPC**, bir projenin VPC'sinin başka projelerdeki kaynaklar tarafından kullanılmasına izin verir, tam olarak IAM'in kontrol ettiği şekilde.

**Neden var:** Büyük kuruluşlar sık sık işi birden çok proje arasında böler ama yine de o projelerin kaynaklarının kontrollü bir ağ sınırı üzerinden birbiriyle konuşması gerekir.

**İlgili servisler:** VPC, IAM, Project.

**Yaygın yanlış anlamalar:** Bunlar farklı sorunları çözer — Peering iki bağımsız ağı eşit olarak bağlar; Shared VPC, başka projelerin ödünç aldığı tek bir ağı, IAM kontrolü altında merkezileştirir.

**Gerçek dünya benzetmesi:** VPC Peering, iki komşu apartmanın sakinlerinin birbirinin avlusunu kullanmasına izin vermeyi kabul etmesi gibidir. Shared VPC, birkaç binanın hepsinin tek bir paylaşılan bina yöneticisinin altyapısına bağlanması gibidir.

---

### Workflows

**Tanım:** GCP servislerini ve API çağrılarını stateful, otomatikleştirilmiş süreçlere orchestrate eden workflow'lar tasarlamak ve deploy etmek için Google Cloud'un tamamen yönetilen orchestration platformu.

**Neden var:** Service-orchestration deseninin somut implementasyonudur — bir iş süreci için merkezi, observable bir source of truth, state tutabilen, retry yapabilen, poll edebilen ya da bir yıla kadar bekleyebilen, bu da gerçekten uzun süren süreçleri pratik hale getirir.

**İlgili servisler:** Service Choreography and Service Orchestration, Eventarc, Cloud Tasks.

**Yaygın yanlış anlamalar:** Workflows ve Eventarc, aynı iş için deploy edilen rakipler değildir — pratikte genelde birleştirilirler: Workflows, bir servisin kendi süreci **içinde** orchestration'ı yönetir, Eventarc ise bağımsız olarak orchestrate edilen servisler **arasında** event trigger'ları taşır (sınırlar arasında choreography).

**Gerçek dünya benzetmesi:** Workflows, çok adımlı bir sürecin ana kontrol listesini elinde tutan, işlerin tam olarak nerede durduğunu izleyen, başarısız olan bir adımı yeniden deneyen, ve günlerce duraklayıp bekleyebilen ama yerini hiç kaybetmeyen bir proje yöneticisi gibidir.

---

### Workload Identity and Workload Identity Federation

**Tanım:** **Workload Identity**, bir GKE cluster'ı içindeki bir Kubernetes servis hesabının, Google Cloud API'lerini çağırırken bir IAM servis hesabını taklit etmesini (impersonate) sağlar. **Workload Identity Federation**, Google Cloud'un **dışında** çalışan workload'lar (başka bir bulut, on-premises) için eşdeğerini yapar; bir OIDC token'ını kısa ömürlü bir Google Cloud erişim token'ıyla değiştirir — ikisi de indirilebilir bir servis hesabı anahtarı ihtiyacını ortadan kaldırır.

**Neden var:** İndirilmiş servis hesabı anahtarları bir güvenlik yükümlülüğüdür; bu mekanizmalar, harici ve cluster içi workload'ların uzun ömürlü bir anahtarla hiç uğraşmadan kimlik doğrulamasını sağlar.

**İlgili servisler:** Service Account, GKE, Application Default Credentials.

**Yaygın yanlış anlamalar:** Neredeyse özdeş isimler sık karışıklığa yol açar. **Workload Identity**, bir GKE cluster'ının **içindeki** workload'lar içindir; **Workload Identity Federation**, OIDC uyumlu bir kimlik sağlayıcı kullanarak Google Cloud'un **tamamen dışındaki** workload'lar (başka bir bulut ya da on-premises) içindir.

**Gerçek dünya benzetmesi:** Workload Identity, şirket merkezinin içinde çalışan bir çalışan rozetidir. Workload Identity Federation, güvenilen bir ortak şirketin çalışanına verilen, ona hiçbir zaman kalıcı bir şirket rozeti vermeden tanınan bir ziyaretçi kartıdır.

---

### Zone

**Tanım:** Google Cloud kaynaklarının fiilen dağıtıldığı belirli konum; bir bölge birden çok zondan oluşur.

**Neden var:** Kaynakları aynı bölge içinde birden çok zona yaymak, uzak bölgelere yayılmanın getirdiği ek gecikme olmadan tek bir zonun çökmesine karşı korur.

**İlgili servisler:** Region, Multi-Region.

**Yaygın yanlış anlamalar:** Bir zon, bir bölgeyle yer değiştirilebilir değildir — zon, bir VM gibi bir kaynağın fiilen çalıştığı, bir bölgenin *içindeki* belirli, daha dar konumdur.

**Gerçek dünya benzetmesi:** Bir bölge bir şehirse, bir zon onun mahallelerinden biridir — şehirdeki diğer zonlara düşük gecikme için yeterince yakın, ama yerel bir kesintinin tüm şehri devre dışı bırakmaması için yeterince ayrıdır.
