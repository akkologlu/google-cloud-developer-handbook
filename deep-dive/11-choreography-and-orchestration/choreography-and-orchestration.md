# Choreography and Orchestration — Baştan Sona Öğretici

> Bu metin, **"Choreography and Orchestration"** dersinde anlatılan **her şeyi** kavratmak için yazıldı. Bu ders, "Service Orchestration and Choreography on Google Cloud" kursunun **3. modülüdür** ve doğrudan önceki iki modülün üzerine inşa edilir. Modül 09 ("Introduction to Microservices"), microservices mimarisinin beş zorluğundan birinin **iletişim karmaşıklığı** olduğunu göstermişti — servisler birbirini doğrudan çağırdıkça ortaya çıkan "örümcek ağı" sorunu. Modül 10 ("Event-Driven Applications"), bu soruna kavramsal bir çözüm sunmuştu: servislerin arasına bir **event intermediary** koymak, producer ve consumer'ı birbirinden **decouple** etmek. Ama modül 10, bu event intermediary fikrini hiçbir somut Google Cloud ürünüyle eşleştirmemişti — tamamen kavramsal kalmıştı. Bu modül, tam olarak bu boşluğu dolduruyor: event intermediary artık somut bir isim alıyor (**Pub/Sub**, **Eventarc**), ve microservices'i koordine etmenin iki temel yaklaşımı — **service choreography** ve **service orchestration** — beş gerçek Google Cloud ürünüyle (**Pub/Sub**, **Eventarc**, **Workflows**, **Cloud Tasks**, **Cloud Scheduler**) hayata geçiriliyor.
>
> **Kapsam notu:** Bu doküman, "Service Orchestration and Choreography on Google Cloud" kursunun **Modül 3 — Choreography and Orchestration** dersini kapsıyor. Bu, seride şimdiye kadarki **ilk modül** — Pub/Sub, Eventarc, Workflows, Cloud Tasks, Cloud Scheduler gibi **somut Google Cloud ürünlerini isimlendiren** ilk modül; modül 09 ve 10 tamamen kavramsaldı. Kursun ilerleyen modüllerinin transkriptleri eklendikçe, bu handbook'a `deep-dive/12-...`, `deep-dive/13-...` gibi ayrı numaralı modüller olarak eklenmeye devam edilecek.

---

## Bu ders tam olarak neyi öğretiyor ve neden önemli?

Modül 09'un kapanışını hatırlayalım: microservices mimarisi, monolith'in tight coupling'ini ve SOA'nın ESB darboğazını çözer, ama kendi yeni sorununu getirir. Ders'in kendi cümlesiyle o modülden bir kez daha alıntılayalım — bu dersin açılış cümlesi neredeyse **birebir aynı fikri** tekrar ediyor: **Her microservice, monolithic bir uygulamadan daha küçük ve daha basit olarak tasarlanabilir, ama bir miktar karmaşıklık servisten servisler-arası iletişime kayar (shifted).** Bu cümle, üç modül boyunca izlediğimiz "karmaşıklık yok olmaz, sadece yer değiştirir" temasının **tam burada, üçüncü kez** karşımıza çıkışıdır: monolith'te karmaşıklık kod tabanındaydı, SOA'da ESB'ye kaydı, microservices'te operasyona ve servisler-arası iletişime kaydı. Şimdi bu son kaymanın **nasıl yönetileceğini** öğreniyoruz.

Birden fazla microservice bir araya gelip bir **iş sürecini (business process)** oluşturduğunda, bu iletişimin **koordinasyonu**, uygulamanın tasarımının kilit bir parçası haline gelir. Ders, event-driven bir mimaride koordinasyona iki temel yaklaşım olduğunu söylüyor: **service choreography** ve **service orchestration**. Bu ders üç ana durağı sırayla geziyor:

1. **İki koordinasyon deseni** — service choreography (dans analojisi) ve service orchestration (orkestra/şef analojisi) — her birinin nasıl çalıştığı ve hangi ödünleşimleri (trade-off) getirdiği.
2. **Google Cloud'da choreography'i somutlaştıran araçlar** — Pub/Sub ve Eventarc; bunların modül 10'daki "event intermediary" kavramının gerçek ürünler olarak karşılığı olduğu.
3. **Google Cloud'da orchestration'ı somutlaştıran araç** — Workflows; ve iki deseni **ne zaman** seçeceğine dair pratik bir karar çerçevesi.
4. **Tamamlayıcı iki araç** — Cloud Tasks (belirli, bilinen bir servisi asenkron çalıştırmak için) ve Cloud Scheduler (zamanlanmış işler için).

Bu dördünü ayrı ayrı ürün tanıtımları olarak değil, **tek bir sorunun (servisler arası koordinasyon) farklı yüzleri** olarak düşün: bazen koordinasyonu dağıtık tutmak (choreography) daha iyidir, bazen merkezileştirmek (orchestration) daha iyidir; bazen bir servisi **belirli ve bilinerek** tetiklemek istersin (Cloud Tasks), bazen de **zamanla** tetiklemek istersin (Cloud Scheduler). Beşi birlikte, Google Cloud'un **Application Integration** araç kutusunu oluşturuyor.

---

# BÖLÜM 1 — İki Koordinasyon Deseni: Service Choreography ve Service Orchestration

## Neden bu ayrım gerekiyor?

Microservices mimarisinin faydalarından (ölçeklenebilirlik, yeniden kullanılabilirlik, değişim kolaylığı) yararlanırsın, ama her microservice bir öncekinden daha küçük ve tasarımı daha basit olsa da, **bir miktar karmaşıklık servisten servisler-arası iletişime kayar.** Birden fazla microservice sıkça bir araya gelip iş süreçlerini oluşturur, ve bu iletişimin koordinasyonu uygulama tasarımının **kilit bir yönü** haline gelir.

Event-driven bir mimaride koordinasyona iki temel yaklaşım vardır: **service choreography** ve **service orchestration**. Bu iki kelimenin İngilizcedeki gündelik anlamları, tam olarak aralarındaki mimari farkı taşıyor — bu yüzden ders her ikisini de bir performans sanatı analojisiyle açıklıyor.

## Service Choreography — dans koreografisi gibi

**Service choreography**, bir dans performansının koreografisine benzer. Bir dans koreografe edildiğinde, dansçılara dansı nasıl **icra edecekleri** öğretilir, ama performans sırasında **kendi bölümlerini icra etmekten tamamen kendileri sorumludur.** Sahnede onlara "şimdi şunu yap" diyen bir merkezi ses yoktur — her dansçı, önceden öğrendiği koreografiyi, diğer dansçıların hareketlerine bakarak **kendi başına** uygular.

Service choreography'de de aynı mantık işler: **her servis bağımsız olarak çalışır.** Her servis, event'leri **asenkron olarak** almaktan ve göndermekten sorumludur — genellikle servisler arasındaki **tanımlı etkileşim kurallarını (defined rules of interaction)** takip ederek. Servisler arasında değiş tokuş edilen **event yapıları (event structures) belirtilmiştir (specified)**, ve her servis bu event'leri **doğru formatlarda** üretmeli ve beklemelidir.

Choreography ile servisler **gevşek bağlıdır (loosely coupled)** ve ayrı ayrı oluşturulabilir, değiştirilebilir ve ölçeklendirilebilir.

Ama bir zorluk var: **iş mantığı (business logic) dağıtıktır (distributed).** Dağıtık bir uygulamayı anlamak daha zor olabilir, çünkü **merkezi bir kaynak-doğruluğu (central source of truth) yoktur.**

> **Bu, modül 10'daki hangi fikri somutlaştırıyor?** Modül 10'da, event intermediary'nin producer ve consumer'ı birbirinden decouple ettiğini, her servisin sadece kendi ilgilendiği event'in formatını bilmesi gerektiğini öğrenmiştin. Service choreography, tam olarak bu decoupling'in **mimari sonucudur**: hiçbir servis "büyük resmi" görmez, her biri sadece kendi rolünü bilir. Modül 10'daki "merkezi auditing/access control" faydasını hatırla — o fayda hâlâ geçerlidir (event intermediary seviyesinde kimlik doğrulama yapılır), ama bu, **iş sürecinin mantığının** merkezi olduğu anlamına gelmez. İş süreci mantığı, choreography'de servislerin **her birine dağıtılmış** durumdadır.

> **Analoji:** Bir orkestrasyon olmadan çalışan bir restoran mutfağını düşün — ama modül 09'daki "tek büyük salon" monolith analojisinin tam tersi bir versiyonu. Her aşçı (servis), kendi istasyonunda bağımsız çalışır ve "önümdeki tabak hazır olduğunda bir sonraki aşçıya haber ver" kuralını bilir. Hiçbir şef (merkezi orkestratör) koşuşturup "şimdi sen başla, şimdi sen bitir" demez. Bu esnek ve hızlıdır — ama bir tabak mutfaktan çıkmadan önce hangi aşamalardan geçtiğini **tek bir yerden** izlemek istersen, bunu yapamazsın; her aşçıya ayrı ayrı sormak zorunda kalırsın.

## Service Orchestration — orkestra performansı gibi

