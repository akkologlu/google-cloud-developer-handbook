# Modül 9 – Microservices'e Giriş

> Bu, "Service Orchestration and Choreography on Google Cloud" kursunun 1. Modülü'dür. Bu kursun sonraki modülleri (Google Cloud'da servis orkestrasyonu ve koreografi desenleri), transkriptleri eklendikçe bu handbook'a kendi numaralı modülleri olarak eklenecek.

---

# 🎯 Genel Bakış

Servisleri nasıl orkestre edeceğini ya da koreografize edeceğini konuşmadan önce, servislerin neden ayrı birimler olarak var olduğunu bilmen gerekir.

Bu ders, uygulama mimarisinin tek bir kod tabanından bağımsız olarak dağıtılabilen microservices'e nasıl evrildiğini anlatıyor ve bu geçişin somut artılarını/eksilerini ortaya koyuyor.

```text
Monolith

↓ (yeniden kullanılabilir servislere ayrıştır)

Service-Oriented Architecture (SOA)

↓ (merkezîleştir, paylaşılan middleware'i kaldır → decentralize)

Microservices
```

---

# Monolithic Applications (Monolitler)

Erken dönem kurumsal uygulamalar tek, kendi kendine yeten bir kod tabanı olarak inşa edildi: UI, iş mantığı (business logic) ve veri erişimi hepsi tek bir uygulamada, tek büyük bir ilişkisel veritabanının üzerinde.

```text
Monolith
├── User Interface
├── Business Logic
└── Data Access → İlişkisel Veritabanı
```

- Her büyük değişiklikle kod tabanı daha da karmaşıklaşır.
- Her şey tek bir uygulamada yaşadığı için kod genelde **sıkı bağlı (tightly coupled)** hale gelir.
- Sıkı bağlı kod bakımı zordur — bir hatayı düzeltmek yeni bir hata yaratma riski taşır.

---

# Service-Oriented Architecture (SOA)

SOA, bir uygulamayı yeniden kullanılabilir **service**'lere ayırarak monolitleri düzeltme girişimiydi; her servis ayrı bir iş fonksiyonunu yürütür ve tanımlı arayüzler üzerinden messaging ile iletişim kurar.

```text
Service A   Service B   Service C
    \           |           /
     \          |          /
        Enterprise Service Bus (ESB)
   (bağlantı, güvenlik, routing, transformation)
```

**SOA'nın doğru yaptığı şeyler:**

- Servisler, bir monolitten daha küçük ve daha gevşek bağlıydı (loosely coupled).
- Daha küçük servisler → daha küçük, odaklanmış takımlar.
- Uygulamalar, yeniden kullanılabilir servisler birleştirilerek oluşturuldu.

**SOA'nın bozulduğu noktalar:**

- Servisler arası tüm mesajlaşma, merkezi bir **Enterprise Service Bus (ESB)** üzerinden geçiyordu — protokol dönüşümü, routing ve veri dönüşümünü halleden bir messaging middleware bileşeni.
- Karmaşıklık ortadan kalkmadı, servislerden **ESB entegrasyonlarına kaydı.**
- ESB genelde tek bir merkezi takım tarafından yönetiliyordu, bu yüzden herhangi bir uygulama için entegrasyon işi bir darboğaza dönüştü.
- Bir uygulamanın entegrasyonunu değiştirmek, aynı ESB'yi paylaşan diğer uygulamaları istikrarsızlaştırabilirdi.
- ESB yazılımının kendi güncellemeleri bile mevcut entegrasyonları bozma riski taşıyordu, bu yüzden ciddi test gerektiriyordu.

---

# Microservices

Microservices, SOA'ya **decentralized (merkeziyetsiz)** bir alternatiftir: her biri kendi veritabanına sahip, kapsamı sınırlı, ayrı servisler; diğer servislerin çağırdığı bir API sunar.

```text
Orders Service        Products Service       Reviews Service
   + kendi DB'si          + kendi DB'si          + kendi DB'si
      \                      |                       /
       \                     |                      /
       Çağrılar doğrudan API'ler üzerinden yapılır (paylaşılan ESB yok)
```

- Microservices arasındaki ayrım, **loose coupling**'e (gevşek bağlılığa) yol açar.
- Gevşek bağlı servisleri bakımı yapmak, güncellemek ve dağıtmak (deploy etmek) daha kolaydır.

## Microservices ile mi Başlamalı, Monolith ile mi?

| Durum | Önerilen başlangıç noktası |
| --- | --- |
| Servis sınırlarını çizecek kadar problem domain'i henüz iyi anlamıyorsun | **Monolith** ile başla, domain'i öğrendikçe daha sonra microservices'e geç |
| Hızlı değişiklik yayınlaman ve agile şekilde iterasyon yapman gerekiyor | **Microservices** ile başla |
| Takımın zamanla önemli ölçüde büyüyecek | **Microservices** ile başla — doğal servis sınırları, yeni üyelerin daha küçük bir parçaya odaklanmasını sağlar |

Monolith ile başlıyorsan, ileride microservices'e geçişi kolaylaştırmak için onu **modüler** tasarla.

> **Sınav tuzağı:** "Microservices her zaman doğru ilk seçimdir" yanlıştır. Domain uzmanlığı olmadan servis sınırlarını tasarlamak, yeni bir microservices projesinin en zor kısımlarından biridir — bir monolith, domain'i daha iyi anlayana kadar bu kararı ertelemeni sağlar.

---

# Microservices'in Faydaları

