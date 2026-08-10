# Modül 9 — Microservices'e Giriş: Pratik Sorular

Bu soru seti şunları kapsıyor: monolithic applications, Service-Oriented Architecture (SOA) ve Enterprise Service Bus (ESB), microservices, monolith ile mi yoksa microservices ile mi başlamalı kararı, ve bir microservices mimarisinin faydaları ile zorlukları. Bu modül, "Service Orchestration and Choreography on Google Cloud" kursunun 1. Modülü'dür.

Sorular, gerçek sınavda insanların gerçekten takıldığı ayrımlara ağırlık veriyor: SOA'da aslında neyin bozulduğu, bir monolith'in ne zaman *doğru* başlangıç noktası olduğu ve microservices'in hangi "faydasının" aslında gizli bir trade-off olduğu.

Önce tüm soruları cevaplamayı dene, ardından cevaplarını aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle karşılaştır.

---

## Sorular

**1.** Bir takım, beş yıllık uygulamalarını "UI, business logic ve data access'in tek bir kod tabanında, tek bir ilişkisel veritabanının üzerinde toplandığı, bir alandaki değişikliğin sık sık ilgisiz bir şeyi bozduğu" bir yapı olarak tanımlıyor. Burada tanımlanan mimari stil nedir ve kırılganlığın (fragility) spesifik nedeni ne?

A. Service-Oriented Architecture; kırılganlık ESB yanlış yapılandırmasından kaynaklanır.
B. Bir monolith; kırılganlık, sürekli büyüyen tek bir kod tabanının parçaları arasındaki sıkı bağlılıktan (tight coupling) kaynaklanır.
C. Microservices; kırılganlık, bağımsız deploy edilen çok fazla servisten kaynaklanır.
D. Bir monolith; kırılganlık çok fazla veritabanı olmasından kaynaklanır.

**2.** Bir takım, servis kodundaki karmaşıklığı azaltmak amacıyla özellikle Service-Oriented Architecture (SOA)'yı benimsedi. Bir yıl sonra, genel sistem karmaşıklığının aslında ortadan kalkmadığını — sadece başka bir yere taşındığını, ve o "başka yerin" artık herhangi bir değişikliği göndermenin (ship etmenin) en büyük tek darboğazı olduğunu bildiriyorlar. Karmaşıklık nereye taşındı ve neden bir darboğaz haline geldi?

A. İlişkisel veritabanı şemasına taşındı, artık her deploy için bir DBA gerekiyor.
B. Genelde tek bir merkezi takım tarafından yönetilen Enterprise Service Bus (ESB) entegrasyonlarına taşındı, bu yüzden neredeyse her uygulama veya servis değişikliği ESB işi gerektirdi ve bu takımın zamanı için rekabet etti.
C. Kayboldu — SOA, tasarımı gereği entegrasyon karmaşıklığını tamamen ortadan kaldırır.
D. İstemci uygulamalarına taşındı, onlar artık kendi mesaj routing'lerini implemente etmek zorunda.