**Service orchestration** ise bir orkestra performansına benzer. Orkestradaki her müzisyen kendi enstrümanını nasıl çalacağını bilir, ama **şef (conductor)**, performans sırasında **aktif bir rol** üstlenerek müzisyenlerin senkronize olmasını sağlar.

Service orchestration'da, her servis kendi işlerini (tasks) yerine getirir, ama **merkezi bir orkestratör (central orchestrator)**, servisler arasındaki **tüm etkileşimleri kontrol eder.** Servislerin, orkestrasyondaki diğer servisleri bilmelerine ya da onlarla doğrudan iletişim kurmalarına **gerek yoktur.**

Choreography'de olduğu gibi, orchestration'da da servisler gevşek bağlıdır ve ayrı ayrı oluşturulabilir, değiştirilebilir, ölçeklendirilebilir. Ama orchestration'ın ekstra bir faydası var: **iş süreçlerinin yüksek seviyeli bir görünümünü (high-level view) sağlar** — bu da uygulamayı anlamaya, execution'ı izlemeye (tracking) ve sorunları gidermeye (troubleshooting) yardımcı olur.

Ama choreography'deki tamamen dağıtık servislerin aksine, service orchestration'ın **tek bir arıza noktası (single point of failure)** vardır: **orkestratör çalışmıyorsa, orkestre edilen süreçler de çalışamaz.**

> **Neden şef metaforu bu kadar isabetli?** Bir orkestrada, her müzisyen enstrümanını mükemmel çalabilir — ama şef olmadan, kiminle senkronize olacaklarını, ne zaman başlayıp ne zaman duracaklarını bilemezler; kakofoni çıkar. Şef, müziği **kendisi çalmaz**; sadece **kimin ne zaman çalacağını** koordine eder. Service orchestration'da orkestratör da böyledir: iş mantığının kendisini (örneğin ürün resmini yeniden boyutlamayı) **yapmaz**, sadece "şimdi Servis A'yı çağır, sonucuna göre Servis B ya da Servis C'yi çağır" akışını yönetir. Ama tıpkı bir orkestrada şef sahneden ayrılırsa performansın dağılması gibi, orkestratör çökerse **tüm süreç durur** — bu, choreography'de olmayan bir risktir, çünkü choreography'de "merkez" diye bir şey yoktur, dolayısıyla çökecek bir merkez de yoktur.

> **Sınav tuzağı:** "Service orchestration her zaman choreography'den üstündür, çünkü merkezi görünürlük ve troubleshooting kolaylığı sağlar" düşüncesi **eksik bir çıkarımdır.** Ders, orchestration'ın somut faydalarını (yüksek seviyeli görünüm, troubleshooting kolaylığı) açıkça sayarken, **aynı paragrafta** onun **single point of failure** olduğunu da vurguluyor — bu, choreography'nin tamamen dağıtık doğasında **bulunmayan** bir risktir. Sınav sorusu "orchestration'ın choreography'ye göre dezavantajı nedir?" diye sorarsa, doğru cevap **"orkestratör çökerse tüm orkestre edilen süreç durur (single point of failure)"** olmalı — "hiçbir dezavantajı yoktur" değil.

**Service Choreography vs. Service Orchestration — özet tablo:**

| Özellik | Service Choreography | Service Orchestration |
| --- | --- | --- |
| Analoji | Dans koreografisi — her dansçı bağımsız icra eder | Orkestra performansı — şef tüm etkileşimleri yönetir |
| Kontrol | Merkezi değil; her servis kendi rolünü bilir | Merkezi orkestratör tüm etkileşimleri kontrol eder |
| Servisler birbirini tanır mı? | Hayır, sadece event formatını bilirler | Hayır, orkestratörle konuşurlar, birbirleriyle değil |
| Coupling | Gevşek (loosely coupled) | Gevşek (loosely coupled) |
| İş mantığı (business logic) | Dağıtık — servislere yayılmış | Merkezi — orkestratörde tanımlı |
| Kaynak-doğruluğu (source of truth) | Yok — merkezi bir kaynak yok | Var — orkestratör tek kaynak-doğruluğudur |
| Görünürlük / troubleshooting | Zor — dağıtık mantığı anlamak zordur | Kolay — yüksek seviyeli, uçtan uca görünüm var |
| Arıza noktası (failure point) | Yok — dağıtık, tek bir merkez çökmez | Var — orkestratör çökerse süreç durur |
| Google Cloud karşılığı (bu derste) | Pub/Sub, Eventarc | Workflows |

---

# BÖLÜM 2 — Pub/Sub: Choreography'i Google Cloud'da Gerçekleştirmek

## Pub/Sub nedir ve neden var?

Choreography ve orchestration'ın temellerini öğrendiğine göre, şimdi servisleri Google Cloud'da nasıl **choreograph** edebileceğine bakalım. **Pub/Sub**, bu iş için kullanabileceğin Google Cloud servislerinden biridir.

**Pub/Sub, bağımsız servisler ya da uygulamalar arasında mesaj göndermeni ve almanı sağlayan, tamamen yönetilen (fully managed), gerçek zamanlı (real-time) bir mesajlaşma servisidir.**

Mekanizma şöyle işler:

1. Bir **publisher**, bir Pub/Sub **topic**'ine bir mesaj gönderir. Bu publisher genellikle özel olarak inşa edilmiş (custom-built) bir uygulamadır, ama bir Google servisi de olabilir.
2. Mesaj **Pub/Sub içinde saklanır**, ardından **topic'in her bir subscriber'ının mesaj kuyruğuna (message queue)** teslim edilir.
3. Her Pub/Sub **subscriber**, mesajı alır. **Pull subscription** kullanan bir subscriber, topic'e gönderilmiş mesajları görmek için **ara sıra (occasionally) poll eder.** **Push subscription** kullanan bir subscriber ise, mesajın subscription için yapılandırılmış bir uç noktaya (endpoint) **otomatik olarak gönderilmesine** neden olur.
4. Subscriber, mesajı ardından **acknowledge eder (onaylar).** Bir mesajı acknowledge etmek, onu **kuyruktan kaldırır.** Kuyruktaki mesajın silinmesi, mesajın **işlenmesinin ardından** gerçekleşir; bu da bir mesajın **en az bir kez (at least once)** işleneceğini garanti eder.

> **Analoji:** Pub/Sub'ı bir **dergi aboneliği (magazine subscription)** sistemine benzet. Bir yayıncı (publisher), bir dergi sayısını (mesajı) **bir kere** yazar ve basar; yayıncının, o sayıya kaç kişinin abone olduğuyla, hatta hiç abonesi olup olmadığıyla ilgilenmesi gerekmez (tam olarak modül 10'daki "producer, event'i consume edilmese bile üretir" özelliğini hatırla). Her bir abone (subscriber), **kendi kopyasını** alır — abone sayısı ister bir, ister bin olsun, yayıncının işi **değişmez.** Bir abone dergiyi hemen okuyabilir (push — kapına bırakılır) ya da postane kutusunu kendi zamanında kontrol edebilir (pull — kendisi gidip alır). Dergiyi okuyup "okudum" diye işaretlemek (acknowledge), o sayının **senin okuma listenden** düşmesi demektir — ama bu, derginin **basılı hiç olmadığı** ya da **başka hiçbir abonenin** onu okumadığı anlamına gelmez.

## Pull vs. push subscription — ne zaman hangisi?

İki subscription türü, iki farklı **kontrol modelini** temsil eder:

- **Pull subscription**: Subscriber, **kendi zamanlamasıyla** mesajları sorgular (poll eder). Subscriber, ne zaman ve ne sıklıkla kontrol edeceğine kendisi karar verir.
- **Push subscription**: Pub/Sub, mesajı **otomatik olarak**, subscription için yapılandırılmış bir uç noktaya **gönderir.** Subscriber, mesajın gelmesini bekler — kendisi sormaz.

> **Bu ayrım modül 10'daki hangi kavramla örtüşüyor?** Modül 10'un BÖLÜM 6'sında **push-based messaging vs. polling** karşılaştırması vardı: polling, sürekli "iş var mı?" diye sormak demekti ve artan network I/O ile gereksiz gecikmelere yol açıyordu; push ise event oluştuğunda otomatik bilgilendirme demekti. Pub/Sub'ın pull ve push subscription'ları, tam olarak bu iki modelin **somut Google Cloud karşılığıdır.** Bir subscriber'ın hangi modeli seçeceği, o servisin ihtiyaçlarına bağlıdır — sürekli çalışan, yüksek hacimli bir işleyici pull tercih edebilir (kendi hızında çeker); hafif, olay-tetiklemeli bir servis push tercih edebilir (gelen mesajı anında işler).

## Acknowledge etmek ve "at-least-once delivery"

Bir mesajı acknowledge etmek, onu kuyruktan siler. Bu silme işlemi **mesaj işlendikten sonra** olur — bu da Pub/Sub'ın **en az bir kez (at-least-once) teslim garantisinin** kaynağıdır.

