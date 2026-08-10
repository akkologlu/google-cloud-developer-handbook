# Introduction to Microservices — Baştan Sona Öğretici

> Bu metin, **"Introduction to Microservices"** dersinde anlatılan **her şeyi** kavratmak için yazıldı. Bu ders, "Service Orchestration and Choreography on Google Cloud" kursunun 1. modülüdür. Modülün merkezinde tek bir soru var: **Bir uygulamayı büyütmek zorunda kaldığında, kod tabanını nasıl parçalara ayırırsın — ve bu parçalama kararı neden bu kadar önemli?** Ders, bu soruya üç farklı mimari yaklaşımın (monolithic applications, Service-Oriented Architecture, microservices) tarihsel gelişimi üzerinden cevap veriyor; sonra da microservices'in ne zaman tercih edileceğini, hangi faydaları getirdiğini ve hangi zorlukları beraberinde sürüklediğini öğretiyor.

> **Kapsam notu:** Bu doküman şu anda yalnızca **Introduction to Microservices** dersini kapsıyor. Bu dersin ait olduğu kursun ismi "orchestration and choreography" olsa da, bu terimler ve onları uygulayan Google Cloud servisleri (Pub/Sub, Cloud Tasks, Workflows, Eventarc gibi) **henüz bu derste işlenmedi** — derste sadece microservices'in ne olduğu, neden ortaya çıktığı ve hangi problemleri beraberinde getirdiği anlatılıyor. Kursun sonraki modüllerinin transkriptleri eklendikçe, orchestration/choreography konuları ayrı bir modül olarak bu handbook'a eklenecek ve tam olarak bu derste ortaya konan zorlukları (özellikle "servisler arası iletişimin karmaşıklığı" ve "operasyonel yük") çözmek için nasıl kullanıldığı anlatılacak.

---

## Bu ders tam olarak neyi öğretiyor ve neden önemli?

Bir düşünce deneyiyle başlayalım. Sıfırdan bir kurumsal (enterprise) uygulama yazmaya başlıyorsun. İlk günlerde her şey tek bir kod tabanında, tek bir takımda, tek bir deploy sürecinde yürüyor — ve bu gayet mantıklı, çünkü küçük bir uygulamayı gereksiz yere parçalara bölmek sadece karmaşıklık katar. Ama uygulama büyüdükçe, takım büyüdükçe, özellik sayısı arttıkça bir noktada şu soruyla karşılaşırsın: **Bu kod tabanını böyle tek parça bırakmaya devam mı edeyim, yoksa parçalara mı ayırayım?**

Bu ders, tam olarak bu sorunun tarihini ve cevabını anlatıyor. Dersin kendi açılış cümlesiyle: **Bugün birçok yeni kurumsal uygulama microservices mimarisiyle tasarlanıyor.** Ama microservices'i anlamak için önce ondan önce ne vardı, o yaklaşımın hangi sorunları çözemediğini ve microservices'in bu boşluğu nasıl doldurduğunu görmen gerekiyor. Ders bu yüzden üç durağı sırayla geziyor:

1. **Monolithic applications (monoliths)** — geleneksel, tek parça uygulama yaklaşımı ve onun getirdiği sorunlar.
2. **Service-Oriented Architecture (SOA) ve Enterprise Service Bus (ESB)** — monolith sorununu çözmek için yapılan ilk kurumsal girişim, ve bu girişimin nereye çıktığı.
3. **Microservices** — SOA'nın merkezi/kurumsal yaklaşımına karşı, ayrıştırılmış (decentralized) bir alternatif.

Bu üçünü ayrı ayrı mimari stiller olarak değil, **aynı problemin çözümünde birbirini izleyen nesiller** olarak düşün. Her nesil, bir öncekinin çözemediği bir sorunu çözüyor ama kendi yeni sorununu da beraberinde getiriyor. Dersin son bölümü de zaten bunu itiraf ediyor: microservices'in faydaları genelde zorluklarına ağır basıyor, ama zorluklar **gerçek** ve **ciddi**. İşte bu yüzden kursun geri kalanı (orchestration ve choreography) tam olarak bu zorlukları azaltmaya adanmış olacak.

---

# BÖLÜM 1 — Monolithic Applications: Sorunun Başlangıç Noktası

## Monolith nedir?

Ders, "önce uygulamaların geleneksel olarak nasıl tasarlandığını anlamalıyız" diyerek başlıyor. Erken kurumsal uygulamalar, **büyük, kendi kendine yeten (self-contained)** uygulamalar olarak geliştiriliyordu. Bu uygulamalar **kullanıcı arayüzünü (user interface), iş mantığını (business logic) ve veri erişim kodunu (data access code)** aynı çatı altında barındırıyordu; veri de genellikle **büyük, ilişkisel (relational) bir veritabanında** saklanıyordu.

Bugün bu tür uygulamalara **monolithic applications**, ya da kısaca **monoliths** diyoruz.

> **Analoji:** Bir monolith'i, mutfağı, kasayı, deposu ve masaları aynı büyük salonda barındıran bir restorana benzet. Her şey aynı çatı altında; menüde bir değişiklik yapmak istediğinde bile mutfağa, kasaya ve servis akışına aynı anda dokunman gerekebilir — çünkü hiçbir bölüm birbirinden gerçekten bağımsız değil.

## Monolith neden sorun çıkarır?

Bu uygulamalar **birçok görevi aynı anda halletmek üzere tasarlanmıştı**, bu yüzden kod tabanı **kaçınılmaz olarak karmaşıktı**. Ders burada kritik bir gözlem yapıyor: **Uygulamaya yapılan her büyük değişiklik, kod tabanını daha da karmaşık hale getirme eğilimindeydi.** Yani karmaşıklık statik bir maliyet değil, **zamanla büyüyen bir maliyettir.**

Bunun temel nedeni de açıklanıyor: Kodun tamamı tek bir uygulama içinde olduğu için, uygulama kodu genellikle **sıkı sıkıya bağlıydı (tightly coupled).** Bu birbirine bağımlı (interdependent) kodu bakımını yapmak zordu; bu da **bir hatayı düzeltirken yeni hatalar sokmadan** düzeltme yapmayı zorlaştırıyordu.

