# Modül 20 — Service Identity and Authentication: Pratik Sorular

Bu set, service account'ları ve identity'yi (IAM'in API çağrılarını nasıl yetkilendirdiği, bir policy binding'in yapısı, üç identity türü, bir service account'ı bir user account'tan ayıran şey, built-in ve user-managed service account'lar, Cloud Run'ın default service identity'si, Google API çağrıları için OAuth 2.0 access token'ları, asenkron ve senkron service-to-service iletişim, Cloud Run Invoker role'ü, ve doğrudan servis çağrıları için OpenID Connect ID token'ları), resource hierarchy'yi (Organization/Folder/Project, kaynak başına IAM policy'ler, policy binding inheritance, ve effective IAM policy), least privilege prensibini (üç IAM role türü, Cloud Run'ın default service account'unun neden bir güvenlik riski olduğu, ve üç adımlı azaltma), ve secret'lar ile environment variable'ları (environment variable'ların nasıl çalıştığı, ve Secret Manager secret'larının nasıl farklı olduğu ve nasıl erişildiği) kapsar.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: yeni oluşturulan bir service account'ın neden bir tür varsayılan erişimle değil sıfır izinle başladığı, resource hierarchy'de daha yüksek bir seviyede verilen izinlerin neden daha düşük bir seviyede hiçbir zaman geri alınamadığı, default Cloud Run service account'unun neden sadece bir kolaylık takası değil gerçek bir güvenlik riski olduğu, OAuth 2.0 access token'larının ve OpenID Connect ID token'larının neden farklı amaçlara hizmet ettiği, ve bir volume olarak mount edilen bir secret'ın bir environment variable olarak geçirilenden neden farklı davrandığı.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Uygulama kodu Pub/Sub gibi bir Google Cloud API'sini çağırdığında, IAM çağrıya izin verip vermeyeceğine nasıl karar verir?

A. IAM, herhangi bir Google Cloud kaynağından gelen her çağrıya koşulsuz olarak izin verir, çünkü yetkilendirme yalnızca bir browser üzerinden giriş yapan insan kullanıcılar için geçerlidir.
B. IAM, isteği inceler ve API isteğindeki credential'lardan çağıranı tanımlar, ardından hedef kaynağa (örn. Pub/Sub topic'i) attach edilmiş IAM policy'de gereken role'e sahip bir policy binding olup olmadığını kontrol eder — identity yetkili değilse çağrıyı reddeder.
C. IAM yalnızca çağıranın IP adresini ve coğrafi konumunu kontrol eder, herhangi bir türde policy binding ya da role dahil olmadan.
D. IAM, tüm yetkilendirme kararlarını, hiçbir bağımsız doğrulama yapmadan, tamamen çağıran uygulamanın kendi koduna devreder.

**2.** Tek bir IAM policy binding'i gerçekte neyden oluşur, ve bir member birden fazla role'e sahip olabilir mi?

A. Bir policy binding, hiçbir role dahil olmadan bir member'ı ve bir kaynağı doğrudan birbirine bağlar, ve bir member hiçbir zaman birden fazla kaynakla ilişkilendirilemez.
B. Bir policy binding, herhangi bir member ya da role ile ilgisiz olan, bir API çağrısından sonra süresi dolan, geçici, tek kullanımlık bir yetkilendirme token'ıdır.
C. Bir policy binding, bir member'ın önceki tüm izinlerini kalıcı olarak siler, ve bir member tüm bir IAM policy genelinde hiçbir zaman tam olarak bir role'den fazlasına sahip olamaz.
D. Bir policy binding, bir ya da daha fazla member'ı (identity) tek bir role'e bağlar, burada role, member'ın bir kaynak üzerinde belirli aksiyonlar gerçekleştirmesini sağlayan bir izinler kümesi içerir; bir member, bir IAM policy içinde birden fazla policy binding'e attach edilebilir, bu da ona birden fazla role verir.

**3.** IAM'in policy binding member'ları olarak desteklediği üç identity türü nedir?

A. Human identity'ler (bir grubun ya da domain'in parçası olabilen bir Google hesabı), service account'lar (bir VM, Cloud Run servisi, ya da Cloud Run function'ı gibi makineler, uygulamalar, ya da servisler tarafından kullanılır), ve "all users" (public/anonim erişim için özel bir tanımlayıcı).
B. IAM tarafından yalnızca human identity'ler desteklenir; service account'lar ve public erişim tanımlayıcıları modelde ayrı kavramlar olarak mevcut değildir.
C. Root identity'ler, admin identity'ler, ve guest identity'ler — IAM'in gerçekte kullandığından tamamen farklı, üçlü bir sınıflandırma.
D. Faturalandırma identity'leri, network identity'leri, ve storage identity'leri, kimi/neyi temsil ettiklerine göre değil, hangi Google Cloud ürünüyle kullanıldıklarına göre kategorize edilir.

