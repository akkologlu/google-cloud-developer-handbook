# Event-Driven Applications — Baştan Sona Öğretici

> Bu metin, **"Event-Driven Applications"** dersinde anlatılan **her şeyi** kavratmak için yazıldı. Bu ders, "Service Orchestration and Choreography on Google Cloud" kursunun 2. modülüdür ve doğrudan 1. modülün ("Introduction to Microservices") bıraktığı yerden devam eder. O modülde, microservices mimarisinin beş zorluğundan biri olarak **"iletişim karmaşıklığı" (İletişim karmaşıklığı / "örümcek ağı" sorunu)** karşımıza çıkmıştı: servisler birbirini doğrudan çağırdıkça, kimin kiminle nasıl konuştuğunu takip etmek imkânsız hale gelen bir **point-to-point** ağ oluşuyordu. Bu ders, tam olarak bu soruna bir çözüm sunuyor: **event-driven architecture**. Modül, önce bir **event**'in ne olduğunu — ve bunun göründüğünden daha ince bir tanım olduğunu — öğretiyor, sonra bu event'lerin bir **event intermediary** aracılığıyla nasıl point-to-point bağımlılığı ortadan kaldırdığını gösteriyor, son olarak da event-driven mimarinin somut üç faydasını (merkezi auditing/access control, decoupling, asenkron işlemden gelen resilience) ve push-based messaging'in polling'e göre neden daha verimli olduğunu anlatıyor.

> **Kapsam notu:** Bu doküman, "Service Orchestration and Choreography on Google Cloud" kursunun **Modül 2 — Event-Driven Applications** dersini kapsıyor. Bu ders, kursun ismindeki "orchestration" ve "choreography" kavramlarına henüz girmiyor; sadece event-driven architecture'ın kavramsal temelini (event nedir, event intermediary nedir, hangi faydaları getirir) öğretiyor. Ayrıca bu derste **hiçbir spesifik Google Cloud ürünü isim olarak geçmiyor** — yani Pub/Sub, Eventarc, Cloud Tasks, Workflows gibi servisler bu modülde anlatılmıyor. Bu kavramsal çerçeve, kursun ilerleyen modüllerinde bu somut servislerle eşleştirilecek. Kursun sonraki modüllerinin transkriptleri eklendikçe, bu handbook'a `deep-dive/11-...`, `deep-dive/12-...` gibi ayrı numaralı modüller olarak eklenmeye devam edilecek.

---

## Bu ders tam olarak neyi öğretiyor ve neden önemli?

Modül 09'da öğrendiğin şeyi kısaca hatırlayalım, çünkü bu ders doğrudan onun üzerine inşa ediliyor. Microservices mimarisinde her servis kendi API'sini sunar ve diğer servisleri **doğrudan** çağırır. Bu, monolith'in tight coupling sorununu ve SOA'nın ESB darboğazını çözer — ama kendi yeni sorununu getirir: **her servis, konuşması gereken her downstream servisi bilmek zorundadır.** Servis sayısı arttıkça bu, modül 09'un tabiriyle bir **"örümcek ağına" (spider web)** dönüşür — kimin kimi çağırdığını, bir servisi değiştirdiğinde kimin etkileneceğini kestirmek gitgide zorlaşır.

Bu dersin açılış cümlesi bu sorunu doğrudan hedef alıyor: **Microservices kullanılarak inşa edilen birçok uygulama, event-driven architecture'dan fayda görür.** Peki bu mimari, örümcek ağı sorununu nasıl çözüyor? Cevap kısaca şu: **servisler birbirini doğrudan çağırmak yerine, aralarına bir aracı (event intermediary) koyar.** Üretici (producer) servisler bu aracıya event gönderir, tüketici (consumer) servisler bu aracıdan event alır — ve hiçbiri, karşı tarafın kim olduğunu bilmek zorunda değildir.

Ama derse bu aracı yapıyı anlatmadan önce, çok daha temel bir soruyla başlıyor: **Event tam olarak nedir?** Bu, ilk bakışta bariz görünen ama aslında üç ince ve sınav açısından kritik özelliği olan bir kavram. Ders bu yüzden üç ana durağı sırayla geziyor:

1. **Event nedir** — ve onu event yapan üç temel özellik.
2. **Point-to-point sorunundan event intermediary'ye geçiş** — producer ve consumer'ın nasıl birbirinden ayrıştığı (decouple).
3. **Event-driven mimarinin üç somut faydası** — merkezi auditing/access control, decoupling, ve asenkron işlemden gelen resilience — artı push-based messaging'in polling'e göre üstünlüğü.

Bu üçünü ayrı ayrı bilgi parçaları olarak değil, **tek bir mimari kararın sonuçları** olarak düşün: "servisler arasına bir aracı koy" kararını verdiğin anda, hem event'in doğasını anlaman gerekir hem de bu kararın seni nereye götürdüğünü (faydaları ve mekanizmayı) görmen gerekir.

---

# BÖLÜM 1 — Event Nedir? Göründüğünden Daha İnce Bir Tanım

## Basit tanım