> **Neden bu döngü bu kadar tehlikeli?** Çünkü kısır bir döngü yaratıyor: Kod tabanı karmaşıklaştıkça değişiklik yapmak zorlaşır; değişiklik yapmak zorlaştıkça yapılan her değişiklik daha fazla yan etki (side effect) yaratma riski taşır; her yan etki de kod tabanını biraz daha karmaşık ve kırılgan hale getirir. Bu döngüyü kıracak bir mimari değişiklik olmazsa, uygulama zamanla "dokunulması korkulan" bir yapıya dönüşür — tam olarak SOA ve microservices'in çözmeye çalıştığı durum budur.

> **Sınav tuzağı:** "Monolithic applications kötüdür, her zaman microservices'e geçilmelidir" gibi bir çıkarım yapmak **yanlıştır.** Ders ileride açıkça söylüyor: eğer problem alanında (problem domain) yeterli uzmanlığın yoksa, **monolith ile başlamak** doğru bir stratejidir. Monolith'in kendisi kötü değildir; büyüdükçe ve doğru şekilde bölünmeden büyütüldükçe sorun çıkarır. Bu ayrımı BÖLÜM 4'te detaylı göreceğiz.

---

# BÖLÜM 2 — Service-Oriented Architecture (SOA) ve Enterprise Service Bus (ESB)

## SOA neden ortaya çıktı?

**Service-Oriented Architecture (SOA)**, monolithic applications'ın yarattığı zorlukları çözme girişimiydi. SOA, **yeniden kullanılabilir yazılım bileşenleri** (services) inşa etmeye odaklanan bir mimari stildir.

SOA'daki her bir servis, **ayrık (discrete) bir iş fonksiyonunu** yerine getirmeliydi; servisler arası iletişim de **tanımlı servis arayüzleri (defined service interfaces) üzerinden mesajlaşma (messaging)** ile gerçekleştiriliyordu.

Burada dikkat edilmesi gereken önemli bir nokta var: SOA tipik olarak **kurumsal (enterprise) düzeyde** uygulanıyordu. Organizasyonlar iş aktivitelerini (business activities) servislere eşliyor (map ediyor) ve servisleri için **birlikte çalışabilirlik (interoperability) ve keşfedilebilirlik (discoverability) standartları** zorunlu kılıyordu.

> **Neden bu "kurumsal düzey" vurgusu önemli?** Çünkü bu, SOA'yı microservices'ten ayıran en temel farklardan biridir. SOA, tek bir takımın kendi başına karar verip uygulayabileceği bir şey değildi — organizasyon çapında standartlar, yönetişim (governance) ve merkezi koordinasyon gerektiriyordu. Microservices'i BÖLÜM 3'te gördüğünde bu farkı hatırla: microservices **ayrıştırılmış (decentralized)** bir yaklaşımdır, SOA ise doğası gereği **merkezi.**

## SOA'nın gerçek faydaları

Ders açık sözlü: **SOA somut faydalar sağladı, ama tipik olarak karışık (mixed) sonuçlar üretti.** Önce faydalara bakalım.

En net fayda, servislerin büyük monolithic uygulamalara kıyasla **daha küçük ve daha gevşek bağlı (loosely coupled)** olmasıydı. Daha küçük servisler genellikle **tek bir servise ve daha küçük bir problem alanına odaklanan daha küçük geliştirme takımlarına** yol açıyordu, bu da verimliliği artırabiliyordu.

SOA'nın büyük bir odak noktası da **servis yeniden kullanımıydı (service reuse).** Uygulamalar, servisleri **birleştirerek (combining)** oluşturuluyordu — yani bir servis, farklı uygulamalar tarafından tekrar tekrar kullanılabiliyordu.

## Enterprise Service Bus (ESB): SOA'nın omurgası

Servisler arasındaki mesajlar, **Enterprise Service Bus (ESB)** adı verilen bir mesajlaşma ara katmanı (messaging middleware) bileşeni tarafından yönetiliyordu.

ESB, **bağlantı (connectivity), güvenlik (security), mesaj yönlendirme (message routing) ve dönüştürme (transformation)** işlerini üstlenerek, uygulamaların — organizasyon dışındaki uygulamalar da dahil olmak üzere — entegre edilmesini mümkün kılıyordu. Uygulamalar ESB'ye bağlanıyor; ESB de protokolleri dönüştürüyor, servisler arasında mesajları yönlendiriyor ve veriyi dönüştürüyordu.

> **Analoji:** ESB'yi büyük bir havalimanının merkezi kontrol kulesine benzet. Her uçuş (mesaj) doğrudan diğer uçaklarla konuşmuyor; hepsi kuleyle (ESB) iletişim kuruyor, kule de kimin nereye, hangi rotayla, hangi kurallarla gideceğine karar veriyor. Bu, kaosu önler — ama aynı zamanda **her şeyin kuleden geçmesi gerektiği** anlamına gelir; kule tıkanırsa, tüm sistem tıkanır.

## SOA'nın gerçek maliyeti: Karmaşıklık nereye taşındı?

Burada dersin en önemli gözlemlerinden birine geliyoruz: **SOA, servis kodundaki karmaşıklığı azalttı, ama bu karmaşıklık tipik olarak ESB entegrasyonlarına kaydı (shifted).**

ESB entegrasyonları, uygulamaları başarıyla başlatmanın ve güncellemenin **darboğazı (bottleneck)** haline geldi. Bunun somut nedenleri şunlardı:

- ESB tipik olarak **merkezi bir takım** tarafından yönetiliyordu, ve tüm uygulama ve servis takımlarının ESB üzerinde çalışması gerektiği için ESB entegrasyon işi **ciddi gecikmelerle** karşılaşabiliyordu.
- Bir uygulama için bir entegrasyonu değiştirmek, **aynı entegrasyonu kullanan diğer uygulamaları istikrarsızlaştırabiliyordu (destabilize).**
- ESB yazılımının kendisine yapılan güncellemeler bile **mevcut entegrasyonları bozabiliyordu**, bu yüzden ESB güncellemeleri **kapsamlı testler** gerektiriyordu.