**4.** Bir service account, sıradan bir user account'tan nasıl farklıdır?

A. Bir service account, tıpkı bir user account gibi bir şifreye sahiptir, ve tam olarak aynı şekilde cookie'ler kullanarak bir browser üzerinden giriş yapabilir.
B. Bir service account, tıpkı bir user account gibi otomatik olarak Google Workspace domain'inin üyesidir, ikisi arasında hiçbir ayrım yoktur.
C. Bir service account, benzersiz bir e-posta adresiyle tanımlanan, makineler, uygulamalar, ya da servisler tarafından kullanılan özel bir hesap türüdür; user account'lardan farklı olarak, şifresi yoktur ve bir browser ya da cookie'ler aracılığıyla giriş yapamaz, diğer kullanıcıların ya da service account'ların onun adına hareket etmesine izin verilebilir, ve Google Workspace domain'inin üyesi değildir (yine de gruplara eklenebilir).
D. Bir service account, her zaman öyle olabilen bir user account'ın aksine, hiçbir koşulda hiçbir grubun parçası olamaz.

**5.** Bir sanal makinede, bir Cloud Run servisinde, ya da bir Cloud Build build'inin parçası olarak kod çalıştırırsan, client kütüphaneleri varsayılan olarak hangi service account'u kullanır, ve bunun yerine ne önerilir?

A. Client kütüphaneleri hiçbir koşulda hiçbir service account'u otomatik olarak kullanmaz, her tek API çağrısının geliştirici tarafından elle kimlik doğrulanmasını gerektirir.
B. Built-in service account hiçbir koşulda değiştirilemez, bu da user-managed bir service account'u mimari olarak yapılandırılamaz kılar.
C. Client kütüphaneleri, herhangi bir türde kimlik doğrulama gerçekleşmeden önce, elle oluşturulup yapılandırılması gereken yepyeni bir service account gerektirir, hiçbir zaman bir varsayılan sağlanmaz.
D. Google Cloud API'lerine bağlanırken client kütüphanelerinin kimlik doğrulama için otomatik olarak kullandığı built-in bir service account'a her zaman erişimin vardır, ama bunu kendi user-managed service account'unla değiştirmen önerilir.

**6.** Varsayılan olarak, birini belirtmezsen bir Cloud Run servisi ya da job'ı hangi service account olarak çalışır, ve bu hesap başka ne olarak da bilinir?

A. Her Cloud Run servisi ya da job'ı, "service identity" olarak bilinen bir service account'a bağlıdır — varsayılan olarak bu, Editor role'üne sahip default Compute Engine service account'udur.
B. Varsayılan olarak, bir Cloud Run servisine ya da job'ına hiçbir service account bağlı değildir, bu da ondan herhangi bir Google Cloud API çağrısını configuration'dan bağımsız olarak imkansız kılar.
C. Varsayılan olarak, Cloud Run servisleri ve job'ları, sıfır izne sahip ve değiştirilemeyen özel bir "yalnızca Cloud Run" service account'u olarak çalışır.
D. Varsayılan olarak, Cloud Run servisleri ve job'ları, hiçbir service account olmadan, onları en son deploy eden hangi insan kullanıcı ise onun credential'larını kullanarak çalışır.

**7.** Bir Cloud Run servisindeki uygulama kodu, bir Google Cloud API'sini çağırmak için bir client kütüphanesi kullandığında, client kütüphanesi ne tür bir token elde eder, ve kimlik doğrulamak için neyi kullanır?

A. Client kütüphanesi, herhangi bir API çağrısı başarılı olmadan önce, son kullanıcının kişisel şifresinin doğrudan uygulama kaynak koduna sabit kodlanmasını gerektirir.
B. Client kütüphanesi, hangi API çağrıldığından bağımsız olarak, bir Cloud Run container'ının içinden gelen herhangi bir çağrı için IAM'i tamamen atlar.
C. Client kütüphanesi, servisin runtime service account'unu kullanarak kimlik doğrular ve çoğu Google API'si için, çağrıyı yapmak üzere otomatik olarak bir OAuth 2.0 access token'ı elde eder — IAM daha sonra bunu hedef kaynak üzerindeki policy binding'e karşı doğrular.
D. Client kütüphanesi, bir kez üretilen ve hiçbir zaman süresi dolmayan ya da IAM tarafından başka bir doğrulama gerektirmeyen, kalıcı, değişmeyen bir API anahtarı kullanır.

