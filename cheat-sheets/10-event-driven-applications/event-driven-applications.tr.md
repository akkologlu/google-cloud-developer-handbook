# Modül 10 – Event-Driven Applications

> Bu, "Service Orchestration and Choreography on Google Cloud" kursunun 2. Modülü'dür (1. Modül [Microservices'e Giriş](../09-introduction-to-microservices/introduction-to-microservices.tr.md) idi). Bu derste henüz belirli bir Google Cloud ürünü adı geçmiyor — konu, kavramsal/mimari düzeyde kalıyor. Bu kursun sonraki modülleri, transkriptleri eklendikçe bu handbook'a kendi numaralı modülleri olarak eklenecek.

---

# 🎯 Genel Bakış

Modül 9, bir sorunla bitmişti: microservices arasındaki point-to-point iletişim, okunamaz bir "örümcek ağına" dönüşme eğilimindedir ve her servis, doğrudan çağırdığı her downstream servise bağımlı (coupled) hale gelir.

Event-driven architecture, tam olarak bu sorunu çözen desendir — servislerin birbirini doğrudan çağırması yerine aralarına bir **event intermediary** koyarak.

```text
Point-to-point (Modül 9'un sorunu)

Service A ──→ Service B
    │             │
    └────→ Service C ←────┘

Event-driven (bu modülün çözümü)

Service A ──┐              ┌──→ Service C
            ├─→ Event Intermediary ─┤
Service B ──┘              └──→ Service D
```

---

# Event Nedir?

Bir **event**, gerçekleşmiş bir şeyin kaydıdır (record) — bir çalışanın uygulamaya login olması, bir ürünün sepete eklenmesi gibi.

Bu tanım bariz görünüyor, ama sade tanımdan daha önemli olan üç özellik var:

| Özellik | Ne anlama gelir |
| --- | --- |
| Immutable (değişmez) | Bir event, bir olayın tarihsel kaydıdır. Sonradan asla değiştirilmemeli ya da silinmemelidir. |
| Consumption'dan bağımsız | Bir event, hiç consume edilmese bile üretilebilir. Producer genelde bir şeyin onu dinleyip dinlemediğini bilmez, umursamaz. |
| Kalıcı ve replayable | Bir event süresiz olarak persist edilebilir ve gerektiği kadar çok kez consume edilebilir — tek bir event, birden fazla servis tarafından paralel olarak consume edilebilir. |

> Bu üç özellik, bir event'i bir request/response çağrısındaki bir request'ten temelde farklı kılan şeydir. Bir request, birinin ona hemen tepki vermesini bekler ve işlendikten sonra ortadan kalkar. Bir event ise, gelecekte herhangi bir anda sıfır, bir ya da birçok consumer tarafından okunabilecek kalıcı bir gerçektir.

---

# Point-to-Point'ten Event Intermediary'ye

Modül 9'da görüldüğü gibi, microservices arasındaki point-to-point iletişim, her servisin her downstream servisle nasıl konuşacağını bilmesini gerektirir — bu coupling yaratır ve anlaşılması zor bir "örümcek ağına" dönüşebilir.

Bir event-driven architecture, servisler arasına bir **event intermediary** ekler:

- **Event producer** olarak davranan bir servis, event'leri intermediary'ye gönderir. Bu event'leri kimin consume edeceği hakkında hiçbir şey bilmesine gerek yoktur.
- **Event consumer** olarak davranan bir servis, event'leri intermediary'den alır. Bu event'leri kimin ürettiği hakkında hiçbir şey bilmesine gerek yoktur.

```text
Producer → sadece event formatını bilir → Event Intermediary → route eder → Consumer(lar)
```

> **Sınav tuzağı:** Bir event intermediary'yi, Modül 9'daki Service-Oriented Architecture tartışmasından bir Enterprise Service Bus (ESB) ile karıştırma. ESB, her entegrasyon değişikliğinin kendisinden ve onu yöneten takımdan geçmesi gerektiği için tam olarak bir darboğaza dönüşen merkezi bir routing/transformation katmanıydı. Event intermediary'nin rolü daha dar ve ruhen daha decentralized'dır — producer'ları consumer'lardan ayırır, böylece hiçbir taraf diğeri hakkında bir şey bilmek zorunda kalmaz, ama ESB'nin dönüştüğü türden merkezi bir entegrasyon-değişikliği darboğazına dönüşmez.

---

# Event-Driven Applications'ın Faydaları

## 1. Merkezi Auditing ve Access Control

Merkezi bir event service, dağıtık bir uygulama için auditing ve control'ü basitleştirir:

- Immutable event'lerin bir logu, auditing amacıyla kullanılabilir — bir uygulamanın state'indeki her değişikliğin zamanlanmış, sıralı bir kaydını verir.
- Event service'te authentication ve authorization zorunlu kılarak, event-based servislerine ve verine erişimi tek bir yerden kontrol edebilirsin.

## 2. Decoupling

Bir event intermediary ile, producer'lar ve consumer'lar decouple edilir (birbirinden ayrılır):