| Fayda | Neden önemli |
| --- | --- |
| Daha basit kod tabanı | Her microservice daha küçüktür, küçük bir takımın onu tam olarak anlaması kolaydır |
| Daha kolay unit testing | Net servis sınırları, izole test yapmayı kolaylaştırır |
| Bağımsız deploy edilebilirlik | Takımlar kendi servislerini kendi takvimlerinde günceller ve deploy eder; diğer servisler yalnızca breaking bir interface değişikliğinde etkilenir |
| Daha agile geliştirme | Servisler, sistemin geri kalanına dokunmadan güncellenebilir ve deploy edilebilir |
| Çoklu teknoloji (polyglot) seçimi | Her takım kendi servisine en uygun dili/framework'ü seçebilir — çağıranlar yalnızca API'ye bağımlıdır, implementasyona değil |
| Platformlar arası birlikte çalışabilirlik | Farklı platformlardaki servisler yine de HTTP API'leri üzerinden birbirini çağırabilir |
| Bağımsız ölçeklenebilirlik (scaling) | Her servis kendi trafiğine göre ölçeklenir, altyapı maliyetini her yerde zirve yüke göre aşırı provizyon yapmak yerine optimize eder |

---

# Microservices'in Zorlukları

| Zorluk | Neden zor |
| --- | --- |
| Operasyonel yük | Onlarca, yüzlerce ya da binlerce deploy edilebilir varlığın yönetilebilir kalması için otomatik build, test ve deployment gerekir |
| Servisler arası tutarlılık | Servis sayısı arttıkça logging, reporting, security ve authorization tutarlı kalmalıdır |
| İletişim karmaşıklığı | Kötü tasarlanmış servisler arası iletişim, anlaşılması zor bir "örümcek ağına" dönüşür |
| Ağ gecikmesi (latency) | Microservices arasındaki çağrılar ağ üzerinden geçer — bir monolitteki in-process çağrılardan binlerce kat daha yavaştır; bir iş operasyonu birçok çağrı gerektirdiğinde gecikme birikir |
| Integration testing | Sistemin tamamını gerçekçi biçimde test etmek, her servisi izole test etmek değil, tüm production dağıtımını modellemeyi gerektirir |
| Debugging | Her microservice kendi loglarını üretir, bu yüzden birçok servise yayılan bir isteği izlemek, tek bir process'i debug etmekten daha zordur |

> Bir microservices mimarisi, ancak otomasyona ve operasyonel mükemmelliğe gerçek bir bağlılıkla karşılığını verir — faydalar genelde zorluklardan ağır basar, ama yalnızca eklenen karmaşıklığı yönetecek araçlara yatırım yaparsan.

---

# Monolith vs. SOA vs. Microservices

| | Monolith | SOA | Microservices |
| --- | --- | --- | --- |
| Bağlılık (coupling) | Sıkı bağlı (tightly coupled) | Gevşek bağlı, ama ESB-merkezli | Gevşek bağlı, decentralized |
| İletişim | In-process çağrılar | Merkezi ESB üzerinden messaging | Servisler arası doğrudan API çağrıları |
| Veri | Tek paylaşılan veritabanı | Genelde paylaşılan/merkezi yönetilen | Her servis kendi veritabanına sahip |
| Deploy edilebilirlik | Tek bir deploy edilebilir birim | Servisler deploy edilebilir, ama ESB değişiklikleri darboğaz | Servis başına tamamen bağımsız deployment |
| Darboğaz | Kod tabanının kendisi | Merkezi ESB ve onu yöneten takım | Operasyonel araçlar ve servisler arası tutarlılık |

---

# Modül Özeti

Uygulamalar, **monolithlerden** (başlaması basit ama ölçekte sıkı bağlı ve bakımı zor) **SOA**'ya (yeniden kullanılabilir servisler, ama karmaşıklık merkezi olarak yönetilen ve yeni darboğaz haline gelen bir ESB'ye kaydı) ve oradan **microservices**'e (decentralized, API'ler üzerinden doğrudan iletişim kuran, bağımsız deploy edilebilir servisler) evrildi.

Microservices'i ilk günden seçmek otomatik olarak doğru değildir — domain uzmanlığı olmadan servis sınırlarını tasarlamak gerçekten zordur, bu yüzden modüler bir monolith ile başlayıp daha sonra geçiş yapmak genelde daha güvenli bir yoldur. Microservices; daha basit kod tabanları, daha kolay test, bağımsız deploy edilebilirlik, teknoloji özgürlüğü ve bağımsız ölçeklenme sunar — bunun bedeli ise operasyonel yük, servisler arası tutarlılık işi, ağ gecikmesi, daha zor integration testing ve daha zor debugging'dir.

---

# Önemli Noktalar

- Monolithler başlaması basittir ama büyüdükçe sıkı bağlı ve bakımı zor hale gelir.
- SOA, bir Enterprise Service Bus (ESB) üzerinden bağlanan yeniden kullanılabilir servisler getirdi — ama ESB'nin kendisi merkezi bir darboğaza dönüştü.
- Microservices daha da decentralize eder: her servis kendi verisine sahiptir ve bir API sunar, paylaşılan bir middleware katmanı yoktur.
- Problem domain'ini henüz anlamıyorsan monolith ile başla; agile teslimat gerekiyorsa ya da takımın önemli ölçüde büyüyecekse microservices ile başla.
- Microservices'in ana faydaları: daha basit kod tabanları, daha kolay test, bağımsız deploy edilebilirlik, çoklu teknoloji (polyglot), bağımsız ölçeklenme.
- Microservices'in ana zorlukları: operasyonel yük, servisler arası tutarlılık, iletişim karmaşıklığı, ağ gecikmesi, integration testing ve dağıtık loglar üzerinde debugging.