**8.** Uygulama mimarin, birbiriyle iletişim kurması gereken birden fazla Cloud Run servisi kullanıyorsa, modül asenkron ve senkron iletişim için hangi mekanizmaları tanımlıyor?

A. Hem asenkron hem senkron iletişimin tam olarak aynı mekanizmayı gerektirdiği belirtiliyor, modülde ikisi arasında hiçbir yerde bir ayrım yapılmıyor.
B. Asenkron iletişim için, Cloud Tasks, Pub/Sub, Cloud Scheduler, ya da Eventarc gibi çeşitli Google Cloud servisleri kullanılabilir; senkron iletişim için, bir servis başka bir servisin endpoint URL'ini doğrudan HTTP üzerinden çağırır, çağıran servis için IAM ve bireysel bir service identity kullanmak best practice olarak öneriliyor.
C. Cloud Run servisleri arasında asenkron iletişimin hiçbir koşulda imkansız olduğu, senkron HTTP çağrılarını mevcut tek seçenek olarak bıraktığı belirtiliyor.
D. Senkron iletişimin yalnızca Cloud Tasks gerektirdiği, asenkron iletişimin ise alıcı servisin endpoint'ine doğrudan bir HTTP çağrısı gerektirdiği belirtiliyor.

**9.** Çağıran bir Cloud Run servisinin, private bir alıcı Cloud Run servisini senkron olarak çağırabilmesi için, modül hangi spesifik kurulumu tanımlıyor?

A. Alıcı servisi, çağıran servisin service account'unu alıcı servis üzerinde bir principal yaparak, çağıran servisten gelen istekleri kabul edecek şekilde yapılandırırsın, ve o service account'a Cloud Run Invoker (`roles/run.invoker`) role'ünü verirsin — örn. `gcloud run services add-iam-policy-binding RECEIVING_SERVICE --member='serviceAccount:CALLING_SERVICE_IDENTITY' --role='roles/run.invoker'` ile.
B. Her iki serviste de IAM'i tamamen devre dışı bırakırsın, çünkü private-to-private Cloud Run iletişiminin hiçbir kimlik doğrulama mekanizmasını desteklemediği belirtiliyor.
C. Çağıran servisin service account'una tüm Google Cloud projesi üzerinde Owner role'ünü verirsin, çünkü bu amaç için daha hedeflenmiş bir role mevcut değildir.
D. Her iki servisi de, herhangi bir noktada ayrı role'ler ya da policy binding'ler dahil olmadan, tek, özdeş bir service account'u paylaşacak şekilde yapılandırırsın.

**10.** Senkron service-to-service çağrıları için, çağıran servisten gelen isteğin kimlik kanıtı olarak ne sunması gerekir, ve bu OpenID Connect ile nasıl ilişkilidir?

A. İstek, OpenID Connect'in süreçte hiçbir rolü olmadan, çağıran servisin ham, şifrelenmemiş service account şifresini doğrudan HTTP istek gövdesinde sunmalıdır.
B. İstek, OpenID Connect yalnızca insan giriş akışlarına uygulandığı için, herhangi bir service-to-service çağrısı kimlik doğrulanmadan önce bir insan operatör tarafından çözülen bir CAPTCHA yanıtı sunmalıdır.
C. İstek, kimliğin kanıtı olarak Google tarafından imzalanmış bir OpenID Connect ID token'ı sunmalıdır; OpenID Connect, bir authorization server tarafından gerçekleştirilen kimlik doğrulamaya dayanarak bir client'ın kimlik doğrulamasını sağlayan, OAuth 2.0 tabanlı bir identity protokolüdür, ve ayrıca client hakkında temel profil bilgisi elde etmek için de kullanılabilir.
D. İstek, OpenID Connect ya da başka herhangi bir identity protokolüyle hiçbir ilişkisi olmayan, kalıcı olarak geçerli, süresi dolmayan bir API anahtarı sunmalıdır.

**11.** Google Cloud kaynakları nasıl organize edilir, ve bu yapıda belirli bir kaynağın kaç ebeveyni vardır?