- Bir servis, onu consume edene doğrudan istek göndermeden bir event yaratabilir.
- Bir servis, kim ürettiğini bilmeden bir event'i consume edebilir.
- Producer'ların ve consumer'ların yalnızca belirli bir event'in **formatı** üzerinde anlaşması gerekir — başka hiçbir şey değil.
- Mevcut hiçbir servisi değiştirmeden uygulamaya **yeni event consumer'lar eklenebilir.**

Point-to-point örümcek ağını ortadan kaldıran şey tam olarak budur: her event intermediary üzerinden geçer, o da event'i doğru consumer'a ya da consumer'lara route eder.

## 3. Asenkron İşlemle Resilience

Senkron request/response çağrıları üzerine kurulu microservices'in yapısal bir zayıflığı vardır: bir servisin sağlığı, doğrudan ya da dolaylı olarak çağırdığı her servisin sağlığından etkilenir. Tek bir başarısız servis, tüm zinciri çökertebilir.

Event-driven architecture, event'leri **asenkron** olarak, bir yanıt beklemeden üretir:

- Sistem, bir servisin geçici kaybına dayanacak şekilde tasarlanabilir.
- Sağlıksız bir servise gönderilen event'ler, o servis geri geldiğinde **replay edilebilir ya da yeniden teslim edilebilir (redelivered).**

| | Senkron request/response | Event-driven (asenkron) |
| --- | --- | --- |
| Başarısızlık davranışı | Çöken bir servis, çağrı zincirinde yukarı doğru hataları katlayarak yayabilir (cascade) | Çöken bir consumer sadece geride kalır; event'ler replay/redelivery için bekler |
| Coupling | Çağıran, çağrılanı doğrudan bilmeli ve ona ulaşmalıdır | Producer ve consumer sadece bir event formatını paylaşır |
| Consumption | Tam olarak bir çağıran, tam olarak bir yanıt bekler | Aynı event'i sıfır, bir ya da birçok consumer paralel olarak işleyebilir |

---

# Push-Based Messaging vs. Polling

Asenkron, event-driven servisler genelde polling yerine **push-based** teslimat kullanır:

- **Polling** — consumer'lar kaynağa sürekli "yeni bir şey var mı?" diye sorar. Bu, network I/O'yu artırır ve işin alınmasından önce gereksiz gecikme ekler.
- **Push-based messaging** — consumer'lar, consume edilecek bir event olduğunda otomatik olarak bilgilendirilir; event'ler, consumer'ın sormasına gerek kalmadan verimli biçimde consumer'lara route edilir.

```text
Polling: Consumer → "yeni bir şey var mı?" → Kaynak → "hayır" → Consumer → "yeni bir şey var mı?" → ... (tekrar)

Push:    Event Intermediary → bildirir → Consumer (yalnızca teslim edilecek bir şey olduğunda)
```

---

# Modül Özeti

Bir event, consume edilmeden üretilebilen, süresiz olarak persist edilip replay edilebilen, immutable bir tarihsel kayıttır. Event-driven architecture, Modül 9'daki point-to-point coupling sorununu, producer'lar ile consumer'lar arasına bir event intermediary koyarak çözer — böylece hiçbir taraf diğeri hakkında bir şey bilmek zorunda kalmaz, sadece bir event'in formatı üzerinde anlaşmaları yeterlidir.

Bu decoupling üç somut fayda getirir: immutable, authenticated bir event log'u üzerinden merkezi auditing ve access control; mevcut servislere dokunmadan yeni consumer ekleyebilme yeteneği; ve resilience, çünkü asenkron işlem, bir servisin başarısızlığının otomatik olarak yayılmayacağı (cascade etmeyeceği) anlamına gelir ve teslim edilmemiş event'ler bir consumer geri geldiğinde replay edilebilir. Push-based teslimat, consumer'ların sürekli yeni iş olup olmadığını sormasının getirdiği network yükü ve gecikmeyi önlediği için polling'e tercih edilir.

---

# Önemli Noktalar

- Bir event immutable'dır, hiç consume edilmeyebilir, ve paralel olarak birden fazla kez persist edilip consume edilebilir.
- Bir event intermediary, producer'ları consumer'lardan ayırır — producer'lar event'lerini kimin consume ettiğini bilmez, consumer'lar da event'i kimin ürettiğini bilmez.
- Bir event intermediary, SOA tarzı bir Enterprise Service Bus (ESB) ile aynı şey değildir — özellikle o tür merkezi bir darboğaza dönüşmemek için var olur.
- Event'leri merkezileştirmek, auditing'i (immutable, sıralı log) ve access control'ü (event service'te zorunlu auth) basitleştirir.
- Mevcut servisleri değiştirmeden yeni consumer'lar eklenebilir — point-to-point "örümcek ağını" ortadan kaldıran şey budur.
- Asenkron, event-driven işlem, senkron request/response zincirlerinden daha resilient'tır: başarısız bir servis otomatik olarak cascade etmez, event'ler replay ya da redeliver edilebilir.
- Push-based messaging, consumer'ların sürekli yeni iş sorması yerine, network I/O ve gecikme yükünü önlediği için polling'e tercih edilir.