Ders, event'i şöyle tanımlıyor: **Bir event, gerçekleşmiş bir şeyin kaydıdır (a record of something that has happened).** Örnekler somut: bir çalışanın (employee) bir uygulamaya giriş yapması (logging in), ya da bir ürünün (product) bir alışveriş sepetine (shopping cart) eklenmesi.

Ders burada dürüst bir itirafta bulunuyor: **bu tanım bariz görünebilir.** Gerçekten de, "olan bir şeyin kaydı" dediğinde kulağa yeni bir fikir gibi gelmiyor. Ama event-driven architecture'ı tartışırken, event'in **başka önemli özellikleri** var — ve asıl mesele burada başlıyor.

> **Analoji:** Bir event'i, bir **günlük (diary) defterine yazılmış, tarihli bir kayıt** gibi düşün. "12 Mart, saat 14:03 — müşteri X, sepete Y ürününü ekledi" cümlesini günlüğüne yazdığında, bu cümle artık **geçmişte olmuş bir şeyin sabit kaydıdır.** Ertesi gün müşteri fikrini değiştirip ürünü sepetten çıkarsa, bu **yeni bir günlük kaydı** olur ("13 Mart — müşteri X, sepetten Y ürününü çıkardı") — eski kaydı silmez ya da değiştirmezsin. Geçmiş, geçmiş olarak kalır.

## Özellik 1 — Event, immutable bir gerçektir (immutable fact)

Ders'in vurguladığı ilk özellik: **bir event, tipik olarak immutable (değiştirilemez) bir gerçek olarak ele alınır.** Bu, bir olayın (occurrence) **tarihsel bir kaydıdır** ve **değiştirilmemeli ya da silinmemelidir.**

> **Neden bu kadar önemli?** Çünkü bir event, "şu anki durum"u değil, "şu anda gerçekleşmiş olan şey"i temsil eder. Bir veritabanı satırını (row) güncelleyebilirsin çünkü o satır "şu anki durumu" tutar. Ama bir event'i güncelleyemezsin çünkü event zaten **geçmişte kalmış bir gerçektir** — "müşteri sepete ürün ekledi" olayı, müşteri daha sonra fikrini değiştirse bile, **gerçekten oldu.** Bunu değiştirmek, tarihi yeniden yazmaya çalışmak gibi olur. Eğer durum değişirse, bunun için **yeni bir event** üretilir; eskisi silinmez.

## Özellik 2 — Event, hiç tüketilmese (consume edilmese) bile üretilebilir

İkinci özellik daha az sezgisel: **bir event, hiç consume edilmese bile üretilebilir (generated).** Ders bunu açıkça söylüyor: **event üreten birçok uygulama, ürettiği event'lerin hiç consume edilip edilmediğini bilmez.**

> **Neden bu sezgiye aykırı geliyor?** Çünkü senkron (request/response) dünyasına alışkınsan, "bir şey gönderdiysen, biri onu almış ve işlemiş olmalı" diye düşünürsün — çünkü senkron bir çağrıda, yanıt (response) almazsan bunu **anında** fark edersin. Ama event-driven dünyada producer, event'i üretip intermediary'ye gönderdiği anda **işini bitirmiştir.** O event'in şu anda kaç consumer tarafından okunduğu, hiç okunup okunmadığı ya da ne zaman okunacağı, **producer'ın umurunda bile değildir** — ve olması da gerekmez. Bu, producer ve consumer'ı birbirinden ayıran (decouple eden) temel mekanizmalardan biridir; BÖLÜM 2'de bunu daha derinlemesine göreceğiz.

> **Sınav tuzağı:** "Hiç consume edilmeyen bir event, sistemde bir hata (bug) ya da kayıp veridir" düşüncesi **yanlıştır.** Ders açıkça, birçok producer'ın event'lerinin hiç tüketilip tüketilmediğini **bilmediğini** söylüyor — bu, tasarım gereği böyledir, bir arıza belirtisi değildir. Event-driven mimaride bir event, tüketilmek **için garanti edilmiş** bir şey değildir; tüketilebilir **olacak şekilde** üretilmiş bir kayıttır. Sınav sorusu "producer, event'inin consume edildiğini nasıl bilir?" diye sorarsa, doğru cevap genelde "bilmesi gerekmez / bu onun sorumluluğunda değildir" yönündedir.

## Özellik 3 — Event süresiz saklanabilir ve birden çok kez tüketilebilir

Üçüncü özellik: **bir event süresiz (indefinitely) olarak saklanabilir (persisted) ve gerektiği kadar çok kez tüketilebilir.** Tek bir event, **birçok servis tarafından** tüketilebilir — bu da event işlemenin (event processing) **paralel** gerçekleşmesine izin verir.