> **Sınav tuzağı — "at-least-once" ile "exactly-once" karıştırmak:** Pub/Sub'ın garantisi **"en az bir kez"**dir, **"tam olarak bir kez" (exactly-once) değildir.** Bunun pratik anlamı şudur: eğer bir subscriber bir mesajı işler ama acknowledge etmeden **önce** çöker ya da zaman aşımına uğrarsa, Pub/Sub o mesajın **hâlâ işlenmediğini varsayar** ve onu **tekrar teslim edebilir.** Bu, subscriber tarafındaki işleme mantığının **idempotent (aynı mesajı birden fazla kez işlese bile sonucu bozmayan)** olacak şekilde tasarlanması gerektiği anlamına gelir. Sınav sorusu "Pub/Sub bir mesajın sadece bir kez işleneceğini garanti eder mi?" diye sorarsa cevap **hayır** — garanti **en az bir kez**dir, tekrarlar mümkündür ve uygulamanın buna dayanıklı olması beklenir.

## Örnek: Görsel yeniden boyutlandırma (image resizing) uygulaması

Ders, Pub/Sub'ın servisleri nasıl bağladığını somutlaştırmak için bir görsel yeniden boyutlandırma uygulaması örneği veriyor. Akış şöyle işliyor:

1. Görsel yüklemeleri (image uploads) için bir Pub/Sub **topic**'i oluşturulur.
2. Belirli bir Cloud Storage bucket'ı, yeni bir görsel alındığında o topic'e bir Pub/Sub mesajı göndermek üzere **yapılandırılır.**
3. Mobil cihaz uygulaması, "Image Uploads" bucket'ına yeni bir görsel yüklediğinde, Pub/Sub'a **otomatik olarak** bir mesaj gönderilir.
4. Bu mesaj, **iki subscriber'a** iletilir: **Resizing Service** ve **Upload Confirm** uygulaması.
5. **Upload Confirm** uygulaması, görsel yüklemesinin başarıyla tamamlandığı bilgisini **Firestore**'da saklayarak Firestore'u günceller.
6. **Resizing Service**, görseli yeniden boyutlandırır, "Resized Images" bucket'ına saklar, ve durumu **Firestore**'da günceller.
7. Uygulama, durum güncellemelerini Firestore üzerinden **Firebase Realtime Database** güncellemeleriyle izleyebilir.
8. Her Cloud Run servisi **oldukça basit** kalabilir; Pub/Sub, işlenecek bir görsel olduğunda işlemi **başlatır (initiate).**

> **Bu örnek, choreography'nin ne anlama geldiğini nasıl gösteriyor?** Cloud Storage bucket'ı, kimin dinlediğini bilmeden bir mesaj yayınlar. Resizing Service ve Upload Confirm uygulaması, birbirlerinden **habersiz**, aynı mesajı **paralel olarak** tüketirler — her biri kendi görevini yapar (biri yeniden boyutlandırır, diğeri onay kaydeder) ve hiçbiri diğerinin varlığından **haberdar olmak zorunda değildir.** Bu, tam olarak BÖLÜM 1'de tanımlanan "her servis bağımsız çalışır, önceden tanımlı etkileşim kurallarını takip ederek event alır/gönderir" tanımının somut halidir. Yarın bu topic'e üçüncü bir subscriber (örneğin bir analytics servisi) eklemek istersen, ne Cloud Storage bucket'ının ne de mevcut iki servisin **tek bir satırını bile değiştirmen gerekmez** — modül 10'un BÖLÜM 4'ünde gördüğün "yeni consumer'lar mevcut servisleri etkilemeden eklenebilir" faydasının **tam burada** gerçekleştiğini görüyorsun.

**Pull vs. push subscription — özet tablo:**

| Özellik | Pull subscription | Push subscription |
| --- | --- | --- |
| Kim harekete geçer? | Subscriber, kendi zamanlamasıyla poll eder | Pub/Sub, mesajı otomatik olarak subscriber'ın endpoint'ine gönderir |
| Modül 10 karşılığı | Polling | Push-based messaging |
| Uygun senaryo | Subscriber kendi hızında işlemek istiyor | Subscriber olay anında anında tepki vermek istiyor |

---

# BÖLÜM 3 — Eventarc: Event-Driven Mimarinin Somutlaşması ve CloudEvents

## Eventarc nedir ve neden var?

**Eventarc**, event-driven uygulamalar inşa etmeyi kolaylaştıran, Google Cloud'un **tamamen yönetilen (fully managed) eventing sistemidir.**

Eventarc, event kaynağı (event source) olarak **birçok Google Cloud ürününü** destekler:

- Bazı Google Cloud servisleri için, kaynak **doğrudan (directly)** Eventarc'a event gönderebilir.
- Doğrudan erişimi desteklemeyen Google servisleri ya da event türleri için, Eventarc **sorunsuzca (seamlessly) Cloud Audit Logs** girdilerini kullanarak event üretebilir.
- **Üçüncü taraf sağlayıcılar (third-party providers)** da event oluşturmak için Eventarc API'sini kullanabilir.
- Uygulamalar, audit log'ları parse etmek ya da event'leri poll etmek için **kod yazmak zorunda değildir.**
- Belirtilen topic'lere gönderilen **Pub/Sub mesajları** da event kaynağı olarak kullanılabilir — bu entegrasyon, özel (custom) uygulamaların da event göndermesine izin verir.

## Event trigger

Bir **event trigger**, belirli event türlerini belirli bir event kaynağından belirli event tüketicilerine (event consumers), yani **hedeflere (targets)**, yönlendiren, **kural tabanlı (rule-based) bir filtredir.**

## Eventarc'ın motoru: Pub/Sub, ama gizli

Burada dersin en kritik teknik detayına geliyoruz: **Eventarc, üstün güvenilirliği (reliability) ve gözlemlenebilirliği (observability) nedeniyle taşıma katmanı (transport layer) olarak Pub/Sub'ı kullanır.** Eventarc, event teslimatını kolaylaştırmak için Pub/Sub topic'lerini ve subscription'larını **otomatik olarak oluşturur ve yönetir.**

Bunun pratik sonucu şu: **senin uygulamanın sadece Eventarc tarafından otomatik olarak gönderilen HTTP isteklerini kabul etmesi yeterlidir.** Uygulamaların Pub/Sub'ı **doğrudan kullanmasına gerek yoktur.**

> **Analoji:** Eventarc'ı, **her kaynağın dilini zaten konuşan, evrensel bir olay santrali (universal event dispatcher / switchboard)** gibi düşün. Farklı bir Google Cloud servisi (Cloud Storage, BigQuery, Firestore...), farklı bir audit log biçimi, farklı bir üçüncü taraf sağlayıcı — hepsi kendi "diliyle" event üretir. Eventarc, bu farklı dillerin **hepsini zaten anlar** ve arkada, kendi kurduğu ve yönettiği Pub/Sub topic'leri/subscription'ları üzerinden, sana **tek, standart bir dilde (HTTP + CloudEvents formatı)** konuşan bir mesaj olarak iletir. Sen santralin arkasındaki telefon hatlarıyla (Pub/Sub'ın kendisiyle) hiç uğraşmazsın — sadece santralin sana ilettiği çağrıyı (HTTP isteğini), tek bir standart formatta karşılarsın.

> **Sınav tuzağı — Eventarc ve Pub/Sub'ı rakip sanmak:** "Eventarc mi kullanmalıyım, Pub/Sub mı?" sorusu **yanlış bir ikilemdir.** Eventarc, Pub/Sub'ın **yerine geçen** bir ürün değil, **onun üzerine inşa edilmiş bir soyutlama katmanıdır (abstraction layer).** Eventarc, arka planda hâlâ Pub/Sub'ı taşıma katmanı olarak kullanır — sadece topic/subscription yönetimini senden **alıp** kendisi üstlenir. Doğru soru "ne zaman Eventarc'ı, ne zaman Pub/Sub'ı doğrudan kullanmalıyım?" şeklinde olmalı; bunu bu bölümün sonunda cevaplayacağız.

## CloudEvents: standart bir format

Event'ler, kaynağı ne olursa olsun, hedeflere **standart CNCF CloudEvents formatı** kullanılarak teslim edilir. Bir trigger oluşturduğunda, o spesifik event için kullanılacak **format ve alanları (fields)** görebilirsin.

Eventarc'ın kullandığı CloudEvents formatı, event verisini tanımlamak için **ortak bir metadata formatı** belirtir; bu da servisler, platformlar ve sistemler arasında **birlikte çalışabilirlik (interoperability)** sağlar.

Neden bu önemli? Event yayıncıları (publishers), **tarihsel olarak** kendi event'leri için **kendi formatlarını** kullanma eğilimindeydi. CloudEvents, belirli bir formatı standartlaştırarak, geliştiricilerin **event'in kaynağı ya da türü ne olursa olsun aynı event işleme mantığını (event handling logic)** kodlarında kullanabilmesini sağlar.

CloudEvents, günümüzde yaygın kullanılan birçok programlama dili için **SDK'lar** sunar: **Python, JavaScript, Java, Go, C#, Ruby ve PHP.** Geliştiriciler bu SDK'ları kullanarak gelen event'leri kolayca **parse edebilir** — bu da kod taşınabilirliğini (portability) ve geliştirici verimliliğini (productivity) artırır.

> **Modül 10'un neresine bağlanıyor?** Modül 10'da, event intermediary'nin producer ve consumer'ı, sadece **event'in formatını bilmeleri gerektiği** ölçüde birbirinden bağımsızlaştırdığını öğrenmiştin. CloudEvents, tam olarak bu "format" fikrini **standart hale getirir**: artık her kaynağın kendi özel formatını öğrenmene gerek yok, tek bir CloudEvents şemasını öğrenmen, tüm kaynaklardan gelen event'leri işlemek için yeterli.