> **Neden bu döngü tanıdık geliyor?** Çünkü bu, BÖLÜM 1'de gördüğün monolith sorununun **aynı örüntüsünün** başka bir katmanda tekrarlanmasıdır. Monolith'te sorun "her şey tek bir kod tabanında sıkı sıkıya bağlı" idi. SOA bunu servis düzeyinde çözdü ama karmaşıklığı yok etmedi — sadece **ESB'ye taşıdı.** Sonuç olarak ESB, kendi başına bir "merkezi darboğaz monolith'i" haline geldi. Bu gözlem çok önemli çünkü microservices'in neden **merkezi bir ara katman olmadan** çalışmayı tercih ettiğini anlamanı sağlıyor.

> **Sınav tuzağı:** "SOA başarısız oldu, bu yüzden kimse kullanmamalı" demek yanlıştır — ders SOA'nın **tangible (somut) faydalar sağladığını** açıkça belirtiyor (küçük/loosely coupled servisler, yeniden kullanım). Doğru çıkarım şudur: SOA, sorunu **çözmedi, taşıdı.** Karmaşıklık servis kodundan ESB entegrasyonuna kaydı ve ESB merkezi bir darboğaz haline geldi. Sınav sorusu "SOA'nın temel zayıflığı neydi?" diye sorarsa cevap **"ESB merkezi bir darboğaz ve değişiklik riskine dönüştü"** olmalı, "SOA hiçbir fayda sağlamadı" değil.

---

# BÖLÜM 3 — Microservices: Ayrıştırılmış Bir Alternatif

## Microservices nedir?

**Microservices**, uygulamaları servislere ayrıştırmak (decompose) için **alternatif, ayrıştırılmış (decentralized) bir yaklaşımdır.**

Microservices, **kapsamı sınırlı (limited in scope)**, ayrı servislerdir. Dersin verdiği örnek çok öğretici: bir e-ticaret uygulamasında **orders (siparişler), products (ürünler) ve reviews (yorumlar)** gibi her bir iş alanı (business domain), **kendi veritabanıyla birlikte kendi microservice'i** içinde yaşar.

Bir microservice, diğer servisler tarafından microservice'in operasyonlarını çağırmak için kullanılan bir **arayüz (interface)** — tipik olarak bir **API** — belirtir (specify).

> **SOA ile fark burada netleşiyor:** SOA'da servisler bir ESB üzerinden, merkezi bir ara katmandan iletişim kuruyordu. Microservices'te ise her servis kendi API'sini doğrudan sunar ve çağıran servisler bu API'ye **doğrudan** bağlanır — aradaki merkezi mesajlaşma katmanı (ESB) yok. Bu, microservices'i "ayrıştırılmış (decentralized)" yapan şeyin tam olarak kendisidir.

## Neden loose coupling bu kadar önemli?

Microservices'in ayrılığı (separation), microservices arasında **gevşek bağlılığa (loose coupling)** yol açma eğilimindedir. Gevşek bağlı servisler; **bakımı, güncellenmesi ve dağıtılması (deploy edilmesi)** daha kolay servislerdir.

> **Analoji:** Bir microservices mimarisini, her biri kendi menüsü, kendi mutfağı ve kendi kasası olan **ayrı restoranların bulunduğu bir yemek pasajına (food court)** benzet. Bir restoran menüsünü değiştirdiğinde, diğer restoranların hiçbiri etkilenmez — çünkü aralarında ortak bir mutfak (monolith'te olduğu gibi) ya da ortak bir merkezi sipariş kulesi (SOA'daki ESB gibi) yok. Her biri kendi API'sini (siparişini nasıl aldığını) sunar, geri kalanı kendi işidir.

Bu, BÖLÜM 1'deki restoran analojisiyle doğrudan tezat oluşturuyor: monolith'te tek salon vardı, her şey birbirine bağlıydı. Microservices'te her "restoran" (servis) bağımsız çalışır ve sadece tanımlı arayüzü (API) üzerinden dış dünyayla konuşur.

---

# BÖLÜM 4 — Microservices mi, Monolith mi? Ne Zaman Hangisiyle Başlamalı?

Yeni bir uygulama tasarlamaya başladığında, monolith yerine doğrudan microservices ile başlamaya karar verebilirsin. Ama ders burada çok pratik, sınav-dostu bir uyarı veriyor: **Yeni bir microservices uygulamasını mimarilemenin en zor kısımlarından biri, servis sınırlarını (service boundaries) tasarlamaktır.**

## Servis sınırlarını tasarlamak neden bu kadar zor?

Eğer bir uygulama tasarlıyorsan ve problem alanında (problem domain) uzmanlığın yoksa, oluşturulacak ayrı servisleri seçmek **zor olabilir.** Yani "orders, products, reviews gibi hangi sınırları çizmeliyim?" sorusuna doğru cevap vermek, o iş alanını **derinlemesine** bilmeyi gerektirir.

Eğer monolithic bir uygulamayla başlarsan, servisleri **nasıl ayıracağını tam olarak anlamadan önce** uygulamanı inşa edebilirsin. Problem alanıyla ilgili daha fazla deneyim kazandıkça, **daha sonra** microservices mimarisine geçiş yapabilirsin (migrate).

> **Neden bu sezgiye aykırı geliyor?** Çünkü "en modern/en iyi mimariyle baştan başlamak" doğal bir içgüdüdür. Ama servis sınırlarını **yanlış** çizmek — mesela orders ve payments'ı gereğinden fazla ayırmak ya da gereğinden az ayırmak — sonradan düzeltmesi çok pahalı bir hatadır (iki servis arasında veri taşımak, API'leri yeniden tasarlamak, veritabanlarını bölmek). Domain'i henüz tanımıyorsan, monolith ile başlamak sana **ucuza yanılma** hakkı verir; yanlış sınırları monolith içinde ücretsiz "refactor" edebilirsin, ama microservices'te bu maliyetli bir migrasyon operasyonuna dönüşür.