A. Kaynaklar, hiçbir türde hiyerarşi olmadan tamamen düz bir yapıda var olur, ve "ebeveyn" kaynak kavramı Google Cloud'da hiçbir yerde geçerli değildir.
B. Google Cloud kaynakları hiyerarşik olarak organize edilir: organization kök düğümdür, folder'lar altında opsiyonel bir gruplama mekanizmasıdır (örn. departmanlara ya da ekiplere eşlenir), ve project, kaynak oluşturmak, API'leri kullanmak, izinleri yönetmek, ve billing'i etkinleştirmek için gereken temel seviye varlıktır; her kaynağın tam olarak bir ebeveyni vardır, en üstteki organization düğümü hariç, onun hiç ebeveyni yoktur.
C. Her kaynağın her zaman tam olarak iki ebeveyni vardır, biri faturalandırma amaçlı biri erişim kontrolü amaçlı, hiyerarşide hiçbir yerde istisna yoktur.
D. Hiyerarşinin kök düğümünün projeler olduğu, organizasyonların ise bireysel projelerin altında opsiyonel bir gruplama mekanizması olarak var olduğu belirtiliyor — modülün gerçek yapısının tersi.

**12.** Google Cloud resource hierarchy'sindeki her kaynağın kendi IAM policy'si var mıdır, ve üzerinde izinleri nasıl verirsin?

A. Yalnızca organization düğümünün bir IAM policy'si vardır; belirli bir Pub/Sub topic'i gibi altındaki bireysel kaynakların kendi IAM policy'si yoktur.
B. IAM policy'leri yalnızca faturalandırmayla ilgili kaynaklar için mevcuttur, ve bir Pub/Sub topic'i gibi faturalandırma dışı kaynaklar üzerinde hiçbir zaman doğrudan izin verilemez.
C. Yalnızca projelerin IAM policy'leri vardır; bir projenin altındaki folder'ların ve bireysel kaynakların hiçbir policy'ye sahip olamayacağı belirtiliyor.
D. Hiyerarşideki her kaynağın bir IAM Policy'si vardır, ve üzerinde izinleri policy binding'ler kullanarak verirsin — örneğin, belirli bir topic'in IAM policy'si üzerinde "Pub/Sub Publisher" role'ünü vermek.

**13.** Bir Google Cloud projesine, içindeki bireysel bir Pub/Sub topic'i yerine bir policy binding eklersen, bu binding'e projedeki topic'ler açısından ne olur?

A. Binding, yalnızca proje kaynağının kendisine uygulanır ve içindeki herhangi bir Pub/Sub topic'i dahil, herhangi bir alt kaynak üzerinde kesinlikle hiçbir etkisi yoktur.
B. Binding, her bireysel alt kaynağa tek tek elle yeniden uygulanmadıkça 24 saat içinde otomatik olarak silinir.
C. Pub/Sub topic'leri gibi daha düşük seviyeli kaynaklar, policy binding'i ebeveyn kaynaklarından (proje) miras alır — bu, yalnızca tek bir topic'e değil, bir projedeki tüm topic'lere izin vermen (örn. mesaj publish etmek için) gerektiğinde kullanışlıdır.
D. Binding, oluşturulduğunda başlangıçta belirtilen role'le ilgisiz, tamamen farklı bir role'e otomatik olarak dönüştürülür.

**14.** Bir kaynak üzerindeki "effective IAM policy" neden oluşur, ve bir ebeveyn kaynağa verilen izinler bir alt kaynakta kaldırılabilir mi?

A. Effective IAM policy, yalnızca o belirli kaynağa doğrudan eklenen binding'lerden oluşur, ebeveyn kaynaklardan gelen herhangi bir binding'i tamamen dışlar.
B. Bir kaynak üzerindeki effective IAM policy, kaynağın kendi binding'lerini artı ebeveyninden (ve o ebeveynin kendi atalarından) miras alınan binding'leri içerir; bir ebeveyn kaynağa verilen izinler, daha düşük bir seviyede geri alamayacağın binding'lerdir.
C. Effective IAM policy her 24 saatte bir sıfırdan yeniden hesaplanır ve herhangi bir ebeveyn kaynağın binding'leriyle kalıcı bir ilişkisi yoktur.
D. Bir ebeveyn kaynakta verilen izinler, inheritance'ı tamamen tavsiye niteliğinde, bağlayıcı olmayan bir şey haline getirerek, herhangi bir kısıtlama olmadan her zaman bireysel alt kaynaklar için seçici olarak geri alınabilir.