## Eventarc bir soyutlama katmanıdır — ne zaman kullanmalı?

Eventarc, event-driven bir uygulama tasarlarken **önemli faydalar** sağlayan, Pub/Sub'ın üzerine kurulu bir soyutlama katmanıdır:

- Eventarc, event kaynağı olarak **birçok yerleşik (built-in) servisi** kullanabilir.
- Eventarc, birçok Google Cloud servisi ve üçüncü taraf uygulamasındaki değişiklikleri **kolayca tespit etmeni** ve bu değişikliklere yanıt olarak **otomatik olarak kod çalıştırmanı** sağlar.
- Eventarc, bir trigger'ın kaynağını (source), filtresini (filter) ve hedefini (destination) seçmek için **basit, kural tabanlı bir arayüz** sunar.
- Pub/Sub ile, event'leri **ingest etmek için kod yazman**, topic'leri ve subscription'ları **kendin yönetmen** gerekir.

Son olarak: **standart bir event formatı istiyorsan Eventarc'ı kullanmalısın.** Standart CloudEvents formatını kullanarak, event oluşturmak ve tüketmek için CloudEvents SDK'larını kullanabilirsin.

**Pub/Sub vs. Eventarc — özet tablo:**

| Özellik | Pub/Sub (doğrudan kullanım) | Eventarc |
| --- | --- | --- |
| Rolü | Temel mesajlaşma altyapısı, taşıma katmanı | Pub/Sub üzerine kurulu soyutlama katmanı |
| Event kaynakları | Sadece senin publish ettiğin mesajlar | Birçok GCP servisi (doğrudan ya da Audit Logs üzerinden), üçüncü taraf sağlayıcılar, Pub/Sub topic'leri |
| Topic/subscription yönetimi | Sen yönetirsin | Otomatik oluşturulur ve yönetilir |
| Uygulamanın yapması gereken | Pub/Sub client kütüphanesiyle mesaj ingest etmek | Sadece gelen HTTP isteklerini kabul etmek |
| Event formatı | Kendi tanımladığın (standart yok) | Standart CNCF CloudEvents formatı |
| Kaynak seçimi/filtreleme | Kendi kodunla yapman gerekir | Kural tabanlı (rule-based) trigger arayüzü |
| Ne zaman tercih edilir | Sadece kendi custom publish/subscribe akışın varsa, tam kontrol istiyorsan | Birçok kaynaktan event almak, standart format ve basit kurulum istiyorsan |

---

# BÖLÜM 4 — Workflows: Service Orchestration'ı Google Cloud'da Gerçekleştirmek

## Workflows nedir ve neden var?

Google Cloud'un **Workflows** platformu, **service orchestration deseninin** hayata geçirilmesinde kullanılabilir. **Workflows, tamamen yönetilen (fully managed) bir orkestrasyon platformudur** ve service orchestration deseni için **merkezi orkestratör** rolünü üstlenir.

Workflow'ları **tasarlar ve deploy edersin**; bu workflow'lar, **stateful (durumlu), otomatik süreçler** inşa etmek için **Google Cloud servislerini ve API çağrılarını orkestre eder.**

Bir workflow, uygulama akışı (application flow) için **merkezi bir kaynak-doğruluğu (central source of truth)** sağlar. Her workflow execution'ı **loglanır ve gözlemlenebilir (observable)** — bu da workflow'un mevcut durumunu anlamayı ve herhangi bir sorunu gidermeyi (troubleshoot) kolaylaştırır.

Bir workflow; **durum (state) tutabilir, yeniden deneyebilir (retry), poll edebilir ya da bir yıla kadar bekleyebilir.** Bu esneklik, **uzun süren (long-running) iş süreçlerinin** oluşturulmasına olanak tanır.

> **Bu, BÖLÜM 1'deki hangi vaadin somut karşılığıdır?** Service orchestration'ın vaat ettiği "merkezi kaynak-doğruluğu" ve "yüksek seviyeli görünüm" soyut kavramlarını hatırla. Workflows, bunu **her execution'ın loglanıp gözlemlenebilir olması** ile somutlaştırıyor: soyut olarak "orkestratör süreci takip eder" demek yerine, Workflows'ta gerçekten **her bir çalıştırmayı** tek tek, ayrı ayrı görebilir ve inceleyebilirsin.

## Örnek: Ürün siparişi (product ordering) workflow'u

Ders, Workflows'un nasıl çalıştığını somutlaştırmak için bir ürün siparişi workflow'u örneği veriyor:

1. Workflow sürecinin ilk adımı, sipariş edilen ürünlerin **stok durumunu** kontrol etmek için **Firestore inventory database**'ini kontrol etmektir.
2. Eğer ürünler mevcutsa (available), **kilitlenirler (locked)**.
3. Workflow, siparişteki herhangi bir ürünün stokta olup olmadığına (out of stock) bağlı olarak **iki yoldan birini izler.**
4. "Out of stock" boolean koşulu, workflow boyunca **erişilebilir durumdadır** ve sürecin **sonunda da** tekrar kullanılır.
5. Siparişteki her ürün mevcutsa, workflow sipariş onay mesajını hazırlamak için bir **Cloud Run function** çağırır.
6. Bir ya da daha fazla ürün stokta yoksa, ilgili tedarikçilerden (suppliers) daha fazla envanter talep etmek için bir **Cloud Run service** tetiklenir.
7. Cloud Run service çağrıldıktan sonra, müşteri için bir **"üzgünüz" (we're sorry) mesajı** hazırlanır.
8. Ardından, envanter ve sipariş detayları **Firestore veritabanlarında** güncellenir.
9. Müşteriye, siparişin başarılı olup olmadığını belirten bir **e-posta** gönderilir.
10. Son olarak, bir ya da daha fazla ürün stokta değilse, satış temsilcisini (sales rep) bilgilendirmek için bir **Slack mesajı** gönderilir.

Tek bir işlemi (transaction) işleyen her workflow execution'ı, troubleshooting ya da tracing için **loglanır.** Workflow, API'ler tarafından fırlatılan **retry'ları ya da exception'ları yönetir** — bu da tüm sürecin güvenilirliğini artırır.

> **Bu örnek, "central source of truth" ve "single point of failure" fikirlerini nasıl gösteriyor?** "Out of stock" boolean'ının workflow boyunca erişilebilir olması ve sürecin sonunda **tekrar** kullanılması dikkat çekici: bu, bir choreography senaryosunda her servisin **kendi başına** yeniden hesaplaması ya da bir yerlerde saklaması gereken bir bilginin, orchestration'da **tek bir yerde, workflow'un state'inde** yaşadığını gösteriyor. Aynı zamanda, eğer bu workflow'u çalıştıran Workflows servisi bir şekilde erişilemez hale gelirse, **hiçbir sipariş bu süreçten geçemez** — BÖLÜM 1'de tanımlanan single point of failure riski burada somutlaşıyor.

## Workflows ne zaman doğru seçimdir?

Workflows, **HTTP tabanlı microservices'i dayanıklı (durable) ve stateful workflow'lara zincirlemek (chain)** istediğinde **mükemmel bir seçimdir.** Workflows, uzun süren süreçleri uygulamana ve her execution için gözlemlenebilirliği korumana olanak tanır.

Workflows aynı zamanda, bir **öğe kümesi (set of items) ya da batch data üzerinde işlem yapmak** için de harika bir seçimdir. **Sağlam hata yönetimi (robust error handling)**, her öğenin doğru işlendiğini garanti edebilir.

---

# BÖLÜM 5 — Choreography mi, Orchestration mı? Pratikte Seçim

## "Bağlı" (it depends) — ama daha iyi bir soru var

Choreography ve orchestration'ın Google Cloud'da nasıl çalıştığını öğrendiğine göre, asıl soruya gelelim: **hangi desen daha iyi, choreography mi orchestration mı?**

Birçok uygulama mimarisi kararında olduğu gibi, cevap **"bağlı" (it depends)**. Daha iyi soru şudur: **"Ne zaman choreography kullanmalıyım, ne zaman orchestration?"**

## Choreography'de kontrol kimde?

Event-tabanlı choreography kullandığında, iletişimi kontrol etmek için **alıcı servise (receiving service)** güvenirsin. Servis A bir işi tamamlayıp bir event gönderdiğinde, **Servis A, başka bir servisin o event üzerinde işlem yapıp yapmayacağını bilmez.** Herhangi bir downstream servisin o event'i tüketip tüketmediğini bilmek, **Servis A'nın sorumluluğunda değildir.**

Eventarc kullanarak, çoğu Google Cloud ya da özel (custom) servis bir **event producer** rolü üstlenebilir. Ama bir event'in **alıcısı (receiver)**, event'i tüketebilmek için **event'in detaylarını anlamak zorundadır.** Servis B, Servis A tarafından gönderilen event'i tükettiğinde, event'in formatlamasını (formatting) ve event üzerine nasıl işlem yapılacağını **anlar.** Servis B, event'in Servis A tarafından gönderildiğini de **bilebilir** — ama Servis A'ya **doğrudan bağlı (coupled)** değildir.

## Ürün siparişi süreci choreography ile nasıl olurdu?

Ürün siparişi akışını **choreography kullanarak** implemente etmek mümkün. Her servis, süreçteki bir sonraki adıma **hazır olduğunu belirten (indicate readiness)** bir event gönderirdi. "Out of stock" karar noktaları, **belirli servisler tarafından**, "in stock" ya da "out of stock" event'lerini gönderip göndermeyeceğine karar vererek implemente edilebilirdi.

Ama bir kurumsal (enterprise) sipariş sistemi **görünürlük (visibility), hata yönetimi (error handling) ve retry'lar** gerektirir. Bu özellikleri event-tabanlı bir çözümde implemente etmek **zor olabilir.** Ders şu soruları soruyor:

- **Event-tabanlı bir sistemde sorunları nasıl giderirsin (troubleshoot)?**
- **Süreç, envanteri veritabanında kilitlemekle envanteri ve siparişi güncellemek arasında bir yerde durursa (abort) ne olur?**
- **Tedarikçilere gönderilen isteklerin başarıyla iletildiğinden nasıl emin olursun?**

> **Bu üç soru neden bu kadar keskin?** Çünkü bunlar, BÖLÜM 1'de choreography'nin zayıf noktası olarak tanımladığımız "dağıtık iş mantığı ve merkezi kaynak-doğruluğunun olmaması" fikrinin **somut, gündelik sonuçlarıdır.** Choreography'de her servis kendi rolünü bilir ama **kimse sürecin bütününü görmez.** Bir sipariş "kayboluyorsa" (envanter kilitlendi ama hiçbir zaman güncellenmedi), bunu fark etmek için **her servisin loglarını ayrı ayrı** incelemen, aralarındaki event zincirini **elle yeniden kurman** gerekir — tek bir yerde "bu siparişin şu anki durumu budur" diyen bir kayıt **yoktur.**

## Orchestration ile aynı süreç

Orchestration ile, **her workflow execution'ı ayrı ayrı izlenir (tracked).** Sipariş süreci mantığı — birçok servis ya da fonksiyon kullanılarak inşa edilmiş olsa bile — **tek bir yerde tanımlıdır.** Retry'lar ve hata yönetimi, **güvenilir bir sipariş işleme akışı** sağlayabilir.

## Ama orchestration her zaman mümkün değil: Merkezi kontrol gerektirir

Bazen mimarin, **tek bir orkestre edilmiş workflow tarafından sağlanamayacak** merkezi olmayan (decentralized) kontrol gerektirir. Bu servisler, **farklı takımlar ya da organizasyonlar tarafından** inşa edilip yönetiliyorsa, merkezi orkestrasyon sürecinin **paylaşılan yönetimi (shared management)** zor olabilir.

Choreography, merkezi olmayan kontrol sağlar: her servis ya da uygulama, diğerlerine **event'ler aracılığıyla** bağlanır. Her servis, **farklı bir takım ya da organizasyon tarafından ayrı ayrı yönetilebilir.**

> **Bu, modül 09'daki hangi ayrımı yansıtıyor?** Modül 09'da SOA'nın "kurumsal düzeyde, merkezi standartlar ve merkezi ESB takımı" gerektirdiğini, microservices'in ise "ayrıştırılmış (decentralized)" bir yaklaşım sunduğunu öğrenmiştin. Orchestration ile choreography arasındaki bu seçim, **aynı gerilimin** bir başka biçimidir: orchestration, merkezi bir "orkestratör takımının" tüm süreç mantığını sahiplenmesini varsayar — bu da SOA'nın merkezi ESB takımına benzer bir yapı gerektirir. Choreography ise, her takımın **kendi servisini** kendi başına yönetip, sadece event formatları üzerinden anlaşmasını sağlar — tıpkı microservices'in kendi API'siyle bağımsız çalışması gibi.

## Kural: Ne zaman hangisi?

**Orchestration**, **merkezi olarak yönetebileceğin karmaşık bir süreci kontrol etmek istediğinde** güçlü bir desendir. **Choreography** ise, **ayrı, merkezi olmayan servisleri ve uygulamaları event'ler kullanarak birleştirdiğinde**, ya da **Google Cloud servislerinden gelen event'lerden doğrudan yararlanmak istediğinde** daha uygundur.

Karmaşık servislerini oluşturmak için gereken görünürlüğü, hata yönetimini ve güvenilirliği, Workflows ve orchestration kullanmanın sağladığını görebilirsin. Servisler arasında event göndermek için Eventarc'ı kullanabilirsin.

## İkisi bir arada: Gerçek dünyada hibrit mimari

Pratikte, choreography ve orchestration **birbirini dışlamaz — birleşirler.** Ders'in verdiği örnekte: **Order, Fulfillment ve Marketing servisleri Workflows kullanılarak implemente edilir.** Servisler arasındaki event trigger'ları ve bir Cloud Storage bucket'a yeni sipariş dosyalarının yüklenmesinin tespiti (detection), **Eventarc kullanılarak implemente edilir.**

> **Bu mimariyi nasıl okumalısın?** Her bir servis (Order, Fulfillment, Marketing), **kendi içinde** bir orchestration'dır — her biri kendi Workflows tanımına sahiptir, kendi iç adımlarını merkezi olarak yönetir (BÖLÜM 4'teki ürün siparişi örneği gibi). Ama bu üç servisin **birbirleriyle** nasıl konuştuğu — Order servisinin Fulfillment'ı nasıl tetiklediği, yeni bir dosyanın Cloud Storage'a yüklenmesinin hangi servisleri harekete geçireceği — **choreography** ile, Eventarc üzerinden event'lerle yönetilir. Yani mimari, **iç içe iki katmandır**: her servisin **içinde** orchestration (merkezi, öngörülebilir, izlenebilir), servisler **arasında** choreography (gevşek bağlı, merkezi olmayan, esnek). Bu, "ya orchestration ya choreography" sorusunun aslında **yanlış bir ikilem** olduğunu gösteren en net örnektir.

**Choreography vs. Orchestration — pratik seçim tablosu:**

| Durum | Önerilen desen | Neden |
| --- | --- | --- |
| Karmaşık bir süreci merkezi olarak yönetebiliyorsun (tek takım/org) | Orchestration (Workflows) | Tek bir yerde tanımlı mantık, güvenilir retry/hata yönetimi, görünürlük |
| Servisler farklı takımlar/organizasyonlar tarafından ayrı ayrı yönetiliyor | Choreography (Pub/Sub, Eventarc) | Merkezi bir orkestrasyon süreci paylaşılan yönetim gerektirir, bu zor olabilir |
| Doğrudan Google Cloud servis event'lerinden yararlanmak istiyorsun | Choreography (Eventarc) | Eventarc, birçok GCP servisini doğrudan event kaynağı olarak destekler |
| Görünürlük, hata yönetimi, retry kritik (örn. kurumsal sipariş sistemi) | Orchestration (Workflows) | Merkezi kaynak-doğruluğu, her execution ayrı izlenir |
| Büyük, çok-servisli bir sistem | İkisi birden (hibrit) | Her servis içinde orchestration, servisler arasında choreography |

---

# BÖLÜM 6 — Cloud Tasks: Explicit Invocation ile Asenkron Görev Yönetimi

## Cloud Tasks nedir ve neden var?

**Cloud Tasks**, büyük sayıda dağıtık görevin (distributed tasks) **execution, dispatch ve delivery**'sini yönetmeni sağlar. Bir **task**, bir **HTTP servisine dispatch edilerek** bağımsız olarak işlenebilen bir iş parçasıdır.

Belirli bir servise gönderilecek task'lar için bir **queue (kuyruk)** kurabilirsin. Bu kuyruğa eklenen bir task, **otomatik olarak seçtiğin HTTP servisine dispatch edilir.** Dönen durum kodu (status code), task'ın başarıyla tamamlanıp tamamlanmadığını ya da **yeniden denenmesi (retried) gerekip gerekmediğini** gösterir.

Bir kuyruğa task gönderirken, task'ın **gelecekteki belirli bir zamanda** dispatch edilmesini zamanlayabilirsin (schedule). Kuyruğu, task'ları dispatch etmek için **maksimum bir oran (maximum rate)** ya da **eş zamanlı (concurrently) dispatch edilebilecek maksimum task sayısı** ile yapılandırabilirsin. Task'lar yeniden denendiğinde, **maksimum deneme sayısını (maximum attempts)** ve denemeler arasındaki **gecikmeyi (delay)** de belirtebilirsin.

Cloud Tasks ayrıca **en az bir kez (at-least-once) teslimatı garanti eder** ve oluşturulan **tekrar eden (duplicate) task'ları eler.** Kimlik doğrulama (authentication) gerektiren HTTP servisleri, task'ı oluşturan uygulamanın **service account'una bağlı bir token'ın otomatik olarak eklenmesiyle** çağrılabilir.

## Cloud Tasks vs. Pub/Sub: Kavramsal olarak benzer, kullanım amacı farklı

Cloud Tasks ve Pub/Sub **kavramsal olarak benzerdir** — ikisi de mesaj geçişi (message passing) ve asenkron entegrasyon implemente eder — ama **farklı kullanım durumları için tasarlanmışlardır.**