> **Sınav tuzağı:** "Microservices her zaman monolith'ten üstündür, bu yüzden her yeni proje microservices ile başlamalıdır" — bu **doğrudan yanlış** bir çıkarımdır ve ders bunun tam tersini söylüyor. Eğer problem alanında uzmanlığın yoksa, **monolith ile başlamak** önerilen yaklaşımdır. Sertifikasyon sorularında "yeni bir proje için hangi mimariyle başlamalı" tarzı bir soru geldiğinde, cevap **her zaman "microservices"** değildir — bağlam (domain bilgisi var mı, ekip büyüyecek mi, hızlı iterasyon şart mı) belirleyicidir.

## Ne zaman microservices ile başlamak mantıklıdır?

Ders, monolith lehine olan yukarıdaki uyarının hemen ardından, microservices ile başlamayı **destekleyen** iki somut senaryo veriyor.

**1. Hızlı ve çevik (agile) teslimat gerekiyorsa.** Microservices'i agile bir şekilde teslim etmek, monolith'leri teslim etmekten **daha kolaydır.** Eğer servislerine hızlıca değişiklik yayınlama (release) yeteneğine ihtiyacın varsa, muhtemelen microservices ile başlamak mantıklıdır.

**2. Takımını büyütmeyi planlıyorsan.** Eğer takımının boyutunu büyütmeyi planlıyorsan, microservices yeni takım üyelerinin genel uygulamanın **daha küçük parçalarına odaklanmasına** izin verir. Takım üyelerinin büyük bir monolithic kod tabanını öğrenmesi gerekmez. Bir microservices uygulaması, her takımın **azaltılmış bir kapsama (reduced scope)** sahip olduğu, daha küçük takımlara izin veren **doğal servis sınırları** sağlar.

## Karar tablosu

| Durum | Önerilen başlangıç noktası | Neden |
| --- | --- | --- |
| Problem alanında (domain) uzmanlığın yok | **Monolith** | Servis sınırlarını erken ve yanlış çizme riski yüksek; monolith içinde ücretsiz yeniden düzenleme (refactor) yapılabilir |
| Hızlı, sık ve bağımsız release gerekiyor | **Microservices** | Her servis kendi takvimiyle deploy edilebilir; agile teslimat monolith'te daha zordur |
| Takım büyüyecek, yeni üyeler ekleniyor | **Microservices** | Doğal servis sınırları, her takıma küçük ve odaklı bir kapsam verir |
| Domain'i biliyorsun ama ekip küçük ve sabit | Monolith (modüler tasarlanmış) ya da microservices | Ders kesin bir kural vermiyor; anahtar, domain bilgisi ve gelecekteki büyüme beklentisidir |

## Monolith ile başlıyorsan bile bunu unutma

Ders, bu bölümü çok pratik bir tavsiyeyle kapatıyor: **Bir monolith ile başlıyorsan, uygulamanı modüler (modular) olacak şekilde tasarla** — böylece ileride microservices'e geçiş yapmak daha kolay olur.

> **Bu tavsiye neden kritik?** Çünkü "monolith ile başla" tavsiyesi, "kod tabanını gelişigüzel, sıkı sıkıya bağlı bir şekilde yaz" anlamına gelmiyor. BÖLÜM 1'de gördüğün monolith'in asıl sorunu **tight coupling**'di, "tek uygulama olması" değildi. Modüler bir monolith — yani iç sınırları net, bağımlılıkları kontrollü bir monolith — hem bugün bakımı kolay bir uygulamadır hem de yarın microservices'e ayrıştırılması nispeten kolay bir başlangıç noktasıdır. Modüler olmayan bir monolith ise, tam olarak BÖLÜM 1'de anlatılan "her değişiklik karmaşıklığı artırır" kısır döngüsüne düşer ve migrasyonu da katlanarak zorlaşır.

---

# BÖLÜM 5 — Microservices'in Faydaları

Uygulamalarını microservices olarak geliştirmenin birçok faydası var. Ders bunları üç ana başlık altında topluyor.

## 1. Basitlik ve takım odaklanması

En bariz fayda, her bir microservice'in büyük, monolithic uygulamadan **daha basit** olmasıdır. Her microservice, daha küçük ve daha basit bir kod tabanına sahiptir; bu da genellikle **tek, küçük bir takımın** o microservice'in iç detaylarına odaklanmasına imkân tanır. Bu takımın üyelerinin, uygulamanın diğer kısımlarında sorun yaratmadan microservice'i anlaması ve güncellemesi **daha kolaydır.**

Bunun doğal bir sonucu olarak, microservices tipik olarak **birim test (unit test) yapmak için daha kolaydır** — çünkü farklı servisler arasında **net sınırlar** vardır.

Microservices ayrıca **ayrı ayrı deploy edilebilir (separately deployable)**, bu da takımların kendi microservice'lerini **kendi takvimlerinde** güncellemesine olanak tanır. Diğer microservices, ancak bir microservice'in arayüzünde (interface) **kırıcı (breaking) bir değişiklik** yapıldığında etkilenir.

Bu özellikler bir araya geldiğinde daha **agile bir geliştirme** sürecine yol açar, çünkü microservices diğer servisleri etkilemeden ayrı ayrı güncellenebilir ve deploy edilebilir.

> **Neden "net sınırlar" test edilebilirliği artırır?** Bir monolith'te bir fonksiyonu test etmek istediğinde, o fonksiyonun bağlı olduğu onlarca diğer bileşeni de (doğrudan ya da dolaylı olarak) hesaba katman gerekebilir çünkü her şey tightly coupled'dır. Bir microservice'te ise servisin sınırı (API'si) zaten net tanımlıdır; testin sadece bu sınırın içindeki davranışa odaklanması yeterlidir. Sınır ne kadar netse, test o kadar izole ve güvenilirdir.

## 2. Dil ve teknoloji özgürlüğü

İkinci fayda, microservices'in **farklı programlama dilleri ve teknolojiler** kullanabilmesidir. Microservices, bir **API arayüzü** kullanarak bağlanır. Microservice kodunun kullandığı programlama dilleri, framework'ler ve teknolojiler, **çağıran servisleri etkilemez.**