**15.** Üç IAM role türü nedir, ve genellikle production ortamlarında varsayılan olarak hangi tür verilmekten kaçınılmalıdır?

A. Üç role türü Basic, Predefined, ve Temporary'dir, temporary role'ler configuration'dan bağımsız olarak 24 saat sonra otomatik olarak süresi dolar.
B. Predefined role'lerin, tüm servisleri kapsayan en güçlü ve tehlikeli tür olarak, basic role'lerin ise mevcut en güvenli ve en granular seçenek olarak tanımlandığı belirtiliyor.
C. Custom role'lerin, hiçbir kullanıcı girdisi olmadan tamamen Google Cloud tarafından yönetildiği belirtiliyor, bu da "custom"ı bu role türü için yanıltıcı bir isim yapıyor.
D. Basic role'ler (Owner, Editor, Viewer), tüm servisleri kapsayan izinlere sahip çok güçlü role'lerdir ve production'da varsayılan olarak verilmemelidir; predefined role'ler (örn. Cloud Run Admin, Pub/Sub Publisher) belirli bir servise granular erişim sağlar ve Google Cloud tarafından yönetilir; custom role'ler, kullanıcının belirttiği bir izin listesinden kendi role'ünü inşa etmeni sağlar.

**16.** Modül, Cloud Run'ın default service account'unu neden sadece bir kolaylık takası değil, doğal bir güvenlik riski olarak tanımlıyor?

A. Kullanılan default service account, projede geniş Editor role'üne sahip olan Compute Engine service account'udur; policy binding inheritance nedeniyle, bu default hesap projedeki çoğu kaynak üzerinde okuma ve yazma izinlerine sahiptir, bu da kaynakların bu hesap aracılığıyla oluşturulabileceği, değiştirilebileceği, ya da silinebileceği anlamına gelir — sadece bir rahatsızlık değil, gerçek bir güvenlik riski.
B. Default service account'ın, deploy anında açıkça seçtiğin tek, belirli bir kaynağa erişimi olduğu, daha geniş bir erişimin mümkün olmadığı belirtiliyor.
C. Cloud Run'ın, service account'un role'ünden bağımsız olarak herhangi bir kaynak değişikliğini önlediği söylendiği için, riskin tamamen teorik olduğu ve pratikte hiçbir zaman gerçekleşmediği belirtiliyor.
D. Default service account'ın hiçbir türde izne sahip olmadığı, bu da onu tamamen güvenli ama herhangi bir gerçek workload için işlevsel olarak kullanışsız kıldığı belirtiliyor.

**17.** Modül, bir Cloud Run servisindeki default service account'un güvenlik riskini azaltmak için hangi üç adımı öneriyor?

A. Proje için IAM'i tamamen devre dışı bırak, çünkü IAM'in kendisinin, riski azaltmak için bir araç olmaktan çok, riskin altında yatan neden olduğu belirtiliyor.
B. Cloud Run servisi için yeni bir service account oluştur, o service account'u servisin identity'si olarak yapılandır, ve o identity için, servisin gerçekten erişmesi gereken kaynaklara, sadece predefined ya da custom role'lerle policy binding'ler ekle.
C. Owner basic role'ünü mevcut default service account'a ver, çünkü Owner'ın Editor'dan daha kısıtlayıcı ve dolayısıyla daha güvenli olduğu belirtiliyor.
D. Cloud Run servisini tamamen sil ve onu bir Compute Engine sanal makinesiyle değiştir, çünkü Compute Engine'in bu tür bir riske karşı bağışık olduğu belirtiliyor.

**18.** Bir Cloud Run servisi için ilk kez yeni, user-managed bir service account oluşturduğunda, hangi izinlere sahiptir, ve servis herhangi bir izin vermeden önce bir Google Cloud API'sini çağırmaya çalışırsa ne olur?

A. Yeni service account, default Compute Engine service account'unun sahip olduğu her izni otomatik olarak miras alır, bu da onu değiştirmesi amaçlanan default'tan daha kısıtlayıcı olmayan bir hale getirir.
B. Yeni service account'a, oluşturulduğu anda tüm organizasyon üzerinde otomatik olarak Owner role'ü verilir, hiçbir türde ek configuration gerektirmez.
C. Yeni service account, oluşturulduğu anda herhangi bir Google Cloud API'sini hemen çağırabilir, çünkü service account oluşturmanın kendisi zımni bir evrensel erişim izni olarak ele alınır.
D. Varsayılan olarak, yeni oluşturulan service account'un hiçbir izni yoktur ve hiçbir policy binding'de görünmez; Cloud Run servisinin bir parçası olarak çalışan kod, bir policy binding ona bir role vermeden önce herhangi bir Google Cloud API'sini çağırırsa, çağrı IAM tarafından reddedilir.