> **Neden bu, senkron bir çağrıdan temelden farklı?** Senkron bir HTTP isteğini düşün: bir istemci bir sunucuya istek gönderir, sunucu bir yanıt döner, ve bu etkileşim **biter.** Aynı isteği "tekrar oynatamazsın" ya da başka on tane servisin **aynı isteği** dinlemesini sağlayamazsın — çünkü o bir istektir, bir kayıt değildir. Bir event ise **kalıcı bir kayıt** olduğu için, bugün bir consumer tarafından okunabilir, yarın yeni eklenen başka bir consumer tarafından da (event hâlâ saklandığı için) okunabilir, ve aynı anda beş consumer tarafından **paralel olarak** okunabilir. Bu özellik, BÖLÜM 4'te göreceğin "yeni consumer'lar mevcut servisleri değiştirmeden eklenebilir" faydasının ve BÖLÜM 5'te göreceğin "replay/redeliver" mekanizmasının **temelini** oluşturur.

**Event'in üç özelliği — özet tablo:**

| Özellik | Ne anlama gelir? | Neden önemli? |
| --- | --- | --- |
| Immutable fact | Event değiştirilmez, silinmez; geçmişte olmuş bir şeyin tarihsel kaydıdır | Durum değişirse yeni event üretilir, eskisi asla güncellenmez |
| Consume edilmeden de üretilebilir | Producer, event'inin tüketilip tüketilmediğini bilmeyebilir/umursamayabilir | Producer ve consumer'ı birbirinden bağımsızlaştırmanın temelidir |
| Süresiz saklanabilir, çoklu/paralel tüketilebilir | Aynı event birden fazla servis tarafından, farklı zamanlarda okunabilir | Yeni consumer ekleme ve replay/redeliver mekanizmalarını mümkün kılar |

---

# BÖLÜM 2 — Point-to-Point Sorunundan Event Intermediary'ye

## Hatırlatma: örümcek ağı sorunu

Ders, bu bölüme tam olarak modül 09'daki tartışmaya atıfla giriyor: **"Daha önce tartıştığımız gibi, microservices arasındaki point-to-point iletişimin 'örümcek ağı' bir zorluk olabilir."** Her servis, konuşması gereken **her** downstream servisle nasıl iletişim kuracağını bilmek zorundadır. Ve point-to-point iletişim, servisler arasında **coupling** (bağımlılık) doğurma eğilimindedir.

> **Bunu neden tekrar hatırlatıyoruz?** Çünkü bu ders, modül 09'un bitirdiği yerden başlıyor: orada "microservices'in zorlukları" listesinde iletişim karmaşıklığı bir **sorun** olarak bırakılmıştı, çözümü değil. Bu modül, tam olarak o sorunun **çözümüne** giriş yapıyor. Yani event-driven architecture'ı, "yeni ve havalı bir mimari" olarak değil, **belirli bir sorun için tasarlanmış belirli bir çözüm** olarak öğrenmen gerekiyor.

## Çözüm: Aradaki aracı — event intermediary

Event-driven architecture, servislerin arasına bir **event intermediary** yerleştirir. Bu, mekanizmanın kalbidir:

- Bir servis **event producer** rolünü üstlendiğinde, event'lerini **intermediary'ye** gönderir. Servisin, bu event'leri kimlerin tükettiğini bilmesine **gerek yoktur.**
- Bir servis aynı zamanda **event consumer** rolünü de üstlenebilir; bu durumda event'leri **intermediary'den** alır. Event consumer'lar, bir event'i **nasıl işleyeceğini bilir** — ama event'in **producer'ı hakkında herhangi bir ayrıntı bilmek zorunda değildir.**

> **Analoji:** Event intermediary'yi bir **posta kutusu sistemi (postane)** gibi düşün — ama modül 09'daki ESB analojisinden (merkezi kontrol kulesi) bilinçli olarak **farklı** bir posta sistemi. ESB'de her mesaj, tek bir merkezi kuleden geçmek ve kule tarafından **yönlendirilmek (route edilmek)** zorundaydı; kule tıkanırsa her şey tıkanırdı. Event intermediary'de ise, bir postanenin adres etiketli mektup kutuları sistemine benziyor: bir gönderici (producer), bir mektubu (event) belirli bir **konuya (topic/kategoriye)** göre postaneye bırakır ve işi biter — mektubun kime, kaç kişiye gideceğiyle **ilgilenmez.** İlgilenen herkes (consumer) o kutuyu **kendi isteğiyle** kontrol eder, mektubu okur (hatta kopyasını alır) ve gönderenin kim olduğunu bilmeden işine bakar. Postane (intermediary), gönderici ile alıcı arasında bir **köprü** kurar ama gönderici ile alıcıyı **doğrudan tanıştırmaz.**