**Cloud Tasks ile**, task'ı oluşturan taraf **explicit invocation (açık çağırma)** kullanır: **oluşturucu (creator), task'ın execution'ı ve hedefi üzerinde tam kontrolü elinde tutar.** Oluşturucu, task'ı **belirli bir uç noktaya (endpoint) bağlı bir kuyruğa** yerleştirir, ve task'ın dispatch edilmesini **erteleyebilir (defer).**

**Pub/Sub ile**, mesaj yayıncısı (publisher) **implicit invocation (örtük çağırma)** kullanır. Publisher, mesajı yayınlayarak **var olan herhangi bir subscriber'ın çalışmasına örtük olarak neden olur**, ama publisher'ın **hangi subscribing servislerin mesajı alacağı üzerinde hiçbir kontrolü yoktur.**

> **Analoji:** Bu farkı bir kurye/posta örneğiyle netleştirelim. **Cloud Tasks**, belirli bir mektubu (task), **belirli bir kuryeye (endpoint)**, **belirli bir teslim zamanıyla** elden teslim etmeye benzer: mektubu kime, ne zaman, hangi adrese göndereceğini **tam olarak sen belirlersin**, ve isteğin üzerine kurye "teslim edildi mi, edilemedi mi" diye sana **rapor verir** (status code, retry). **Pub/Sub** ise bir mahalle ilan panosuna **genel bir duyuru (public notice)** asmaya benzer: duyuruyu asarsın, **kimin okuyacağını, kaç kişinin okuyacağını, hatta okuyup okumayacağını bile bilmezsin** — ilgilenen herkes (subscriber'lar) kendi isteğiyle panoya bakar ve harekete geçer.

> **Sınav tuzağı — Cloud Tasks ve Pub/Sub'ı birbirinin yerine kullanılabilir sanmak:** İkisi de "asenkron mesaj gönder" işini yapıyor gibi göründüğü için karışabilirler, ama **temel felsefeleri zıttır.** Cloud Tasks'ta **hedef bellidir ve tektir** (creator, endpoint'i seçer); Pub/Sub'ta **hedef kümesi (subscriber set) publisher'a bilinmez ve değişken olabilir.** Sınav sorusu "bir uygulama belirli, bilinen bir servisi asenkron olarak çalıştırmak ve belki zamanlamasını kontrol etmek istiyor, hangi servisi kullanmalı?" diye sorarsa cevap **Cloud Tasks**'tır — Pub/Sub değil, çünkü Pub/Sub'ta "hangi servisin mesajı alacağı" konusunda kontrol yoktur.

## Ne zaman hangisi?

**Cloud Tasks**, bir uygulamanın **belirli bir servisi asenkron olarak çalıştırmak** istediği ve muhtemelen **execution'ın zamanlamasını kontrol etmek** istediği kullanım durumları için uygundur. **Pub/Sub**, alıcı servislerin **diğer servisler tarafından üretilen event'lere tepki verdiği** event-tabanlı mimariler için kullanılmalıdır.

Cloud Tasks, bağımsız iş parçalarını asenkron olarak işlenmek üzere **ayırma (separating out)** işini çok iyi yapar. **Yavaş arka plan işlemleri (slow background operations)**, ana uygulamanın yükünü azaltıp yanıt süresini (response time) iyileştirmek için **ayrı bir worker'a devredilebilir (offloaded).** Pub/Sub'ın aksine, uygulama **execution'ın kontrolünü elinde tutar** — offload edilen task'ı işleyecek endpoint'i **kendisi belirtir.** Cloud Tasks, retry, zamanlama ve rate limiting yapılandırmanı sağlar.

**Cloud Tasks vs. Pub/Sub — özet tablo:**

| Özellik | Cloud Tasks | Pub/Sub |
| --- | --- | --- |
| Invocation türü | Explicit invocation | Implicit invocation |
| Kontrol kimde? | Task'ı oluşturan (creator) — hedefi ve zamanlamayı belirler | Publisher'da değil — hangi subscriber'ların alacağı publisher'ın kontrolünde değil |
| Hedef sayısı | Tek, bilinen bir HTTP endpoint | Sıfır, bir ya da birçok subscriber (publisher bilmez) |
| Zamanlama kontrolü | Var — gelecekte belirli bir zamana erteleyebilirsin | Yok — mesaj yayınlanınca subscriber'lara akar |
| Kuyruk yapılandırması | Max dispatch rate, max concurrency, max retry, retry delay | Topic/subscription seviyesinde farklı bir model |
| Teslim garantisi | En az bir kez (at-least-once), duplicate elimination | En az bir kez (at-least-once) |
| Tipik kullanım | Ana isteği yavaşlatan işi arka plana devretmek (offload) | Event-tabanlı mimaride, servislerin başka servislerin event'lerine tepki vermesi |

---

# BÖLÜM 7 — Cloud Scheduler: Cron İşleriyle Zamanlanmış Görevler

## Cloud Scheduler nedir ve neden var?

Orchestration ve choreography tasarımlarında kullanabileceğin bir başka yönetilen servis de **Cloud Scheduler**'dır. **Cloud Scheduler, tek bir dashboard'dan yönetilebilen, tamamen yönetilen (fully managed), kurumsal düzeyde (enterprise-grade) bir cron job zamanlayıcısıdır.**

Cloud Scheduler, **tanımlı bir zamanlamada ya da düzenli aralıklarla çalıştırılması gereken işleri (jobs) zamanlamak** için kullanılır. İşler, **bilindik Unix cron job formatı** kullanılarak belirtilir — bu da bir işin günde birden çok kez ya da belirli günlerde ve aylarda çalıştırılmasına izin verir.

İşler, **Pub/Sub topic'lerine, App Engine uygulamalarına ya da herkese açık (publicly accessible) HTTP uç noktalarına** gönderilebilir. Cloud Tasks gibi, Cloud Scheduler da HTTP isteklerine **belirtilen bir service account'a bağlı token'lar** ekleyebilir. İşlerin **garantili execution'ı** vardır, ve başarısız iş execution'ları **yeniden denenebilir (retried).**

## Unix cron string formatı

Unix cron string formatı, bir işin **ne zaman çalıştırılacağını** belirten, **tek bir satırda, boşlukla ayrılmış beş alandan (fields)** oluşan bir kümedir:

1. **Dakika (minute).** Bu alandaki bir 15, işin **saatin 15. dakikasında** çalışacağı anlamına gelir.
2. **Saat (hour).** Bu alandaki bir 0, işin **gece yarısından sonraki saat içinde** çalışacağı anlamına gelir.
3. ve 4. **Ayın günü (day of month) ve ay (month).** Bu alanlar, execution'ı **belirtilen ayın günleri ve aylarıyla** sınırlar.
4. **Haftanın günü (day of week).** 0 (Pazar) ile 6 (Cumartesi) arasında değişir.

Bir alan; **tek bir sayı**, **bir sayı aralığı (range)** ya da **sayıların ve aralıkların bir listesini** içerebilir:

- **Aralıklar (ranges)**, tire (hyphen) ile ayrılmış iki sayıdır, ve aralık **kapsayıcıdır (inclusive)** — yani her iki uç değer de dahildir.
- **Yıldız işareti (`*`)**, izin verilen **tüm aralığı** gösterir.
- Bir aralığın ardından bir **slash ve bir sayı** gelmesi, aralık boyunca o kadar değeri **atlar (skip).** Örneğin, saat alanındaki bir **`*/2`**, execution'ın **her iki saatte bir** gerçekleşmesi gerektiğini gösterir.
- **Virgülle ayrılmış (comma-separated)** bir sayı ya da aralık listesi belirtilebilir.

> **Analoji:** Cron'un beş alanını, bir toplantı davetiyesindeki **beş ayrı süzgeç (filter)** gibi düşün: "her ayın hangi günlerinde, hangi aylarda, haftanın hangi günlerinde, saatin kaçında, dakikanın kaçında" — bu beş süzgecin **hepsi aynı anda doğru olduğunda**, iş çalışır. `*` demek "bu süzgeci hiç uygulama, her değeri kabul et" demektir; `*/2` demek "bu süzgeçten her ikinci değeri geçir" demektir; bir liste (`1,15`) demek "sadece bu belirli değerleri kabul et" demektir.

## Zaman dilimi (time zone) ve UTC uyarısı

İş zamanlamanı **belirli bir zaman dilimi (time zone)** için belirtebilirsin. **Varsayılan zaman dilimi UTC'dir.**

**Daylight saving time (yaz saati uygulaması)** olan zaman dilimleri, işlerin **atlanmasına (skipped)** (saatler ileri alındığında) ya da **iki kez çalışmasına (run twice)** (saatler geri alındığında) neden olabilir. Bu sorunu önlemek için **UTC zaman diliminin kullanılması önerilir.**

> **Neden bu gerçekten olur, sadece teorik bir uyarı değil?** Saatler ileri alındığında (örneğin gece 02:00, 03:00'e atlar), o saatler arasına denk gelen bir cron zamanlaması (örneğin "her gün 02:30'da çalış") **hiç gerçekleşmez** — çünkü takvimde o an **yaşanmamıştır.** Saatler geri alındığında ise (örneğin gece 03:00, tekrar 02:00'ye döner), aynı saat dilimi **iki kez yaşanır** — ve "her gün 02:30'da çalış" zamanlaması, o gece **iki kez** tetiklenebilir. UTC, DST uygulamayan sabit bir referans olduğu için bu belirsizlikten **tamamen bağışıktır** — bu yüzden ders özellikle zamanlanmış işler için UTC'yi öneriyor.

> **Sınav tuzağı:** "Cloud Scheduler'da zaman dilimini kullanıcının bulunduğu yerel saate göre ayarlamak en doğal seçimdir" düşüncesi, **görev kritikliği (job criticality) açısından risklidir.** Yerel bir zaman dilimi (örneğin "Europe/Istanbul") DST uyguluyorsa, yılda iki kez ya bir işin **atlanması** ya da **iki kez çalışması** riskiyle karşılaşırsın. Sınav sorusu "Cloud Scheduler işleri için hangi zaman dilimi önerilir ve neden?" diye sorarsa, doğru cevap **UTC — çünkü DST kaynaklı atlama/çift çalışma riskini ortadan kaldırır**dır.

**Cron alanları — özet tablo:**

| Alan | Değer aralığı | Örnek | Anlamı |
| --- | --- | --- | --- |
| Dakika | 0-59 | `15` | Saatin 15. dakikası |
| Saat | 0-23 | `0` | Gece yarısından sonraki saat |
| Ayın günü | 1-31 | `*` | Ayın her günü |
| Ay | 1-12 | `*/2` | Her iki ayda bir |
| Haftanın günü | 0-6 (0=Pazar, 6=Cumartesi) | `1,3,5` | Pazartesi, Çarşamba, Cuma |

## Cloud Scheduler'ın rolü

Cloud Scheduler, Google Cloud'un **tekrar eden (recurring) işleri zamanlama çözümüdür.** Zamanlanmış işler **tek bir dashboard'dan** yönetilir. İşler, **endüstri standardı unix-cron formatı** kullanılarak oluşturulur, ve **Pub/Sub mesajlarını tetikleyebilir**, ayrıca **Cloud Run functions, Cloud Run services ya da HTTP uç noktalarını güvenli bir şekilde çağırabilir.**

---

# Beş Aracı Bir Arada Görmek: Application Integration Araç Kutusu

Ders, bu beş servisi (Pub/Sub, Eventarc, Workflows, Cloud Tasks, Cloud Scheduler) Google Cloud'un **"Application Integration" araç kutusu** olarak topluca adlandırıyor. Microservices mimarisiyle geliştirilen tam donanımlı bir uygulama, bu servislerden **herhangi birinden ya da hepsinden** fayda görebilir.

| Araç | Deseni | Ne yapar? | Temel soru | Ne zaman başvurursun? |
| --- | --- | --- | --- | --- |
| **Pub/Sub** | Choreography | Publisher'dan topic'e, topic'ten çoklu subscriber'a asenkron mesajlaşma | Bu event'i kimin tükettiğini bilmeden nasıl yayınlarım? | Kendi custom publish/subscribe akışını kurarken, tam kontrol istediğinde |
| **Eventarc** | Choreography | Birçok GCP/üçüncü taraf kaynağından gelen event'leri kural tabanlı trigger'larla, standart CloudEvents formatında hedeflere yönlendirir | Birçok farklı kaynaktan gelen event'lere nasıl standart biçimde tepki veririm? | Birçok event kaynağıyla çalışırken, Pub/Sub'ı elle yönetmek istemediğinde |
| **Workflows** | Orchestration | GCP servislerini ve API çağrılarını merkezi, stateful, gözlemlenebilir bir süreçte orkestre eder | Bu karmaşık süreci merkezi olarak nasıl kontrol eder, izler, güvenilir kılarım? | HTTP tabanlı microservices'i durable bir sürece zincirlerken, batch/set işlerken |
| **Cloud Tasks** | Ne ikisi (explicit invocation) | Belirli, bilinen bir HTTP servisini asenkron ve isteğe bağlı zamanlamayla çalıştırır | Bu belirli işi, bu belirli servise, belki de belirli bir zamanda nasıl gönderirim? | Ana isteği yavaşlatan işi arka plana devretmek istediğinde |
| **Cloud Scheduler** | Ne ikisi (zamanlama) | Unix-cron formatında zamanlanmış işleri Pub/Sub, App Engine ya da HTTP uç noktalarına gönderir | Bu işi belirli bir zamanlamada, tekrar eden şekilde nasıl çalıştırırım? | Periyodik/tekrar eden görevler (raporlar, temizlik işleri, senkronizasyonlar) için |

> **Bu tablodan çıkarılacak en önemli ders:** Bu beş araç, birbirinin **rakibi değil**, farklı **koordinasyon ihtiyaçlarına** cevap veren tamamlayıcı parçalardır. Pub/Sub ve Eventarc, choreography'nin temel taşlarıdır (Eventarc, Pub/Sub'ın üzerine kurulu bir kolaylık katmanıdır). Workflows, orchestration'ın somut karşılığıdır. Cloud Tasks ve Cloud Scheduler ise "bir servisi ne zaman ve nasıl tetikleyeceğim" sorusuna, choreography ve orchestration eksenine tam oturmayan, ama pratikte sıkça ihtiyaç duyulan iki farklı cevap verir: Cloud Tasks **belirli bir işi, belirli bir hedefe, isteğe bağlı gecikmeyle**; Cloud Scheduler **tekrar eden bir işi, belirli bir zaman deseniyle.**

Modül, bu beş servisin birlikte kullanıldığı, **makine öğrenmesi (machine learning) destekli bir görsel uygulaması** inşa eden bir hands-on lab ile kapanıyor — bu lab, bu dokümanın kapsamı dışındadır (transkript, lab'ın içeriğini ayrıntılandırmıyor).

---

# Toplu Özet (Hızlı Tekrar)

**Modülün ana fikri:** Modül 09'da tanımlanan "microservices'te karmaşıklığın bir kısmı servisler-arası iletişime kayar" gözlemi ve modül 10'da kavramsal olarak tanıtılan "event intermediary" fikri, bu modülde **somut Google Cloud ürünlerine** dönüşüyor. Microservices'i koordine etmenin iki temel deseni — **service choreography** ve **service orchestration** — beş gerçek araçla (Pub/Sub, Eventarc, Workflows, Cloud Tasks, Cloud Scheduler) hayata geçiriliyor.

**İki koordinasyon deseni (BÖLÜM 1):** **Service choreography**, bir dans koreografisine benzer — her servis bağımsız çalışır, tanımlı etkileşim kurallarını takip ederek event alır/gönderir; gevşek bağlıdır ama iş mantığı dağıtıktır, merkezi bir kaynak-doğruluğu yoktur. **Service orchestration**, bir orkestra performansına benzer — merkezi bir orkestratör tüm etkileşimleri kontrol eder; servisler birbirini bilmek zorunda değildir; yüksek seviyeli görünüm ve troubleshooting kolaylığı sağlar, ama orkestratör **tek bir arıza noktasıdır (single point of failure)**.

**Pub/Sub (BÖLÜM 2):** Tamamen yönetilen, gerçek zamanlı mesajlaşma servisi. Publisher, topic'e mesaj gönderir; mesaj saklanır ve her subscriber'ın kuyruğuna dağıtılır. **Pull subscription** subscriber'ın kendi zamanlamasıyla poll etmesidir; **push subscription** mesajın otomatik olarak bir endpoint'e gönderilmesidir. Acknowledge etmek mesajı kuyruktan siler ve **en az bir kez (at-least-once, exactly-once değil)** teslimatı garanti eder. Görsel yeniden boyutlandırma örneği: Cloud Storage bucket → Pub/Sub topic → iki subscriber (Resizing Service, Upload Confirm) → ikisi de Firestore'u günceller, birbirlerinden habersiz.

**Eventarc ve CloudEvents (BÖLÜM 3):** Tamamen yönetilen eventing sistemi; birçok GCP servisini doğrudan ya da Cloud Audit Logs üzerinden, üçüncü taraf sağlayıcıları API üzerinden, Pub/Sub topic'lerini de event kaynağı olarak destekler. **Event trigger**, kural tabanlı bir filtredir. Eventarc, **Pub/Sub'ı taşıma katmanı olarak kullanır** ama topic/subscription yönetimini otomatikleştirir — uygulaman sadece HTTP isteklerini kabul eder, Pub/Sub'a hiç dokunmaz. Event'ler standart **CNCF CloudEvents** formatında teslim edilir, birçok dilde SDK'lar mevcuttur. Eventarc, Pub/Sub'ın **üzerine kurulu bir soyutlama katmanıdır**; birçok kaynak, kural tabanlı basit arayüz ve standart format istediğinde Eventarc, tam kontrol ve custom akış istediğinde doğrudan Pub/Sub tercih edilir.

**Workflows (BÖLÜM 4):** Tamamen yönetilen orkestrasyon platformu; GCP servislerini ve API çağrılarını orkestre ederek stateful, otomatik süreçler oluşturur. Merkezi kaynak-doğruluğudur; her execution loglanır ve gözlemlenebilir; durum tutabilir, retry/poll/bir yıla kadar bekleyebilir. Ürün siparişi örneği: Firestore stok kontrolü → kilitleme → out-of-stock boolean'ına göre dallanma (bu boolean süreç sonunda tekrar kullanılır) → Cloud Run function ya da Cloud Run service çağrısı → Firestore güncelleme → e-posta → (gerekirse) Slack mesajı. HTTP tabanlı microservices'i durable/stateful workflow'lara zincirlemek ve batch/set işleme + sağlam hata yönetimi için idealdir.

**Choreography mi orchestration mı (BÖLÜM 5):** Choreography'de alıcı servis kontrolü elinde tutar; producer, kim tükettiğini bilmez/umursamaz. Ürün siparişi choreography ile de yapılabilir (her servis "hazırım" event'i gönderir) ama görünürlük, hata yönetimi ve retry'ları implemente etmek zordur (troubleshooting, süreç ortasında abort, tedarikçi isteklerinin garantisi). Orchestration, her execution'ı ayrı izler, mantık tek yerde tanımlıdır, güvenilir retry sağlar — ama merkezi, tek-takım/org kontrolü gerektirir; decentralized takımlar/organizasyonlar için choreography daha uygundur. Kural: karmaşık, merkezi yönetilebilir süreçler için orchestration; decentralized servisler/organizasyonlar ya da doğrudan GCP event'lerinden yararlanmak için choreography. Pratikte ikisi birleşir: her servis içinde Workflows (orchestration), servisler arasında Eventarc (choreography).

**Cloud Tasks vs. Pub/Sub (BÖLÜM 6):** Cloud Tasks, büyük sayıda dağıtık task'ın execution/dispatch/delivery'sini yönetir; queue'lar max dispatch rate, max concurrency, max retry/delay ile yapılandırılır; gelecekte zamanlanmış dispatch mümkündür; at-least-once teslimat + duplicate elimination; otomatik service account token'ı. Cloud Tasks ve Pub/Sub kavramsal olarak benzer (mesaj geçişi/asenkron entegrasyon) ama kullanım amaçları farklıdır: Cloud Tasks **explicit invocation** (creator, hedefi ve zamanlamayı tam kontrol eder), Pub/Sub **implicit invocation** (publisher, hangi subscriber'ların alacağını kontrol edemez). Cloud Tasks, belirli bilinen bir servisi asenkron çalıştırmak/zamanlamayı kontrol etmek için; Pub/Sub, event-tabanlı mimarilerde servislerin başka servislerin event'lerine tepki vermesi için kullanılır.

**Cloud Scheduler (BÖLÜM 7):** Tamamen yönetilen, kurumsal düzeyde cron job zamanlayıcısı, tek dashboard'dan yönetilir. Standart Unix cron formatı: beş alan (dakika, saat, ayın günü, ay, haftanın günü 0=Pazar..6=Cumartesi); tek sayı, tire ile kapsayıcı aralık, `*` tüm aralık, `*/N` adım değeri, virgülle liste. İşler Pub/Sub topic'lerine, App Engine uygulamalarına ya da HTTP uç noktalarına gönderilebilir; otomatik service account token'ı; garantili execution + retry. Varsayılan zaman dilimi UTC'dir; **UTC önerilir çünkü DST, saatler ileri alındığında işlerin atlanmasına, geri alındığında iki kez çalışmasına neden olabilir.**

**Application Integration araç kutusu:** Pub/Sub, Eventarc, Workflows, Cloud Tasks ve Cloud Scheduler; bir microservices uygulaması bu beşinden herhangi birinden ya da hepsinden fayda görebilir. Modül, bu beş servisi birlikte kullanan bir ML destekli görsel uygulaması hands-on lab'ıyla kapanır.

---

# En Kritik Ayrımlar / Hızlı Tekrar (Cebinde Taşı)

- **Choreography vs. orchestration analojileri:** Choreography = dans koreografisi (her dansçı bağımsız icra eder, merkez yok). Orchestration = orkestra (şef tüm etkileşimleri kontrol eder, merkezi kaynak-doğruluğu var).
- **Orchestration'ın gizli maliyeti:** Yüksek seviyeli görünüm ve troubleshooting kolaylığı sağlar, ama **orkestratör tek bir arıza noktasıdır (single point of failure)** — "orchestration'ın hiçbir dezavantajı yok" tuzağına düşme.
- **Pub/Sub'ın teslim garantisi:** **En az bir kez (at-least-once)**, **tam olarak bir kez (exactly-once) değil.** Subscriber mantığı idempotent olmalıdır.
- **Pull vs push subscription:** Pull = subscriber kendi zamanlamasıyla poll eder (modül 10'daki polling). Push = mesaj otomatik gönderilir (modül 10'daki push-based messaging).
- **Eventarc ≠ Pub/Sub'ın rakibi:** Eventarc, Pub/Sub'ı **taşıma katmanı** olarak kullanan, üzerine kurulu bir **soyutlama katmanıdır.** Uygulaman sadece HTTP isteklerini kabul eder, Pub/Sub'a hiç dokunmaz. Eventarc'ı seç: çok kaynak + kural tabanlı arayüz + standart CloudEvents formatı istediğinde. Pub/Sub'ı doğrudan seç: tam kontrol istediğin custom bir akış kurarken.
- **CloudEvents'in rolü:** CNCF standardı; kaynak ne olursa olsun aynı event işleme mantığını kullanmanı sağlar; birçok dilde SDK mevcuttur.
- **Workflows'un gücü:** Merkezi kaynak-doğruluğu, her execution loglanır/gözlemlenebilir, state tutabilir/retry/poll/bir yıla kadar bekleyebilir — durable/stateful, HTTP tabanlı microservices zincirlemek ve batch işleme için ideal.
- **Choreography mi orchestration mı — pratik kural:** Merkezi yönetilebilen karmaşık süreç → orchestration (Workflows). Decentralized takımlar/organizasyonlar ya da doğrudan GCP event'lerinden yararlanma → choreography (Pub/Sub, Eventarc). Gerçek dünyada genelde **ikisi bir arada**: servis içinde orchestration, servisler arasında choreography.
- **Cloud Tasks vs Pub/Sub — en kritik ayrım:** Cloud Tasks = **explicit invocation**, tek bilinen hedef, creator zamanlamayı kontrol eder. Pub/Sub = **implicit invocation**, hedef kümesi (subscriber'lar) publisher'a bilinmez/kontrolünde değil. "Belirli bir servisi asenkron çalıştır, zamanlamasını kontrol et" → Cloud Tasks. "Servisler event'lere tepki versin" → Pub/Sub.
- **Cron'un beş alanı:** Dakika, saat, ayın günü, ay, haftanın günü (0=Pazar, 6=Cumartesi). Tek sayı / kapsayıcı aralık (tire) / `*` (tüm aralık) / `*/N` (adım) / virgülle liste.
- **Cloud Scheduler'da UTC tuzağı:** Varsayılan UTC'dir ve **önerilir** — çünkü DST uygulayan zaman dilimlerinde, saatler ileri alındığında işler **atlanabilir**, geri alındığında **iki kez çalışabilir.**
- **Beş araç, tek araç kutusu:** Pub/Sub + Eventarc = choreography'nin temel taşları (Eventarc, Pub/Sub üzerine kurulu). Workflows = orchestration'ın somut karşılığı. Cloud Tasks = belirli/bilinen bir hedefi asenkron tetikleme. Cloud Scheduler = tekrar eden/zamanlanmış tetikleme. Beşi birlikte Google Cloud'un **Application Integration** araç kutusunu oluşturur.

---

> **Kapanış:** Üç modüllük bu yolculuk, kurumsal uygulama mimarisinin tam bir yayını çizdi. Modül 09'da, tek parça (monolithic) uygulamalardan, SOA'nın kurumsal-merkezi yaklaşımından, microservices'in ayrıştırılmış (decentralized) alternatifine geçişi ve bu geçişin getirdiği somut bir zorluğu — **servisler-arası iletişim karmaşıklığını** — öğrendin. Modül 10'da, bu karmaşıklığa kavramsal bir çözüm gördün: event'lerin doğasını (immutable, consume edilmeden de üretilebilir, süresiz/çoklu tüketilebilir) ve bir **event intermediary**'nin producer ile consumer'ı nasıl birbirinden decouple ettiğini. Bu modülde ise, o kavramsal çerçeve nihayet **somut isimler** aldı: **Pub/Sub** ve **Eventarc**, choreography'i; **Workflows**, orchestration'ı Google Cloud'da hayata geçiriyor. Bunlara ek olarak **Cloud Tasks** (belirli bir servisi bilerek ve isteğe bağlı zamanlamayla tetiklemek için) ve **Cloud Scheduler**'ı (tekrar eden işleri zamanlamak için) gördün. En kritik pratik çıkarım şu: choreography ve orchestration birbirinin rakibi değil, **farklı koordinasyon ihtiyaçlarına verilen farklı cevaplardır** — ve gerçek dünyadaki büyük mimariler genellikle her ikisini de, birbirinin içine yerleştirilmiş biçimde kullanır. Bu doküman, kursun bu ana kadar mevcut olan üç modülünü de kapsıyor; kursun ilerleyen modüllerinin transkriptleri eklendiğinde, bu handbook'a yeni numaralı modüller olarak eklenmeye devam edecek.