**19.** Cloud Run'da environment variable'lar nasıl çalışır, ve aynı isimli bir değişken hem Dockerfile'da bir `ENV` varsayılanı olarak hem de doğrudan Cloud Run servisinde ayarlanırsa ne olur?

A. Cloud Run servisinde ya da job'ında ayarlanan environment variable'lar key-value çiftleri olarak ayarlanır, uygulama container'ına inject edilir, ve runtime'da uygulama koduna erişilebilir hale getirilir (örn. Python'da `os.environ.get("key")`, Node.js'te `process.env.key`, ya da Java'da `System.getenv("key")` ile); Cloud Run servisinde ya da job'ında aynı isimle ayarlanan bir değişken, Dockerfile'daki `ENV` ile ayarlanan varsayılan değeri override eder.
B. Bir Cloud Run servisinde ayarlanan environment variable'lar yalnızca Google Cloud console'a görünürdür ve çalışan uygulama container'ına hiçbir zaman gerçekten inject edilmez ya da ondan erişilemez.
C. Bir Dockerfile'ın `ENV` varsayılanı her zaman Cloud Run servisinde ayarlanan aynı isimli herhangi bir değişkenden önceliklidir, bu da Cloud Run seviyesindeki ayarı tamamen kozmetik ve işlevsiz kılar.
D. Bir Cloud Run servisinde bir environment variable ayarlamak, container image'ın kendisinin yeniden build edilip yeniden paketlenmesini gerektirir, çünkü environment variable'ların yalnızca build zamanında image'a kalıcı olarak gömüldüğü belirtiliyor.

**20.** Bir Secret Manager secret'ı neden oluşur, ve bir secret'ı bir Cloud Run servisine erişilebilir kılmanın iki yolu nedir?

A. Bir secret'ın, hiçbir türde metadata'sı olmayan, tek, versiyonlanmamış bir string'den oluştuğu, ve yalnızca hiçbir volume-mount seçeneği olmadan düz bir environment variable olarak erişilebileceği belirtiliyor.
B. Secret'lara yalnızca, container image'ı build etmeden önce değerlerinin doğrudan uygulamanın kaynak koduna sabit kodlanmasıyla erişilebilir, çünkü Cloud Run'ın Secret Manager'a erişmek için hiçbir runtime mekanizması sağlamadığı belirtiliyor.
C. Bir secret, (replikasyon konumları, etiketler, ve izinler gibi) metadata artı gerçek secret verisini metin string'i ya da binary blob olarak saklayan secret version'lardan oluşur; secret'ı bir volume olarak mount edebilirsin (container'a bir dosya olarak sunulur, okumada her zaman Secret Manager'dan en güncel değeri getirir) ya da bir environment variable olarak geçirebilirsin (instance başlangıcında bir kez çözülür, bu yüzden "latest" kullanmak yerine belirli bir version'a pinlemen önerilir).
D. Bir secret'ın, hiçbir ayrı servis, versiyonlama, ya da erişim kontrolü mekanizması içermeden, her açıdan sıradan bir Cloud Run environment variable'ıyla aynı olduğu belirtiliyor.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
IAM, API isteğini inceler ve istekteki credential'lardan çağıran uygulamayı tanımlar. Ardından, o identity için hangi operasyonlara izin verildiğini belirlemek üzere, hedef kaynağa (örneğin bir Pub/Sub topic'i) attach edilmiş IAM policy'deki policy binding'leri kontrol eder, yetkili değilse çağrıyı reddeder.

**2. Doğru cevap: D.**
Bir policy binding, bir ya da daha fazla member'ı (identity'yi) tek bir role'e bağlar. Bir role, member identity'nin Google Cloud kaynakları üzerinde belirli aksiyonlar gerçekleştirmesini sağlayan bir izinler kümesi içerir. Bir member, bir IAM policy içinde birden fazla policy binding'e attach edilebilir, bu da o member'ın birden fazla role'e sahip olmasını sağlar.