Bu sayede bir takım, kendi servisi için **en uygun dili ve teknolojiyi** seçebilir. Bir platformda çalışan microservices, farklı bir platformda çalışan microservices'e de bağlanabilir. Genellikle HTTP API'ler üzerinden bağlanan microservices, **servisin nerede bulunduğuyla ilgilenmeden** birbirini çağırabilir.

> **Analoji:** Bunu, farklı ülkelerden gelen turistlerin ortak bir dille (İngilizce, yani API) iletişim kurduğu bir uluslararası havalimanına benzet. Her turist kendi ana dilini (kendi programlama dilini) konuşabilir; havalimanı görevlisiyle iletişim kurmak için herkesin aynı dili bilmesi gerekmez — sadece ortak protokolü (API'yi) bilmeleri yeterlidir. Bir microservice'i Python'dan Go'ya taşısan bile, onu çağıran diğer servisler bunu **fark etmez** — çünkü onlar sadece API sözleşmesiyle (contract) ilgilenir.

> **Sınav tuzağı:** "Microservices mimarisinde tüm servisler aynı dilde/teknolojide yazılmalıdır, tutarlılık için" düşüncesi **yanlıştır.** Aksine, microservices'in en somut faydalarından biri tam olarak budur: her takım, kendi servisi için **en uygun** dili ve teknolojiyi seçebilir, çünkü servisler arası tek bağlayıcı unsur **API arayüzüdür**, uygulama teknolojisi değil.

## 3. Bağımsız ölçeklenebilirlik (scaling)

Üçüncü fayda, microservices'i **ayrı ayrı ölçeklendirebilme (separately scale)** yeteneğidir. Her microservice ayrı ayrı ölçeklendirilebilir; bu da yalnızca **daha fazla kapasiteye ihtiyaç duyan** servislere ek kaynak ayırmayı sağlar.

Microservices, trafikteki dalgalanmalara göre **büyütmek ve küçültmek (scale up/down)** genellikle kolaydır. En yüksek yük anına göre altyapı kurmak yerine, altyapı — ve dolayısıyla maliyet — **o anki trafik ihtiyacına göre** optimize edilebilir.

> **Neden bu monolith'te mümkün değil (ya da çok verimsiz)?** Bir monolith'i ölçeklendirmek istediğinde, **tüm uygulamayı** — kullanıcı arayüzünü, iş mantığını, veri erişimini, hepsini birden — ölçeklendirmek zorundasın, çünkü hepsi tek bir dağıtılabilir birimdir. Diyelim ki uygulamanın sadece "ürün arama" kısmı yoğun trafik alıyor; monolith'te bu yoğunluğu karşılamak için **tüm uygulamanın** kopyalarını çoğaltman gerekir — arama ile hiç ilgisi olmayan kısımlar da dahil. Microservices'te ise sadece "search" servisini ölçeklendirirsin, "orders" ya da "reviews" servisine hiç dokunmazsın. Bu, doğrudan **maliyet verimliliğine** dönüşür.

**Faydalar — özet tablo:**

| Fayda | Ne sağlar? | Kritik ayrıntı |
| --- | --- | --- |
| Basitlik + takım odaklanması | Küçük kod tabanı, kolay unit test, ayrı deploy | Net sınırlar sayesinde diğer servisler ancak breaking change'de etkilenir |
| Dil/teknoloji özgürlüğü | Her takım kendi dilini/framework'ünü seçer | Bağlantı API üzerinden olduğu için uygulama teknolojisi önemsizdir |
| Bağımsız scaling | Sadece ihtiyaç duyan servise kaynak ayrılır | Maliyet, o anki trafik ihtiyacına göre optimize edilir |

---

# BÖLÜM 6 — Microservices'in Zorlukları

Microservices mimarisi, bugün yeni kurumsal uygulamalar için yaygın bir mimari olsa da, ders açık sözlü: **microservices mimarileri önemli zorluklar da beraberinde getirir.**

Her bir microservice, monolithic uygulamaya kıyasla daha basit ve anlaşılması daha kolay olma eğilimindedir; her servis ayrı ayrı geliştirilebilir, deploy edilebilir ve test edilebilir. **Ama daha fazla deploy edilebilir varlığa (deployable entity) sahip olmak, bir organizasyon için daha büyük bir operasyonel yük (operational burden) yaratır.**

## 1. Operasyonel yük

Operasyon takımının onlarca, yüzlerce, hatta binlerce microservice'i yönetmesi gerekebilir. **Otomatik build, test ve deployment**, uygulamalarının ve operasyon takımının sağlığını ve verimliliğini korumak için **hayati önemdedir.**

Bu kadar çok sayıda servisle, tüm servisler genelinde **tutarlı loglama, raporlama, güvenlik ve yetkilendirme (authorization)** sağlamak da önemlidir.

> **Neden bu, monolith'te bu kadar sorun değildi?** Bir monolith'te "loglama nasıl yapılır" sorusunun tek bir cevabı vardır çünkü tek bir uygulama vardır. Yüz microservice'in olduğu bir dünyada, her takım kendi loglama biçimini, kendi güvenlik yaklaşımını seçerse, sonuçta **tutarsız, birbiriyle karşılaştırılamaz** bir sistem ortaya çıkar. Bu yüzden microservices, otomasyona ve organizasyon çapında standartlara olan bağımlılığı **azaltmaz, artırır.**

## 2. İletişim karmaşıklığı ("örümcek ağı" sorunu)

Bir microservices mimarisi, microservices arasındaki iletişimi de içerir ve bu iletişim **karmaşık olabilir.** Sistemlerini iyi tasarlamazsan, microservices arasındaki iletişimin **"örümcek ağını" (spider web)** anlamak zorlaşabilir.

> **Analoji:** Bunu, her kişinin herkesle doğrudan telefonla konuştuğu, hiçbir ortak toplantı odası ya da protokolü olmayan büyük bir ofise benzet. On kişilik bir ofiste bu yönetilebilir; ama bin kişilik bir ofiste, kimin kiminle ne zaman konuştuğunu takip etmek imkânsız hale gelir. İşte bu ders serisinin adındaki **orchestration** ve **choreography** kavramları, tam olarak bu "kimin kiminle nasıl konuşacağı" sorununu düzenli bir hale getirmek için var — ama bu konuyu ileriki derslerde göreceğiz.

## 3. İletişim gecikmesi (latency)

Microservices, iletişim gecikmesi de (communication latency) getirebilir. Bir monolith'te, bileşenler arasındaki çağrılar tipik olarak **aynı süreçte (process), aynı donanımda** çalışır. Microservices'te ise servisler arasındaki çağrılar **ağ üzerinden (across the network)** gerçekleşir — bu da **binlerce kat daha yavaş** olabilir.

Bir iş operasyonu birçok microservice çağrısı gerektirdiğinde, bu gecikme **önemli** hale gelebilir.

> **Neden bu sezgiye aykırı?** Çünkü "her servisi ayrı ayrı optimize ettim, her biri hızlı" demek, "toplam işlem hızlı" demek **değildir.** Bir kullanıcı isteği; orders → payments → inventory → notifications gibi dört microservice'i sırayla çağırıyorsa, her bir ağ çağrısının eklediği gecikme (network round-trip) **toplanır (birikir).** Monolith'te bu dört adım aynı process içinde, bellek üzerinden mikro-saniyeler içinde gerçekleşirdi; microservices'te her adım ayrı bir ağ isteğidir ve bu, toplam yanıt süresini ciddi biçimde artırabilir.

## 4. Test zorluğu

Her bir microservice için unit test yapmak nispeten kolay olsa da, **integration testing (entegrasyon testi) tipik olarak daha zorludur.** Microservices'in dağıtık (distributed) doğası, genellikle sistemin tamamını test etmenin **tüm üretim dağıtımını (entire production deployment) modellemeyi** gerektirdiği anlamına gelir.

> **Neden unit test ile integration test arasındaki bu fark önemli?** Çünkü BÖLÜM 5'te "microservices unit test için daha kolaydır" demiştik — bu hâlâ doğru, çünkü her servisin sınırı net. Ama bir kullanıcı senaryosunun **uçtan uca** doğru çalıştığını doğrulamak için, o senaryoya dahil olan **tüm** servislerin, aralarındaki ağ iletişiminin ve her birinin veritabanının **bir arada, gerçekçi biçimde** ayağa kaldırılması gerekir. Bu, monolith'te "uygulamayı bir kere ayağa kaldır, test et" kadar basit değildir.

## 5. Debugging zorluğu

Bir microservices mimarisini debug etmek de zor olabilir. Bir uygulama birçok microservice'ten oluşuyorsa ve **her bir microservice kendi loglarını** üretiyorsa, birçok microservice'i kapsayan çağrıları **izlemek (tracing)** zorlaşabilir.

> **Bu, BÖLÜM 6.1'deki operasyonel yük sorunuyla nasıl bağlantılı?** Çünkü "tutarlı loglama" ihtiyacı boşuna vurgulanmadı: eğer her microservice loglarını farklı bir formatta, farklı bir yerde tutuyorsa, tek bir kullanıcı isteğinin dört farklı servisten geçen yolculuğunu **yeniden kurmak** neredeyse imkânsız hale gelir. Standartlaştırılmış, merkezi bir loglama ve izleme (tracing) yaklaşımı olmadan, microservices'in "her biri basit" avantajı, sistemin bütünü söz konusu olduğunda **"anlaşılması imkânsız"** bir duruma dönüşebilir.

## Zorlukların ortak paydası: Otomasyon ve operasyonel mükemmellik

Ders bu bölümü şu cümleyle kapatıyor: **Microservices inşa etmek, otomasyona ve operasyonel mükemmelliğe (operational excellence) bir bağlılık gerektirir.**

Bu cümleyi hafife alma — bu, bu dersin belki de en önemli tek cümlesi. Yukarıda saydığımız beş zorluğun (operasyonel yük, iletişim karmaşıklığı, latency, test zorluğu, debugging zorluğu) **hepsi** aynı köke iniyor: **microservices, manuel süreçlerle yönetilemeyecek kadar çok hareketli parça yaratır.** Bu yüzden microservices'e geçmek, sadece bir mimari kararı değil, aynı zamanda **otomasyona yatırım yapma kararıdır.**

> **Sınav tuzağı:** "Microservices zorlukları, kötü tasarım yapan takımların başına gelir; iyi tasarlanmış bir microservices mimarisinde bu sorunlar olmaz" düşüncesi **yanıltıcıdır.** Ders bu zorlukları, microservices mimarisinin **doğasında var olan (inherent)** riskler olarak sunuyor — kötü tasarımın değil, **dağıtıklığın (distribution)** doğal sonucu. İyi tasarım ve güçlü otomasyon bu riskleri **azaltabilir**, ama tamamen **ortadan kaldıramaz.** Bu yüzden ders, microservices'in faydalarının zorluklarına ağır bastığını söylerken bile, zorlukları "acemi hatası" olarak değil, **yönetilmesi gereken kalıcı gerçekler** olarak çerçeveliyor.

**Zorluklar — özet tablo:**

| Zorluk | Ne anlama gelir? | Neden monolith'te yoktu (ya da daha azdı)? |
| --- | --- | --- |
| Operasyonel yük | Onlarca/yüzlerce/binlerce servisin build/test/deploy/log/security'si yönetilmeli | Monolith'te tek bir dağıtılabilir varlık vardı |
| İletişim karmaşıklığı | Servisler arası çağrıların "örümcek ağı" haline gelmesi | Monolith'te çağrılar aynı süreç içindeydi, dışarıdan görünmezdi |
| Latency | Ağ üzerinden çağrılar, aynı-süreç çağrılarından binlerce kat yavaş | Monolith'te bileşenler arası çağrı bellek içindeydi |
| Test zorluğu | Integration test, tüm production dağıtımını modellemeyi gerektirir | Monolith'te tek bir uygulamayı ayağa kaldırmak yeterliydi |
| Debugging zorluğu | Her servisin kendi logu var, çağrı izleme (tracing) zorlaşıyor | Monolith'te tek bir log kaynağı vardı |

---

# Üç Mimarinin Yan Yana Karşılaştırılması

Şimdi üç yaklaşımı yan yana koyup dersin anlattığı evrimi netleştirelim.

| Özellik | Monolithic Application | Service-Oriented Architecture (SOA) | Microservices |
| --- | --- | --- | --- |
| Yapı | UI + business logic + data access tek uygulamada | Ayrık iş fonksiyonlarını yerine getiren servisler | Kapsamı sınırlı, bağımsız servisler (kendi veritabanıyla) |
| İletişim | Aynı süreç içinde, doğrudan çağrı | Enterprise Service Bus (ESB) üzerinden mesajlaşma | Doğrudan API çağrıları (tipik olarak HTTP) |
| Koordinasyon | Merkezi (tek kod tabanı) | Merkezi (kurumsal standartlar, merkezi ESB takımı) | Ayrıştırılmış (decentralized) |
| Coupling | Sıkı bağlı (tightly coupled) | Servisler gevşek bağlı, ama ESB entegrasyonu darboğaz | Gevşek bağlı (loosely coupled) |
| Deploy birimi | Tüm uygulama | Servis + ESB entegrasyonu birlikte | Her microservice ayrı ayrı |
| Ölçeklendirme | Tüm uygulama birlikte ölçeklenir | Servis düzeyinde kısmen mümkün, ESB üzerinden sınırlı | Her servis bağımsız ölçeklenir |
| Ana zorluk | Karmaşıklık artan, sıkı bağlı kod; hata düzeltmek risklidir | Karmaşıklık ESB'ye kayar; ESB merkezi darboğaz olur | Operasyonel yük, iletişim karmaşıklığı, latency, test/debug zorluğu |
| En uygun olduğu durum | Domain bilgisi yok, küçük/basit uygulama, hızlı başlangıç | (Tarihsel bir ara aşama; bu derste "geçmiş zaman" olarak anlatılıyor) | Domain net, hızlı/agile teslimat şart, takım büyüyecek |

> **Bu tablodan çıkarılacak en önemli ders:** Her bir yaklaşım, bir öncekinin sorununu çözerken **karmaşıklığı yok etmiyor, sadece taşıyor ya da şeklini değiştiriyor.** Monolith'te karmaşıklık kod tabanının içindeydi (tight coupling). SOA'da karmaşıklık ESB entegrasyonuna kaydı (merkezi darboğaz). Microservices'te karmaşıklık **operasyona ve servisler arası iletişime** kaydı. Bu yüzden "microservices karmaşıklığı ortadan kaldırır" demek yanlıştır — microservices karmaşıklığı **farklı, daha dağıtık bir yere taşır**, ve bu yeni yeri yönetmek için otomasyon, gözlemlenebilirlik (observability) ve — tam olarak bu kursun konusu olan — **orchestration/choreography** teknikleri gerekir.

---

# Toplu Özet (Hızlı Tekrar)

**Modülün ana fikri:** Kurumsal uygulama mimarisi, tek parça (monolithic) uygulamalardan, kurumsal düzeyde standartlaştırılmış Service-Oriented Architecture'a, oradan da ayrıştırılmış (decentralized) microservices'e doğru evrildi. Her aşama, bir öncekinin sorununu çözerken kendi yeni sorununu getirdi; karmaşıklık hiçbir zaman tamamen yok olmadı, sadece yer değiştirdi.

**Monolithic Applications:** UI, business logic ve data access aynı büyük uygulamada, veriler tek bir ilişkisel veritabanında. Uygulama büyüdükçe kod tabanı kaçınılmaz olarak karmaşıklaşır; her büyük değişiklik karmaşıklığı daha da artırır. Kod tightly coupled olduğu için bakım zordur — bir hatayı düzeltmek yeni hatalar sokma riski taşır.

**Service-Oriented Architecture (SOA):** Monolith sorununu çözmek için kurumsal düzeyde uygulanan bir mimari stil. Servisler ayrık iş fonksiyonlarını yerine getirir, aralarındaki iletişim tanımlı arayüzler üzerinden mesajlaşmayla olur. Somut faydalar sağladı: küçük/loosely coupled servisler, küçük takımlar, servis yeniden kullanımı. Ama karışık sonuçlar üretti çünkü **karmaşıklık Enterprise Service Bus (ESB) entegrasyonuna kaydı.** ESB; bağlantı, güvenlik, mesaj yönlendirme ve dönüştürme yapan merkezi bir mesajlaşma ara katmanıydı. Merkezi takım tarafından yönetildiği, tüm takımların üzerinde çalışması gerektiği ve güncellemelerin diğer entegrasyonları bozma riski taşıdığı için ESB, başlı başına bir **darboğaza** dönüştü.

**Microservices:** Servisleri decompose etmek için ayrıştırılmış (decentralized) bir alternatif. Kapsamı sınırlı, tipik olarak kendi veritabanına sahip ayrı servisler (örnek: orders, products, reviews). Her servis bir API arayüzü sunar; diğer servisler bu arayüz üzerinden doğrudan çağrı yapar (merkezi bir ESB yok). Bu ayrılık, loose coupling'e yol açar — bu da bakımı, güncellemeyi ve deploy'u kolaylaştırır.

**Microservices mi, monolith mi:** En zor karar, servis sınırlarını (service boundaries) doğru çizmektir. Problem alanında uzmanlık yoksa **monolith ile başlamak** ve deneyim kazandıkça migrate etmek önerilir. Hızlı/agile teslimat gerekiyorsa ya da takım büyüyecekse (doğal servis sınırları küçük takımlara izin verdiği için), **microservices ile başlamak** mantıklıdır. Monolith ile başlansa bile, gelecekteki migrasyonu kolaylaştırmak için uygulama **modüler** tasarlanmalıdır.

**Microservices'in faydaları:** (1) Basitlik + takım odaklanması — küçük kod tabanı, net sınırlar sayesinde kolay unit test, ayrı deploy, agile geliştirme. (2) Dil/teknoloji özgürlüğü — servisler API üzerinden bağlandığı için her takım kendi teknolojisini seçebilir, platformlar arası çağrı mümkündür. (3) Bağımsız scaling — sadece ihtiyaç duyan servise kaynak ayrılır, maliyet trafiğe göre optimize edilir.

**Microservices'in zorlukları:** (1) Operasyonel yük — çok sayıda deploy edilebilir varlığın build/test/deploy/log/security/authorization'ı yönetilmeli, otomasyon hayati. (2) İletişim karmaşıklığı — kötü tasarlanmış sistemlerde servisler arası iletişim "örümcek ağına" dönüşür. (3) Latency — ağ üzerinden çağrılar aynı-süreç çağrılarından binlerce kat yavaş olabilir, birçok çağrı gerektiren operasyonlarda bu birikir. (4) Test zorluğu — unit test kolaydır ama integration test tüm production dağıtımını modellemeyi gerektirir. (5) Debugging zorluğu — her servisin kendi logu olduğu için çağrıları izlemek (tracing) zorlaşır. Ders, faydaların genelde zorluklara ağır bastığını, ama microservices inşa etmenin **otomasyona ve operasyonel mükemmelliğe bağlılık** gerektirdiğini vurgulayarak kapanıyor — ve service orchestration ile choreography'nin, tam olarak bu karmaşıklığı azaltmak için var olduğunu işaret ediyor.

---

# En Kritik Ayrımlar / Hızlı Tekrar (Cebinde Taşı)

- **Üç aşamanın rolü:** Monolith = tek parça, tightly coupled, karmaşıklık kod tabanında birikir. SOA = kurumsal, merkezi; karmaşıklık ESB entegrasyonuna kayar. Microservices = ayrıştırılmış (decentralized); karmaşıklık operasyona ve servisler arası iletişime kayar.
- **SOA'nın gerçek zayıflığı:** SOA "başarısız" değildi (somut faydalar sağladı: küçük/loosely coupled servisler, yeniden kullanım). Asıl sorun, karmaşıklığın **ESB entegrasyonuna kayması** ve ESB'nin merkezi bir darboğaza dönüşmesiydi.
- **SOA vs Microservices'in temel farkı:** SOA, servisler arası iletişimi **merkezi bir ESB** üzerinden yürütür ve kurumsal düzeyde standartlaştırılır. Microservices, servisler arası iletişimi **doğrudan API çağrılarıyla**, merkezi bir ara katman olmadan, ayrıştırılmış biçimde yürütür.
- **Monolith ile mi, microservices ile mi başlamalı — tuzak:** "Microservices her zaman üstündür" **yanlıştır.** Domain uzmanlığı yoksa monolith ile başla, deneyim kazandıkça migrate et. Hızlı/agile teslimat gerekiyorsa ya da takım büyüyecekse microservices ile başlamak mantıklıdır.
- **Monolith ile başlarken bile:** Uygulamayı **modüler** tasarla — bu, gelecekte microservices'e geçişi kolaylaştırır. "Monolith = kötü tasarlanmış kod" demek değildir.
- **Microservices'in üç ana faydası:** Basitlik/takım odaklanması (net sınırlar → kolay unit test, ayrı deploy), dil/teknoloji özgürlüğü (bağlantı API üzerinden, uygulama teknolojisi önemsiz), bağımsız scaling (sadece ihtiyaç duyan servise kaynak, maliyet optimizasyonu).
- **Microservices'in beş ana zorluğu:** Operasyonel yük, iletişim karmaşıklığı ("spider web"), latency (ağ çağrıları aynı-süreç çağrılarından binlerce kat yavaş), integration test zorluğu (tüm production'ı modellemek gerekir), debugging zorluğu (dağınık loglar, zor tracing).
- **Zorluklar tasarım hatası değil, doğal sonuçtur:** İyi tasarım ve otomasyon bu riskleri azaltır ama ortadan kaldırmaz — bu yüzden microservices "otomasyona ve operasyonel mükemmelliğe bir bağlılık" gerektirir.
- **Latency'nin kaynağı:** Monolith'te bileşenler arası çağrı aynı süreç/donanımda; microservices'te ağ üzerinden — bu, potansiyel olarak binlerce kat daha yavaştır ve çok adımlı iş operasyonlarında birikir.
- **Unit test vs integration test ayrımı:** Microservices'te unit test kolaydır (net sınırlar sayesinde); integration test zordur (dağıtık doğa, tüm production dağıtımını modelleme ihtiyacı).