> **Sınav tuzağı:** Event intermediary'yi modül 09'daki **Enterprise Service Bus (ESB)** ile aynı şey sanmak yaygın bir hatadır — ikisi de "ortada bir aracı katman var" gibi göründüğü için karışabilir. Ama aralarında kritik bir fark var: ESB, SOA'da servisler arasındaki mesajları **merkezi bir takım tarafından yönetilen, protokol dönüştüren, mesaj yönlendiren tek bir darboğaz noktası** olarak çalışıyordu — modül 09'da gördüğün gibi, ESB'nin kendisi büyüdükçe **merkezi bir monolith'e** dönüşüyordu. Event intermediary ise doğası gereği **producer ile consumer'ı ayrıştırmak (decouple etmek)** için var; bir event, tanımı gereği **birden fazla, önceden bilinmeyen** consumer tarafından tüketilebilir (BÖLÜM 1, Özellik 3) ve yeni consumer'lar mevcut hiçbir şeyi değiştirmeden eklenebilir (BÖLÜM 4). ESB'de merkezi takım her yeni entegrasyonu **elle** kurup yönetiyordu; event intermediary'de yeni bir consumer, sadece ilgilendiği event'i dinlemeye **kendi başına** başlar. Sınav sorusu "event intermediary ile ESB aynı kavram mıdır?" diye sorduğunda cevap **hayır**dır — ikisi de "aradaki aracı" olsa da, coupling'i azaltma biçimleri ve mimari felsefeleri farklıdır.

## Neden bu, örümcek ağını çözüyor?

Point-to-point modelde N servis varsa, potansiyel bağlantı sayısı servis sayısıyla birlikte **çok hızlı** büyür — her servis, konuşması gereken her diğer servisi bilmek zorundadır. Event intermediary modelinde ise her servis sadece **tek bir noktayla** (intermediary'yle) konuşur: producer olarak event gönderir, consumer olarak event alır. Servis sayısı artsa bile, her servisin bilmesi gereken şey **değişmez**: sadece intermediary'nin varlığını ve ilgilendiği event'lerin formatını bilmesi yeterlidir.

> **Bunu neden BÖLÜM 4'te tekrar göreceğiz?** Çünkü bu gözlem, ders'in "decoupling" faydasının **tam kalbidir.** Burada mekanizmayı (nasıl çalıştığını) gördün; BÖLÜM 4'te bunun somut sonucunu (yeni consumer eklemenin ne kadar kolaylaştığını) göreceksin.

---

# BÖLÜM 3 — Fayda 1: Merkezi Auditing ve Access Control

Event-driven uygulamaların ilk somut faydası, **merkezi bir event service'in**, dağıtık (distributed) bir uygulama için auditing ve access control'ü **basitleştirmesidir.**

## Auditing: immutable event log'un doğal sonucu

BÖLÜM 1'de gördüğün ilk özelliği hatırla: bir event, **immutable bir gerçektir**, değiştirilmez ya da silinmez. Bunun doğrudan sonucu: **immutable event'lerden oluşan bir log, auditing (denetim) amacıyla kullanılabilir.** Event'ler, bir uygulamanın state'indeki (durumundaki) **her değişikliğin, zaman damgalı ve sıralı (timed, ordered) bir kaydını** sağlayabilir.

> **Neden bu değerli?** Çünkü "sistemde ne oldu, ne zaman oldu, hangi sırayla oldu" sorularına cevap vermek, geleneksel bir "şu anki durumu" tutan veritabanında **zordur** — çünkü o veritabanı sadece **son hali** gösterir, geçmişi göstermez. Bir event log'u ise, tanımı gereği geçmişteki her adımı **sırasıyla ve değiştirilmemiş halde** saklar. Bu, örneğin "bu sipariş nasıl bu duruma geldi?" ya da "hangi kullanıcı, ne zaman, hangi veriye erişti?" gibi soruların cevabını **doğrudan event log'undan** okuyabilmen anlamına gelir — event log'u kendi başına bir denetim izidir (audit trail).

## Access control: tek bir noktadan yetkilendirme

Merkezi bir event service, aynı zamanda **belirli servislere ve veriye erişimi kontrol etmene** de yardımcı olur. Event service seviyesinde **authentication (kimlik doğrulama) ve authorization (yetkilendirme)** zorunlu kılarak, her bir event-tabanlı servise erişimi **sınırlayabilirsin.**

> **Analoji:** Bunu bir binanın tek bir güvenlikli girişi olması gibi düşün. Binadaki her odaya (servise) ayrı ayrı kilit ve güvenlik görevlisi koymak yerine, **girişte** tek bir kimlik kontrolü yaparsın; buradan geçen herkesin kim olduğu ve nereye gitmeye yetkili olduğu zaten **kayıt altındadır.** Event intermediary de böyle çalışır: producer ve consumer'lar event service'e bağlanmak için kimlik doğrulaması yapar, bu da hem "kim ne gönderdi" hem de "kim neyi okuyabilir" sorularının **tek bir merkezi noktadan** yönetilmesini sağlar — her bir servisin kendi başına, birbirinden bağımsız bir yetkilendirme mantığı icat etmesine gerek kalmaz.

> **Bunu ESB ile karıştırma:** Burada "merkezi" kelimesi geçtiği için BÖLÜM 2'deki ESB uyarısını hatırlamakta fayda var. Event service'in auditing/access control için merkezi olması, onu ESB gibi bir **darboğaza** dönüştürmez — çünkü burada merkezileşen şey **yönlendirme mantığı ve protokol dönüştürme** değil (ESB'nin yaptığı buydu), sadece **kimlik doğrulama ve immutable log tutma** işidir. Producer'lar ve consumer'lar birbirinden hâlâ tamamen bağımsızdır; sadece hepsi aynı güvenlik kapısından geçer.

---

# BÖLÜM 4 — Fayda 2: Decoupling — Yeni Consumer'ları Serbestçe Ekleyebilme

## Producer ve consumer birbirinden bağımsızdır

Bir event intermediary kullanıldığında, bir event'in producer'ı ve consumer'ı **birbirinden ayrışır (decoupled).** Bunun iki yönü var:

- Servisler, event'i tüketecek herhangi bir servise **doğrudan istek göndermeden** bir event **oluşturabilir.**
- Servisler, event'i üreten servis hakkında **hiçbir şey bilmeden** bir event'i **tüketebilir.**

Bu decoupling'in doğrudan sonucu: artık **point-to-point örümcek ağı yoktur.** Her event, event intermediary üzerinden geçer; intermediary de event'i **doğru consumer'a ya da consumer'lara** yönlendirir. Bir producer ya da consumer'ın bilmesi gereken tek şey, **ilgili event'in formatıdır** — karşı tarafın kim olduğu, nerede çalıştığı ya da nasıl uygulandığı **önemli değildir.**

> **Analoji:** BÖLÜM 2'deki postane analojisini burada bir adım ileri götürelim. Point-to-point dünyasında, bir gönderici her alıcıya **ayrı ayrı, kendi elleriyle** mektup götürmek zorundaydı — beş alıcı varsa beş farklı yolculuk gerekiyordu, ve alıcı sayısı arttıkça bu iş katlanarak zorlaşıyordu (BÖLÜM 2'deki örümcek ağı). Event intermediary'de gönderici, mektubu **bir kere** postane kutusuna bırakır; postane kutusunu kimin, kaç kişinin takip ettiği göndericiyi ilgilendirmez. Aynı şekilde, postaneye **yeni abone olan** biri, geçmişte gönderilmiş mektupları (event'ler süresiz saklanabildiği için, BÖLÜM 1) okuyabilir — gönderici bundan **haberdar bile olmaz**, hele ki bir şey **değiştirmesi** hiç gerekmez.