**3. Doğru cevap: A.**
IAM; human identity'leri (bir grubun ya da domain'in parçası olabilen bir Google hesabı), service account'ları (bir sanal makine, bir Cloud Run servisi, ya da bir Cloud Run function'ı gibi makineler ya da uygulamalar tarafından kullanılır), ve "all users"ı (herkese ya da bir servise public erişime izin vermek için özel bir tanımlayıcı) destekler.

**4. Doğru cevap: C.**
Bir service account, benzersiz e-posta adresiyle tanımlanan, makineler, uygulamalar, ya da servisler tarafından kullanılan özel bir hesap türüdür. Service account'lar, şifreleri olmaması ve browser'lar ya da cookie'ler kullanarak giriş yapamamaları, diğer kullanıcıların ya da service account'ların onların adına hareket etmesine izin verilebilmesi, ve Google Workspace domain'inin üyesi olmamaları (gruplara eklenebilseler de) açısından user account'lardan farklıdır.

**5. Doğru cevap: D.**
Bir sanal makinede, bir Cloud Run servisinde, ya da bir Cloud Build build'inin parçası olarak kod çalıştırırsan, Google Cloud API'lerine bağlanırken client kütüphanelerinin kimlik doğrulama için otomatik olarak kullandığı built-in bir service account'a erişimin vardır. Bu service account'u her zaman kendi user-managed service account'unla değiştirebilirsin, bu önerilir.

**6. Doğru cevap: A.**
Her Cloud Run servisi ya da job'ı, "service identity" olarak da bilinen bir service account'a bağlıdır. Varsayılan olarak, Cloud Run servisleri ya da job'ları, Editor role'üne sahip default Compute Engine service account'u olarak çalışır.

**7. Doğru cevap: C.**
Uygulama kodunda bir Google Cloud API'sini çağırmak için bir client kütüphanesi kullanılıyorsa, kütüphane, kodun isteklerini servisin runtime service account'unu kullanarak doğrulamak için uygun token'ları otomatik olarak elde eder; çoğu Google API'sine erişirken OAuth 2.0 access token'ları kullanılır. IAM, access token'ı doğrular ve hedef kaynak üzerinde gereken role'lere sahip bir policy binding olup olmadığını kontrol eder.

**8. Doğru cevap: B.**
Cloud Run servisleri arasındaki asenkron iletişim için, Cloud Tasks, Pub/Sub, Cloud Scheduler, ya da Eventarc gibi çeşitli Google Cloud servisleri kullanılabilir. Senkron iletişim için, bir servis başka bir servisin endpoint URL'ini doğrudan HTTP üzerinden çağırır; çağıran servis için IAM ve bireysel bir service identity kullanmak, gereken minimum izin kümesiyle, bir best practice'tir.

**9. Doğru cevap: A.**
Senkron erişimi kurmak için, çağıran servisin service account'unu alıcı servis üzerinde bir principal yaparak, alıcı servisi çağıran servisten gelen istekleri kabul edecek şekilde yapılandırırsın, ardından o service account'a Cloud Run Invoker (`roles/run.invoker`) role'ünü verirsin — örneğin, `gcloud run services add-iam-policy-binding` komutuyla, çağıran servisin identity'si member olarak ve `roles/run.invoker` role olarak.

**10. Doğru cevap: C.**
Çağıran servisin yaptığı istek, kimliğinin kanıtını Google tarafından imzalanmış bir OpenID Connect token'ı biçiminde sunmalıdır. OpenID Connect, bir authorization server tarafından gerçekleştirilen kimlik doğrulamaya dayanarak bir client'ın kimlik doğrulamasını sağlayan, OAuth 2.0 tabanlı bir identity protokolüdür, ve ayrıca client hakkında temel profil bilgisi elde etmek için de kullanılır.

**11. Doğru cevap: B.**
Google Cloud kaynakları hiyerarşik olarak organize edilir: organization kaynağı, merkezi görünürlük ve kontrol sağlayan kök düğümdür, folder'lar organization'ın altında opsiyonel bir gruplama mekanizmasıdır (departmanlara, ekiplere, ya da iş birimlerine eşlenebilir), ve project, kaynak oluşturmak, Cloud API'lerini ve servislerini kullanmak, izinleri yönetmek, ve billing'i etkinleştirmek için gereken temel seviye varlıktır. Her kaynağın tam olarak bir ebeveyni vardır, en üstteki organization düğümü hariç, onun hiç ebeveyni yoktur.