**3.** Bir SOA dağıtımında, bir geliştirici kendi servisinin mesajlarının paylaşılan Enterprise Service Bus (ESB) üzerinden nasıl route edildiğini değiştirmesi gerekiyor. Başka bir takım buna itiraz ederek, bunun kendi uygulamalarını istikrarsızlaştırabileceği konusunda uyarıyor. Bu, (microservices'in aksine) özellikle SOA'da neden gerçekçi bir endişe?

A. Gerçekçi değil — ESB routing değişiklikleri her zaman tek bir servisle sınırlıdır.
B. Çünkü ESB, paylaşılan, merkezileştirilmiş bir entegrasyon noktasıdır; bir uygulamanın entegrasyonundaki değişiklik, aynı ESB üzerinden route edilen diğer uygulamaları etkileyebilir.
C. Çünkü SOA uygulamaları tek bir kod tabanını paylaşır, bu yüzden herhangi bir değişiklik her şeyin tamamen yeniden deploy edilmesini gerektirir.
D. Çünkü ESB'ler, tam bir platform yükseltmesi olmadan routing değişikliklerini hiç desteklemez.

**4.** İki mühendis, yeni projelerinin ilk günden microservices olarak inşa edilip edilmeyeceğini tartışıyor. Biri "microservices kesinlikle daha modern, bu yüzden her zaman oradan başlamalıyız" diye savunuyor. Ancak takım, inşa ettikleri iş alanında (business domain) henüz derin bir uzmanlığa sahip değil. Modülün rehberliği burada aslında neyi önerir?

A. Her koşulda microservices ile başla — uzmanlık, servis sınırları çizildikten sonra geliştirilebilir.
B. Bir monolith ile başla ve onu modüler tasarla, çünkü servis sınırlarını tasarlamak yeni bir microservices projesinin en zor kısımlarından biridir ve domain'i anladıktan sonra bu daha kolaydır.
C. Monolith ile microservices arasında bir orta yol olarak SOA ile başla.
D. Karar önemsizdir — modül, mimari stilin domain uzmanlığıyla hiçbir ilgisi olmadığını belirtir.

**5.** Bir startup, önümüzdeki iki yılda mühendislik takımının önemli ölçüde büyüyeceğini öngörüyor ve yeni işe alınanların önce tüm büyük kod tabanını öğrenmeden katkıda bulunabilmesini istiyor. Modül bu spesifik endişe için hangi mimari başlangıç noktasını önerir ve neden?

A. Bir monolith, çünkü tek bir kod tabanına yeni mühendisleri bütün olarak onboard etmek daha kolaydır.
B. Microservices, çünkü doğal servis sınırları her takım üyesinin tüm uygulamayı değil, sistemin daha küçük, sınırları belli bir parçasına odaklanmasını sağlar.
C. SOA, çünkü ESB tüm sistemi yeni işe alınanlar için otomatik olarak dokümante eder.
D. Hiçbiri — modüle göre takım büyümesinin mimari seçimiyle bir ilgisi yoktur.

**6.** Bir microservices mimarisinde, Orders, Products ve Reviews domain'lerinin her birinin kendi veritabanı vardır ve kendi API'leri üzerinden çağrılırlar. Bir takım arkadaşı, bunun sadece farklı bir isimle SOA ile fonksiyonel olarak aynı olduğunu varsayıyor. Gerçekte ne farklı?

A. Hiçbir şey — microservices ve SOA, farklı pazarlama terimleri altında aynı desendir.
B. Microservices, tıpkı SOA gibi her çağrıyı paylaşılan bir Enterprise Service Bus üzerinden yönlendirir.
C. Microservices decentralized (merkeziyetsiz) bir yaklaşımdır: paylaşılan bir ESB middleware'i yoktur — servisler birbirinin API'lerini doğrudan çağırır ve her servis kendi veritabanına sahiptir.
D. Microservices, SOA'nın aksine tüm servislerin tek bir merkezi veritabanını paylaşmasını gerektirir.

**7.** Bir takım monolith'ten microservices'e geçiyor ve hiçbir tekil servis yavaşlamadığı halde, eskiden birkaç milisaniyede tamamlanan tek bir iş operasyonunun artık gözle görülür biçimde daha uzun sürmesine şaşırıyor. En olası mimari açıklama nedir?

A. Microservices her zaman monolithlerden daha yavaş donanımda çalışır.
B. Operasyon artık tek bir uygulama içindeki in-process çağrılar yerine servisler arasında ağ üzerinden birden fazla çağrı içeriyor ve ağ çağrıları in-process çağrılardan çarpıcı biçimde daha yavaştır — birçok çağrı zincirlendiğinde bu birikir.
C. Bu bir hatadır — doğru yapılandırılmış microservices her zaman bir monolithten daha hızlı olmalıdır.
D. Migration sırasında veritabanının kendisi yanlış yapılandırılmış olmalı.

**8.** Microservices inşa eden bir takım, her takımın kendi programlama dilini ve framework'ünü seçmesine izin veriyor ve farklı servisler farklı platformlarda çalışan farklı dillerde yazılıyor. Şüpheci biri bunun sistemi bozacağından endişe ediyor. Bu, bir microservices mimarisinde neden genelde sorunsuz çalışır?

A. Çalışmaz — tüm microservices'in birlikte çalışabilmesi için aynı dilde yazılması gerekir.
B. Servisler kendi API arayüzleri (genelde HTTP) üzerinden birlikte çalışır, bu yüzden bir servisin içinde kullanılan dil, framework ya da platform, onu çağıran servisler için görünmezdir.
C. Yalnızca her servis ESB ile aynı dilde yazılırsa çalışır.
D. Yalnızca Kubernetes ağ katmanında diller arasında otomatik çeviri yaptığı için çalışır.

**9.** Bir trafik sıçraması sırasında, bir microservice (checkout), normal yükte çalışan diğerlerinden (search, recommendations) önemli ölçüde daha fazla compute kapasitesine ihtiyaç duyuyor. Bir monolith'te, checkout'un sıçramasını karşılamak için tüm uygulamayı ölçeklendirmeniz gerekirdi. Buradaki microservices'e özgü avantaj nedir?

A. Microservices her zaman trafikten bağımsız sabit bir kapasitede çalışır, bu yüzden bu senaryo hiç oluşamaz.
B. Her microservice bağımsız olarak ölçeklenebilir, böylece tüm uygulamayı aşırı provizyonlamak yerine özellikle checkout'a daha fazla kaynak ayırabilirsiniz.
C. Bu avantaj yalnızca SOA'ya özgüdür ve microservices için geçerli değildir.
D. Bağımsız ölçeklenme, API'lerden vazgeçip doğrudan veritabanı erişimine geçmeyi gerektirir.

**10.** Bir organizasyon 3 monolitik uygulamadan yaklaşık 80 microservice'e geçiyor. Altı ay sonra operasyon takımı, hepsinde build, test ve deployment'ı sorunsuz çalışır tutmakla bunalmış durumda olduklarını söylüyor. Modül, altta yatan nedeni ve gerekli önlemi ne olarak tanımlıyor?

A. Bu mimari değişiklikle ilgisizdir — bir işe alım sorunudur.
B. Çok daha fazla deploy edilebilir varlığa sahip olmak çok daha büyük bir operasyonel yük yaratır; otomatik build, test ve deployment bunu yönetilebilir tutmak için hayati önemdedir.
C. Microservices tam olarak bu nedenle asla 10 servisi geçmemelidir.
D. Çözüm, operasyonel yüzeyi azaltmak için 80 servisin tamamını tekrar tek bir ESB'ye konsolide etmektir.

**11.** Bir güvenlik mühendisi, microservice sayısı arttıkça log formatlarının, authorization kontrollerinin ve reporting'in takımlar arasında tutarsız hale geldiğini ve bunun denetimleri (audit) eski monolith'e göre çok daha zorlaştırdığını fark ediyor. Bu, microservices'in hangi temel zorluğunu gösteriyor?

A. Microservices, hiçbir hafifletme yolu olmadan güvenliği kesinlikle kötüleştirir.
B. Birbirinden bağımsız geliştirilen birçok servisle, hepsinde tutarlı logging, reporting, security ve authorization sağlamak, kasıtlı çaba gerektiren belirgin bir zorluk haline gelir.
C. Bu yalnızca servisler farklı programlama dillerinde yazıldığında olur.
D. Her servis izole olduğu için microservices mimarisinde güvenlik endişeleri önemsizdir.

**12.** Yeni bir mühendis, bir organizasyonun 60 microservice'i arasında isteklerin nasıl aktığını gösteren bir diyagram çiziyor ve bunu "okunamaz bir örümcek ağı" olarak tanımlıyor — hangi servisin hangisine bağımlı olduğunu anlamak neredeyse imkansız. Modül bunun kök nedenini ne olarak söylüyor ve bu kaçınılmaz mı?

A. Kaçınılmazdır — 10'dan fazla servise sahip herhangi bir sistem her zaman okunamaz hale gelir.
B. Yalnızca servisler farklı dillerde yazıldığında olur.
C. Kötü tasarlanmış servisler arası iletişimden kaynaklanır; modül bunu, microservices kullanmanın kaçınılmaz bir sonucu değil, kasıtlı tasarımla yönetilmesi gereken gerçek bir risk olarak çerçeveliyor.
D. Takımın tüm servisleri derhal tek bir monolith'e geri birleştirmesi gerektiğinin bir işaretidir.

**13.** Bir QA lideri, her bir microservice için unit test'lerin hızlı ve basit olduğundan ama *tüm sistemin* doğru davrandığını doğrulamanın neredeyse production ortamının tamamını ayağa kaldırmayı gerektirdiğinden şikayet ediyor. Modül, bunun bir microservices mimarisinde neden beklenen bir şey olduğunu söylüyor?

A. Beklenmez — microservices'te integration testing, unit testing kadar basit olmalıdır.
B. Microservices'in dağıtık doğası, genelde tüm sistemi test etmenin tüm production dağıtımını modellemeyi gerektirdiği anlamına gelir; bileşenlerin aynı process içinde çalıştığı bir monolith'in aksine.
C. Bu yalnızca servisler tek bir veritabanını paylaştığında olur.
D. Her servis için unit test'ler geçtiğinde integration testing gereksizdir.

**14.** Bir olaydan sonra, bir mühendisin on iki farklı microservice'e dokunan, her biri kendi ayrı loglarını yazan başarısız bir tek iş operasyonunu izlemesi gerekiyor. Bunun, takımın eski monolith'indeki eşdeğer akışı debug etmekten çok daha zor olduğunu bildiriyor. Modül bunun nedenini ne olarak tanımlıyor?

A. Microservices hiç log üretmez, bu yüzden izlenecek bir şey yoktur.
B. Her microservice kendi loglarını oluşturduğu için, birçok microservice'e yayılan bir çağrıyı izlemek, tek bir monolitik process içindeki çağrıları izlemekten doğası gereği daha zordur.
C. Bu mimariyle ilgisizdir — tamamen bir araç bütçesi sorunudur.
D. Debugging microservices'te daha kolaydır çünkü her servis daha küçüktür.

**15.** Bir liderlik takımı microservices'i benimseyip benimsememeyi tartıyor. Şunu soruyorlar: "faydalar gerçekten zorluklardan ağır mı basıyor?" Modülün çerçevesine göre, en doğru cevap nedir?

A. Hayır — microservices'in zorlukları her zaman faydalarından ağır basar, bu yüzden monolithler kesinlikle üstündür.
B. Evet, koşulsuz olarak — microservices, benimsendikten sonra anlamlı bir dezavantaja sahip değildir.
C. Genelde evet, ama yalnızca otomasyona ve operasyonel mükemmelliğe gerçek bir bağlılıkla; bu araçlara yatırım yapmadan, birçok bağımsız deploy edilebilir servisin operasyonel yükü faydaları geride bırakabilir.
D. Trade-off önemsizdir — seçim yalnızca takım büyüklüğüne dayanmalıdır, asla operasyonel hazırlığa değil.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
UI, business logic ve data access'i tek bir veritabanı üzerinde birleştiren, ilgisiz değişikliklerin bozulmaya neden olduğu tek bir kod tabanı, ders kitabı tanımıyla bir monolith'tir — ve modül bu kırılganlığı özellikle tek uygulama içindeki sıkı bağlılığa (tight coupling) atfeder, veritabanı sayısına ya da ESB davranışına değil.

**2. Doğru cevap: B.**
SOA, tekil servislerin içindeki karmaşıklığı azalttı, ama modül bu karmaşıklığın ortadan kalkmadığını, *taşındığını* açıkça belirtir — ESB entegrasyonlarına taşındı, bunlar genelde merkezi olarak yönetiliyordu, bu da ESB işini her uygulama/servis takımının rekabet etmek zorunda kaldığı bir darboğaza dönüştürdü.

**3. Doğru cevap: B.**
ESB paylaşılan, merkezileştirilmiş bir mesajlaşma katmanı olduğu için, bir uygulamanın entegrasyonundaki bir routing değişikliği, aynı bus üzerinden route edilen diğer uygulamaları gerçekten istikrarsızlaştırabilir — bu merkezi-darboğaz riski, tam olarak microservices'in decentralized, servis-başına API'lerinin önlemek için tasarlandığı şeydir.

**4. Doğru cevap: B.**
Modül bu konuda nettir: problem domain'inde uzmanlığın yoksa, servis sınırlarını tasarlamak yeni bir microservices projesinin en zor kısımlarından biridir, bu yüzden modüler bir monolith ile başlayıp daha sonra — domain'i anladıktan sonra — geçiş yapmak önerilen yoldur. "Microservices her zaman modern varsayılandır" (A) tuzaktır.

**5. Doğru cevap: B.**
Modül, takım büyümesini özellikle microservices'e bağlar: doğal servis sınırları, yeni takım üyelerinin tüm uygulamayı baştan sona öğrenmek yerine sistemin daha küçük bir parçasına odaklanmasını sağlar.

**6. Doğru cevap: C.**
Microservices, SOA'ya açıkça decentralized bir alternatif olarak çerçevelenir — paylaşılan bir ESB middleware bileşeni yoktur; servisler kendi API'leri üzerinden birbirini doğrudan çağırır ve her servis kendi veritabanına sahiptir. Microservices'i "yeni isimli SOA" olarak ele almak (A) bu yapısal farkı kaçırır.

**7. Doğru cevap: B.**
Microservices arasındaki çağrılar ağ üzerinden geçer; modül bunu bir monolith içindeki in-process çağrılardan binlerce kat daha yavaş olarak tanımlar; tek bir iş operasyonu birçok zincirlenmiş microservice çağrısı gerektirdiğinde, bu gecikme birikir ve fark edilir hale gelir — hiçbir tekil servis yavaş olmasa bile.

**8. Doğru cevap: B.**
Microservices bir API arayüzü üzerinden bağlanır, bu yüzden bir servisin içinde kullanılan dil, framework ve teknoloji, onu çağıran her şey için görünmezdir — farklı takımların birlikte çalışabilirliği bozmadan farklı teknoloji yığınları seçmesini sağlayan tam olarak budur.

**9. Doğru cevap: B.**
Bağımsız ölçeklenme, temel bir microservices faydasıdır: her servis ayrı deploy edilebildiği için, bir monolith'te olduğu gibi tüm uygulamayı ölçeklendirmek (ve bunun için ödeme yapmak) yerine, yük altındaki servise (checkout) özellikle daha fazla kaynak ayırabilirsiniz.

**10. Doğru cevap: B.**
Daha fazla deploy edilebilir varlık doğrudan daha büyük bir operasyonel yüke dönüşür; modül, çok sayıda microservice'i sağlıklı ve yönetilebilir tutmak için otomatik build, test ve deployment'ın hayati olduğunu açıkça belirtir — bu bir işe alım sorunu ya da servis sayısında sert bir tavan değil, bir otomasyon gereksinimidir.

**11. Doğru cevap: B.**
Modül, birçok servis genelinde tutarlı logging, reporting, security ve authorization'ı, microservices mimarilerinin açık bir zorluğu olarak listeler — servis sayısı arttıkça bu otomatik olarak gerçekleşmez ve tek bir programlama diline bağlı değildir.

**12. Doğru cevap: C.**
Servisler arası iletişimin "örümcek ağı", *sistemlerin iyi tasarlanmamasının* bir sonucu olarak tanımlanır — belirli bir servis sayısını geçen herhangi bir sistemin kaçınılmaz kaderi ya da yalnızca çoklu-dil (polyglot) kurulumlarla sınırlı bir şey değil, kasıtlı tasarımla aktif olarak yönetilmesi gereken gerçek bir risktir.

**13. Doğru cevap: B.**
Microservices dağıtık olduğu için, sistem davranışını tam olarak doğrulamak genelde tüm production dağıtımını modellemeyi gerektirir — bu, tüm bileşenlerin aynı process içinde birlikte çalıştığı ve bir bütün olarak daha doğrudan test edilebildiği bir monolith'ten temelde farklıdır.

**14. Doğru cevap: B.**
Her microservice'in kendi ayrı loglarını üretmesi, tam olarak birçok servise yayılan bir isteği izlemenin, her şeyin tek bir yerde daha doğal biçimde görünür olduğu tek bir monolitik process'i debug etmekten neden daha zor olduğunun nedenidir — bu, dağıtık mimarinin doğrudan bir sonucudur, bir araç bütçesi sorunu değil.

**15. Doğru cevap: C.**
Modülün kendi sonucu, microservices'in faydalarının genelde zorluklardan ağır bastığı, ama koşullu olarak — yalnızca otomasyona ve operasyonel mükemmelliğe gerçek bir bağlılıkla. İki uç noktayı da (A: monolithler kesinlikle üstündür, ya da B: microservices'in hiçbir dezavantajı yoktur) sunmak bu koşullu çerçeveyi yanlış yansıtır.