## Ekstra fayda: yeni consumer'lar mevcut servisleri etkilemeden eklenebilir

Bu decoupling'in "ekstra bir faydası" olarak ders şunu vurguluyor: **uygulamana yeni event consumer'ları, mevcut hiçbir servisi değiştirmeden ekleyebilirsin.**

> **Neden bu bu kadar değerli?** Point-to-point bir dünyada, yeni bir servisin bir event'e ihtiyacı olduğunda, o event'i **üreten** servise gidip "bana da gönderir misin?" demen, o servisin kodunu **değiştirmen** gerekirdi — bu da BÖLÜM 6'nın (modül 09) bahsettiği kırılganlık riskini beraberinde getirirdi: bir servisi değiştirmek, ona bağlı diğer her şeyi etkileme riski taşır. Event-driven dünyada ise yeni bir consumer, sadece intermediary'ye bağlanıp **ilgilendiği event'i dinlemeye başlar.** Producer'ın kodunda **tek bir satır bile** değişmez, çünkü producer zaten "kim dinliyor" sorusuyla hiç ilgilenmiyordu (BÖLÜM 1, Özellik 2'yi hatırla). Bu, bir uygulamaya **yeni özellik eklemenin maliyetini** kökten değiştirir: yeni bir analytics servisi, bir bildirim (notification) servisi ya da bir raporlama servisi eklemek istediğinde, mevcut hiçbir şeye dokunmadan sadece **dinlemeye başlarsın.**

**Point-to-point vs. event intermediary — özet tablo:**

| Özellik | Point-to-point iletişim | Event intermediary üzerinden iletişim |
| --- | --- | --- |
| Producer neyi bilmeli? | Kendisini çağıran her downstream servisi | Sadece göndereceği event'in formatını |
| Consumer neyi bilmeli? | Event'i/isteği üreten servisin kim olduğunu, nasıl çağrılacağını | Sadece dinlediği event'in formatını |
| Yeni bir consumer eklemek | Producer'ın kodunu/entegrasyonunu değiştirmeyi gerektirebilir | Mevcut hiçbir servisi değiştirmeden yapılabilir |
| Servis sayısı arttıkça | Bağlantı karmaşıklığı hızla büyür ("örümcek ağı") | Her servis yalnızca intermediary ile konuşur, karmaşıklık artmaz |
| Coupling | Sıkı (point-to-point coupling) | Gevşek (producer/consumer birbirinden decouple) |

---

# BÖLÜM 5 — Fayda 3: Asenkron İşlemden Gelen Resilience

## Senkron dünyanın kırılganlığı

Microservices uygulamaları bazen **senkron request/response çağrıları** kullanacak şekilde tasarlanır. Bu modelde, bir servisin sağlığı (health), **doğrudan ya da dolaylı olarak** çağırdığı servislerin sağlığından **etkilenir.** Ders'in çok net ifadesiyle: **tek bir servis başarısız olduğunda, bu tüm uygulamayı çökertebilir.**

> **Analoji:** Bunu bir domino taşı dizisine benzet. Senkron bir çağrı zincirinde (A servisi B'yi çağırır, B servisi C'yi çağırır, C de D'yi çağırır), her taş bir sonrakine **doğrudan bağlıdır.** D taşı devrilmezse (D servisi yanıt vermezse), C taşı da bekler, bekler, sonunda o da devrilir (timeout ya da hata verir); bu da B'yi, B de A'yı etkiler. Zincirdeki **tek bir zayıf halka**, tüm diziyi devirebilir.

Event-driven bir mimaride ise, event'ler **asenkron olarak üretilir** — event'ler, bir **yanıt beklemeden** oluşturulur. Bu, mimarinin bir servisin **geçici kaybını (temporary loss)** atlatacak şekilde tasarlanabilmesi anlamına gelir. Sağlıksız (unhealthy) bir servise gönderilen event'ler, o servis **tekrar ayağa kalktığında** replay edilebilir (yeniden oynatılabilir) ya da redeliver edilebilir (yeniden teslim edilebilir). Bu asenkron event işleme biçimi, **daha resilient (dayanıklı) uygulamalara** yol açar.

> **Analoji:** Şimdi aynı zinciri, bağımsız posta kutularına (event intermediary) dönüştürelim. A servisi bir event üretip postane kutusuna bırakır ve **işine devam eder** — D servisinin şu anda çalışıp çalışmadığını beklemez. D servisi geçici olarak çöktüyse, onun için bırakılmış event'ler **postane kutusunda beklemeye devam eder** (BÖLÜM 1, Özellik 3 — event'ler süresiz saklanabilir). D servisi tekrar ayağa kalktığında, bıraktığı yerden devam eder: bekleyen event'leri okur, işler. A, B ve C servislerinin bu süreçte **hiçbir şey fark etmesine bile gerek yoktur** — çünkü onlar zaten D'nin yanıtını **beklemiyorlardı.**

> **Neden bu, gerçek bir mimari üstünlük ve sadece "iyi bir yan etki" değil?** Çünkü senkron modelde, bir servisin geçici olarak çökmesi **anlık bir veri kaybına ya da kullanıcıya yansıyan bir hataya** dönüşür — çağrı zaten yapıldı, yanıt gelmedi, istek başarısız oldu, iş bitti. Event-driven modelde ise, "geçici çökme" durumu **kalıcı bir sorun değildir** çünkü event, hâlâ orada durur ve servis geri geldiğinde işlenmeyi bekler. Bu, hata toleransını (fault tolerance) mimarinin **doğasına** gömer; her servisin kendi retry (yeniden deneme) mantığını elle yazmasına gerek kalmaz.

> **Sınav tuzağı:** "Asenkron işlem, event-driven mimariyi senkron mimariden **her açıdan** daha iyi yapar" gibi bir genelleme **dikkatli ele alınmalıdır.** Ders burada spesifik bir fayda anlatıyor — **resilience (dayanıklılık)**, yani bir servisin geçici kaybını atlatabilme — genel bir "her zaman daha iyi" iddiası değil. Sınav sorusu "event-driven mimarinin senkron modele göre resilience avantajı nereden gelir?" diye sorarsa, doğru cevap **"event'lerin asenkron üretilmesi ve unhealthy bir servise giden event'lerin replay/redeliver edilebilmesi"**dir — "event-driven her zaman daha hızlıdır" gibi ilgisiz bir cevap değil.

**Senkron request/response vs. event-driven (asenkron) — özet tablo:**

| Özellik | Senkron request/response | Event-driven (asenkron) |
| --- | --- | --- |
| Çağıran servisin sağlığı | Çağırdığı servislerin sağlığına doğrudan bağlı | Event intermediary'nin ayakta olmasına bağlı, hedef consumer'ın anlık durumuna değil |
| Bir servis çöktüğünde | Zincirdeki diğer servisleri de etkileyebilir, tüm uygulamayı çökertebilir | Sadece o servise ait event'ler birikir, diğer servisler etkilenmez |
| Servis geri geldiğinde | Kaybedilen istekler genelde kaybolmuştur | Bekleyen event'ler replay/redeliver edilerek işlenir |
| Üreten taraf ne bekler? | Yanıt (response) | Hiçbir şey — event'i gönderip devam eder |

---

# BÖLÜM 6 — Push-Based Messaging vs. Polling

## Neden bu konu burada geçiyor?

Servisler asenkron çalıştığında, consumer'ların yeni event'lerden **nasıl haberdar olacağı** ayrı bir tasarım sorusudur. Ders bu soruyu iki modelle cevaplıyor: **push-based messaging** ve **polling.**

## Polling: sürekli sorup durmak

Polling modelinde, consumer'lar **yapılacak bir iş olup olmadığını anlamak için sürekli olarak (continuously) poll etmek** (sorgulamak) zorundadır. Ders'in net değerlendirmesi: polling tipik olarak **artan network I/O'ya ve gereksiz gecikmelere (unnecessary delays)** yol açar.

> **Analoji:** Polling'i, bir paket kargosunu beklerken kargo şirketini **her beş dakikada bir arayıp** "paketim geldi mi, geldi mi, geldi mi?" diye sormaya benzet. Her arama bir kaynak (senin zamanın, kargo şirketinin operatörünün zamanı — event-driven dünyada bu bir network isteği) tüketir, ve çoğu aramanın cevabı **"hayır, henüz gelmedi"** olur. Paket saat 14:03'te geldiyse ama sen son kez 14:00'da aradıysan, bunu ancak **14:05'teki bir sonraki aramanda** öğrenirsin — yani ortada gereksiz bir **gecikme (delay)** de vardır.

## Push: haber verilmesini beklemek

Push-based bir modelde ise, **consumer'lar tükettikleri event'ler olduğunda otomatik olarak bilgilendirilir (notified).** Ders'in ifadesiyle: push-based messaging, client'ların **sürekli olarak uzak servisleri poll etmesine gerek kalmadan** güncellemeleri almasını sağlar; event'ler consumer'lara **verimli bir şekilde yönlendirilir (routed).**

> **Analoji:** Push'u, aynı kargo örneğinde, kargo şirketinin paket kapına bırakıldığı **an sana bir bildirim (notification)** göndermesine benzet. Sen hiçbir şey sormazsın; paket geldiğinde **haberin olur**, ne bir saniye erken ne bir saniye geç. Ne senin zamanın (tekrar tekrar sormak) ne de kargo şirketinin kaynağı (her aramaya cevap vermek) boşa harcanır.

> **Neden bu fark, ölçek büyüdükçe daha da önemli hale gelir?** Tek bir consumer için polling'in maliyeti önemsiz görünebilir — birkaç saniyede bir sorgu atmak kimseyi yormaz. Ama yüzlerce consumer, binlerce event kaynağını aynı anda poll ediyorsa, bu **sürekli, çoğu zaman boşa** giden network trafiğine dönüşür — sunucular sürekli "hayır, yeni bir şey yok" cevabı üretmekle meşgul olur. Push modelinde ise trafik, **sadece gerçekten bir event olduğunda** üretilir; bu da hem network I/O'yu hem de gecikmeyi doğrudan azaltır.

**Push-based messaging vs. polling — özet tablo:**

| Özellik | Polling | Push-based messaging |
| --- | --- | --- |
| Kim harekete geçer? | Consumer, sürekli sorgu atar | Intermediary, event oluştuğunda consumer'ı bilgilendirir |
| Yeni bir event olmadığında | Yine de network isteği gider (boşa) | Hiçbir trafik oluşmaz |
| Gecikme (delay) | Bir sonraki poll döngüsüne kadar gecikme olur | Event oluştuğu an consumer'a yönlendirilir |
| Network I/O | Sürekli, çoğu zaman gereksiz | Sadece gerçek event olduğunda |

---

# Toplu Özet (Hızlı Tekrar)

**Modülün ana fikri:** Modül 09'da tanımlanan "microservices arasındaki point-to-point iletişimin örümcek ağı" sorununa, bu modülde somut bir çözüm getirildi: **event-driven architecture.** Bu mimari, producer ve consumer servislerin arasına bir **event intermediary** yerleştirerek, servislerin birbirini doğrudan tanımasına gerek bırakmaz.

**Event nedir (BÖLÜM 1):** Bir event, gerçekleşmiş bir şeyin kaydıdır (örn. bir kullanıcının login olması, sepete ürün eklenmesi). Üç kritik özelliği vardır: (1) **immutable fact** — değiştirilmez, silinmez, tarihsel bir kayıttır; (2) hiç **consume edilmeden de üretilebilir** — producer'lar genelde event'lerinin tüketilip tüketilmediğini bilmez/umursamaz; (3) **süresiz saklanabilir ve birden çok kez, paralel olarak tüketilebilir** — tek bir event birçok servis tarafından okunabilir.

**Point-to-point'ten event intermediary'ye (BÖLÜM 2):** Point-to-point iletişimde her servis, konuşacağı her downstream servisi bilmek zorundadır — bu, coupling ve "örümcek ağı" yaratır. Event-driven mimari, servisler arasına bir event intermediary koyar: producer'lar event'lerini intermediary'ye gönderir (kimin tükettiğini bilmeden), consumer'lar event'lerini intermediary'den alır (kimin ürettiğini bilmeden). Bu, event intermediary'yi ESB'den ayıran temel noktadır — ESB merkezi bir yönlendirme/dönüştürme darboğazıyken, event intermediary producer ve consumer'ı birbirinden **decouple** etmek için tasarlanmıştır.

**Fayda 1 — Merkezi auditing ve access control (BÖLÜM 3):** Immutable event log'u, uygulama state'indeki her değişikliğin zaman damgalı, sıralı bir kaydı olarak auditing amacıyla kullanılabilir. Event service seviyesinde authentication ve authorization zorunlu kılınarak, servislere ve veriye erişim merkezi bir noktadan kontrol edilebilir.

**Fayda 2 — Decoupling (BÖLÜM 4):** Producer ve consumer birbirinden ayrışır; hiçbiri karşı tarafı doğrudan bilmek zorunda değildir, sadece event'in formatını bilmesi yeterlidir. Bunun ekstra bir sonucu: yeni event consumer'ları, mevcut hiçbir servisi değiştirmeden eklenebilir.

**Fayda 3 — Asenkron işlemden gelen resilience (BÖLÜM 5):** Senkron request/response modelinde bir servisin sağlığı, çağırdığı servislerin sağlığına bağlıdır; tek bir servisin çökmesi tüm uygulamayı devirebilir (domino etkisi). Event-driven modelde event'ler yanıt beklemeden, asenkron üretilir; unhealthy bir servise giden event'ler, servis geri geldiğinde replay/redeliver edilebilir — bu da mimariyi geçici servis kayıplarına karşı dayanıklı (resilient) hale getirir.

**Push-based messaging vs. polling (BÖLÜM 6):** Polling'de consumer'lar sürekli sorgu atarak iş olup olmadığını kontrol eder — bu, artan network I/O ve gereksiz gecikmelere yol açar. Push-based modelde consumer'lar event oluştuğunda otomatik olarak bilgilendirilir; event'ler verimli bir şekilde yönlendirilir, gereksiz trafik ve gecikme oluşmaz.

---

# En Kritik Ayrımlar / Hızlı Tekrar (Cebinde Taşı)

- **Event'in üç özelliği:** Immutable fact (değişmez/silinmez) + consume edilmeden de üretilebilir (producer umursamayabilir) + süresiz saklanır ve çoklu/paralel tüketilebilir. Sınav bu üç özelliği ayrı ayrı, tek tek sorabilir.
- **"Hiç tüketilmeyen event = bug" tuzağı:** Yanlıştır. Producer'ların event'lerinin tüketilip tüketilmediğini bilmemesi, tasarımın **doğal bir parçasıdır**, arıza belirtisi değildir.
- **Event intermediary ≠ ESB:** İkisi de "aradaki aracı" gibi görünse de, ESB merkezi bir yönlendirme/dönüştürme darboğazıyken (modül 09), event intermediary producer ile consumer'ı **decouple etmek** için vardır; yeni consumer'lar merkezi bir takımın müdahalesi olmadan kendi başına eklenebilir.
- **Point-to-point vs. event intermediary:** Point-to-point'te her servis, konuştuğu her diğer servisi bilmek zorundadır (örümcek ağı). Event intermediary'de her servis sadece intermediary'yi ve ilgilendiği event'in formatını bilir.
- **Üç fayda kısaca:** (1) Merkezi auditing/access control — immutable log + tek noktadan authn/authz. (2) Decoupling — producer/consumer birbirini bilmez, yeni consumer eklemek mevcut servisleri etkilemez. (3) Resilience — event'ler asenkron üretilir, unhealthy servise giden event'ler replay/redeliver edilebilir, tek bir servisin çökmesi tüm sistemi devirmez.
- **Senkron vs. event-driven resilience farkı:** Senkron modelde bir servisin sağlığı, çağırdığı servislerin sağlığına bağlıdır (domino etkisi). Event-driven modelde producer yanıt beklemez; hedef servis geçici olarak çökse bile event'ler bekler ve servis geri geldiğinde işlenir.
- **Push vs. polling:** Polling, consumer'ın sürekli "iş var mı?" diye sorması demektir — artan network I/O ve gecikme getirir. Push, event oluştuğunda consumer'ın otomatik bilgilendirilmesi demektir — gereksiz trafik ve gecikme olmaz.
- **Bu ders henüz hiçbir GCP ürünü anlatmadı:** Pub/Sub, Eventarc, Cloud Tasks, Workflows gibi servisler bu modülde **isim olarak geçmiyor.** Bu ders, sadece event-driven architecture'ın kavramsal çerçevesini kuruyor; somut servisler kursun ilerleyen modüllerinde işlenecek.

---

> **Kapanış:** Bu ders, modül 09'da bıraktığın yerden devam ederek, microservices'in en somut zorluklarından biri olan **point-to-point iletişim karmaşıklığına** kavramsal bir çözüm sundu: event-driven architecture. Bir event'in sadece "olan bir şeyin kaydı" olmadığını, aynı zamanda **immutable, tüketilmesi garanti edilmeyen ve süresiz/çoklu tüketilebilen** bir yapı olduğunu gördün. Bu üç özelliğin, event intermediary'nin producer ve consumer'ı nasıl birbirinden ayrıştırdığının **temelini** oluşturduğunu, ve bu ayrışmanın üç somut faydaya (merkezi auditing/access control, decoupling, asenkron resilience) nasıl dönüştüğünü öğrendin. Son olarak, push-based messaging'in polling'e göre neden daha verimli bir bildirim mekanizması olduğunu gördün. Bu doküman, kursun sonraki modülleri (orchestration ve choreography'nin somut tekniklerini ve Google Cloud servislerini anlatacak modüller) eklendikçe genişletilmeye devam edecek.