**12. Doğru cevap: D.**
Hiyerarşideki her kaynağın bir IAM Policy'si vardır, ve üzerinde izinler policy binding'ler kullanılarak verilir — örneğin, belirli bir topic'e attach edilmiş IAM policy üzerinde bir policy binding aracılığıyla "Pub/Sub Publisher" role'ünü vermek.

**13. Doğru cevap: C.**
Bir policy binding, bireysel bir Pub/Sub topic'i yerine bir proje gibi daha yüksek seviyeli bir kaynağa eklenirse, daha düşük seviyeli kaynaklar tarafından miras alınır — bu durumda, Pub/Sub topic'leri, ebeveyn kaynakları olan Google Cloud projesinden policy binding'i miras alır. Bu, yalnızca tek bir topic'e değil, bir projedeki tüm topic'lere mesaj publish etme izni vermen gerektiğinde kullanışlıdır.

**14. Doğru cevap: B.**
IAM, bir kaynağa erişimi değerlendirdiğinde, ebeveyn kaynağın (ve o ebeveynin kendi atalarının) policy binding'lerini de değerlendirir. Kaynak üzerindeki effective IAM policy, bir ebeveyne verilmiş olan ve daha düşük seviyede geri alamayacağın binding'leri de içerir.

**15. Doğru cevap: D.**
Üç tür IAM role vardır: basic role'ler (Owner, Editor, Viewer), tüm servisleri kapsayan izinlere sahip çok güçlü role'lerdir ve production ortamlarında varsayılan olarak verilmemelidir; predefined role'ler (Cloud Run Admin, Pub/Sub Publisher, ya da Cloud Tasks Enqueuer gibi), belirli bir servisi kullanmak için granular role'ler sağlar ve Google Cloud tarafından yönetilir; custom role'ler, kendi spesifik kullanım durumun için bir izinler listesinden kendi role'ünü inşa etmeni sağlar.

**16. Doğru cevap: A.**
Cloud Run tarafından kullanılan default service account, projede geniş Editor role'üne sahip olan Compute Engine service account'udur. Policy binding inheritance nedeniyle, bu default service account, projedeki çoğu kaynak üzerinde okuma ve yazma izinlerine sahiptir — kullanışlı olsa da, bu doğal bir güvenlik riskidir, çünkü kaynaklar bu service account kullanılarak oluşturulabilir, değiştirilebilir, ya da silinebilir.

**17. Doğru cevap: B.**
Bu güvenlik riskini azaltmak için modül şunları öneriyor: bir Cloud Run servisi için yeni bir service account oluşturmak, o service account'u Cloud Run servisinin identity'si olarak yapılandırmak, ve o identity için, sadece servisin erişmesi gereken kaynaklara, predefined ya da custom role'lerle policy binding'ler eklemek.

**18. Doğru cevap: D.**
Varsayılan olarak, yeni oluşturulan bir service account'un hiçbir izni yoktur — henüz hiçbir policy binding'de görünmez. Cloud Run servisinin bir parçası olarak çalışan koddan, bir policy binding hesaba bir role vermeden önce herhangi bir Google Cloud API'si çağrılırsa, çağrı IAM tarafından reddedilir.

**19. Doğru cevap: A.**
Cloud Run environment variable'ları key-value çiftleri olarak ayarlanır, uygulama container'ına inject edilir, ve runtime'da uygulama kodu tarafından erişilir — Python'da `os.environ.get("key")`, Node.js'te `process.env.key`, ya da Java'da `System.getenv("key")` gibi fonksiyonlar kullanılarak. Varsayılan environment variable'lar bir Dockerfile'daki `ENV` ifadesiyle container'da ayarlanabilir, ve bir Cloud Run servisinde ya da job'ında aynı isimle ayarlanan bir environment variable, o varsayılanda ayarlanan değeri override eder.

**20. Doğru cevap: C.**
Secret Manager'daki bir secret, (replikasyon konumları, etiketler, ve izinler gibi) bir metadata koleksiyonu ve gerçek secret verisini — bir API anahtarı ya da şifre gibi — metin string'i ya da binary blob olarak saklayan secret version'lardan oluşan bir nesnedir. Bir secret, bir Cloud Run servisine ya da job'ına ya bir volume olarak mount edilerek (container'a bir dosya olarak sunulur, okumalar her zaman Secret Manager'dan değeri getirir, latest version ile kullanıma uygundur) ya da bir environment variable olarak geçirilerek (instance başlangıç zamanında çözülür, bu yüzden latest kullanmak yerine belirli bir version'a pinlenmesi önerilir) erişilebilir kılınabilir.