---

> **Kapanış:** Bu ders, microservices'i bir "yeni moda mimari" olarak değil, **monolith'in ve SOA'nın çözemediği sorunlara verilen tarihsel bir cevap** olarak öğretti. Monolith'in tight coupling sorununu, SOA'nın kurumsal standartlarla çözmeye çalıştığını ama karmaşıklığı ESB'ye taşıyarak yeni bir darboğaz yarattığını, microservices'in ise bu merkezi ara katmanı kaldırıp servisleri doğrudan API'ler üzerinden konuşturan ayrıştırılmış bir yaklaşım sunduğunu gördün. En kritik pratik çıkarım şu: microservices **otomatik olarak** daha iyi bir seçim değildir — servis sınırlarını doğru çizecek domain bilgisine sahip değilsen, modüler bir monolith ile başlamak daha akıllıcadır. Microservices'e geçtiğinde ise, kazandığın basitlik, esneklik ve bağımsız scaling faydalarının karşılığında; operasyonel yük, iletişim karmaşıklığı, latency, test ve debugging zorluklarını yönetmek zorunda kalacaksın — ve bunun için otomasyona, gözlemlenebilirliğe ve iyi tasarlanmış servisler arası iletişim düzenine ihtiyacın olacak. İşte tam bu noktada, kursun geri kalanında göreceğin **service orchestration ve choreography** kavramları devreye giriyor. Bu doküman, o derslerin transkriptleri eklendikçe genişletilecek.
