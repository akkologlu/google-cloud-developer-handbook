# Modül 16 – Best Practices for Cloud Run Functions

> Bu, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 5. Modülü'dür, Modül 4'ü (Modül 15: Integrating with Cloud Databases) takip eder.

---

# Genel Bakış

Çalışan bir function'ı production'a hazır hale getiren dört pratik alan:

```text
Implementation pratikleri → Performans ve networking → Retry on failure → Configuration ve scaling
```

---

# Implementation Best Practice'leri

| Pratik | Neden |
| --- | --- |
| Idempotent function'lar yazın | Tekrarlanan çağrılarda aynı sonuç — kısmen başarısız olan bir invocation'ı retry etmeyi güvenli kılar |
| HTTP function'lar her zaman bir HTTP response döndürmeli | Aksi halde function timeout'a kadar çalışabilir, ve o süre boyunca faturalandırılırsınız |
| Invocation bittikten sonra arka plan aktivitesi olmamalı | Sonlandırma sonrası CPU erişilemez; aynı ortamdaki sonraki bir invocation, bu arka plan işinin devam etmesine ve müdahale etmesine neden olabilir |
| Function'ı sonlandırmadan önce async işlemlerin bittiğinden emin olun | Sızan işin sonraki invocation'lara karışmasını önler |
| Geçici dizine yazılan dosyaları açıkça silin | `/tmp`, in-memory bir dosya sistemidir — kalan dosyalar belleği tüketir ve sonunda bir cold start'a zorlayabilir |
| Önce local olarak geliştirin ve test edin | Deploy → bekle → log kontrol et döngüsü yavaştır; local test çok daha hızlıdır |
| Data locality ihtiyaçları için Cloud Run functions'ın açık kaynak abstraction layer'larıyla uyumlu bir platform kullanın | Standart ortamın karşılayamadığı coğrafi/ağ sınırı kısıtlamalarına uymanızı sağlar |
| `process.exit()` (Node.js) / `sys.exit()` (Python) kullanmayın | Beklenmedik davranışa neden olur — event-driven function'lardan örtük/açık olarak return edin, HTTP function'lardan HTTP response döndürün |
| Uncaught exception fırlatmayın | Yalnızca mevcut değil, **gelecekteki** invocation'lar için de cold start'a zorlar |
| Runtime hatalarını/exception'ları kodda her zaman handle edin | Handle edilmemiş exception'lar function'ı sonlandırabilir ve gelecekteki cold start'lara neden olabilir |

**Error Reporting ve logging:**

- Runtime exception'lar **Error Reporting**'e gönderilir — Google Cloud console'da toplayın, görüntüleyin, ve bildirim alın.
- Basit runtime logging varsayılan olarak dahildir; `stdout`/`stderr`'e yazılan her şey otomatik olarak console'da görünür.
- HTTP function'lar hatayı raporlamalı ve uygun bir HTTP status code ile yanıt vermeli; event-driven function'lar raporlamalı ve bir hata mesajı döndürmeli.

---

# Performans ve Networking

**Cold start:** bir function'ın execution environment'ını oluşturur ve initialize eder; function tarafından import edilen dependency'ler bu sırada yüklenir, invocation gecikmesine eklenir.

| Teknik | Etki |
| --- | --- |
| Kullanılmayan dependency'leri yüklemeyin | Cold start gecikmesini ve deploy süresini azaltır |
| Pahalı nesneleri (API client'lar, network bağlantıları) global scope'ta cache'leyin | Execution environment'lar sıklıkla recycled olur — global-scope değerleri, aynı instance'taki invocation'lar arasında kalır, yeniden hesaplamayı önler |
| Her code path'te kullanılmayan global değişkenleri lazy initialize edin | Global'leri initialize etmek her zaman cold-start gecikmesi ekler; lazy init, o path'e girilmediğinde bu maliyeti ödemeyi önler |
| Minimum bir instance sayısı belirleyin | Instance'ları warm ve hazır tutar, cold start'ları tamamen azaltır |
| Persistent HTTP bağlantıları oluşturun ve global scope'ta cache'leyin | Her invocation'da yeni bağlantı kurmak için harcanan CPU'yu azaltır, ve connection quota'yı tüketme riskini azaltır |
| Google API service client nesnelerini global scope'ta oluşturun | Invocation başına gereksiz bağlantıları ve DNS sorgularını önler |
| VPC-internal trafik için Serverless VPC Access connector'ları kullanın | Internal kaynaklara giden trafik internal DNS/IP kullanır ve internete asla açık olmaz |

---

# Retry on Failure

| Özellik | Detay |
| --- | --- |
| Kullanılabilirlik | **Yalnızca event-driven function'lar** — HTTP function'lar için kullanılamaz |
| Varsayılan | **Devre dışı** |
| Etkinleştirme | `gcloud functions deploy` ile `--retry` flag'i, ya da console'da "Retry on failure" |
| Devre dışı bırakma | `--retry` olmadan yeniden deploy edin, ya da console seçeneğini kaldırın |
| Retry penceresi | Varsayılan olarak **en fazla 7 gün**, başarıya ya da maksimum retry süresinin dolmasına kadar |

**Yaygın hata nedenleri:** bir bug'dan kaynaklanan handle edilmemiş bir exception, erişilemeyen/timeout olan bir service endpoint, kasıtlı olarak fırlatılan handle edilmemiş bir exception, ya da (Node.js) reddedilmiş bir promise / bir callback'e geçirilen null olmayan bir değer — function durur ve event discard edilir.

**Retry'ı güvenle kullanmak:**

- Kendi kendine çözülme olasılığı yüksek olan **aralıklı/geçici (transient)** hatalar için en uygundur (bağlantı hataları, timeout'lar).
- Başarısızlığa neden olan bug'ları, retry'ı etkinleştirmeden **önce test yoluyla** düzeltin — hatalı bir function, hiçbir zaman başarılı olmadan sürekli retry edilir.
- Retry'a yol açmaması gereken exception'ları handle edin.
- Hatalar kalıcı olduğunda sonsuz retry döngülerini önlemek için bir **end condition** ekleyin (örn. bir timestamp'ten daha eski event'leri discard etmek).

---

# IAM ve Configuration Best Practice'leri

| Alan | Pratik |
| --- | --- |
| Least privilege | Function erişimini minimum kullanıcı/service account sayısına ve minimum izne sınırlayın |
| Function'dan function'a erişim | Her function'ı, gerçekten ihtiyaç duyduğu belirli function alt kümesini çağırmakla sınırlayın (örn. bir login function bir user-profiles function'ı çağırabilir, ama muhtemelen bir search function'ı çağıramaz) |
| Runtime service account | Belirtilmediği sürece default service account kullanılır — production için, minimal bir izin kümesine sahip **dedicated bir user-managed service account** atayın |
| Memory/CPU | Farklı memory miktarları provision edin; allocated memory, allocated CPU'yu belirler; `--memory` ya da console ile ayarlayın |
| Timeout | Erken timeout'ları önlemek için gerçek execution süresinden biraz daha yüksek ayarlayın; `--timeout` ya da console ile ayarlayın |

**Concurrency:**

- Varsayılan olarak, bir instance **aynı anda bir isteği** işler.
- Concurrency (function'ın altta yatan Cloud Run servisi aracılığıyla), function başına bir concurrency değeriyle etkinleştirilebilir — tek bir instance'ın işleyebileceği maksimum eşzamanlı istek sayısı.
- Zaten warm olan bir instance'ın ekstra istekleri absorbe etmesine izin vererek cold start'ları azaltır.
- Function kodu **eşzamanlı çalıştırmaya güvenli (safe)** olmalıdır — Cloud Run functions, aynı instance'taki eşzamanlı istekler arasında **hiçbir izolasyon** sağlamaz.

---

# Scaling, Revisions ve Traffic Splitting

- **Scaling**, istek hacmine göre yeni instance'lar oluşturur; deployment sırasında ayarlanan **minimum ve maksimum instance sayıları** ile function başına kontrol edilir — her function bağımsız olarak ölçeklenir.
- **Minimum instance'lar** cold start'ları ve gecikmeyi azaltır; **maksimum instance'lar**, throughput açısından kısıtlı downstream kaynaklara (örn. veritabanları) giden yükü sınırlar.
- Instance limitleri **geçici olarak aşılabilir**: trafik spike'ları sırasında kısaca, ve bir deployment sonrasında (limitler revision başına olduğu için), böylece mevcut istekler eski instance'lar tarafından işlenmeye devam ederken yeni istekler yeni instance'lara gider.
- Her deployment, function'ın ve altta yatan Cloud Run servisinin yeni, **immutable bir revision**'ını oluşturur — bir function'ı değiştirmek için yeniden deploy edin.
- Varsayılan olarak, **tüm trafik en son revision'a yönlendirilir**; bir custom traffic configuration, trafiği revision'lar arasında bölebilir ya da önceki birine geri alabilir (rollback).

---

# Modül Özeti

Çalışan bir Cloud Run function'ını production'a hazır hale getirmek dört alanı kapsar: her zaman yanıt veren, temp dosyaları ve async işleri temizleyen, uncaught exception fırlatmayan ya da `process.exit()`/`sys.exit()` çağırmayan, idempotent ve düzgün sonlanan kod yazmak; cold start'ları minimize ederek (daha az dependency, pahalı nesnelerin global-scope'ta cache'lenmesi, nadiren kullanılan global'lerin lazy init'i, minimum instance'lar) ve network bağlantılarını yeniden kullanarak (persistent HTTP bağlantıları, Google API client'ları, Serverless VPC Access) performansı optimize etmek; retry'ı yalnızca event-driven function'lar için güvenle etkinleştirmek — etkinleştirmeden önce bug'ları düzeltmek, ve kalıcı hatalarda sonsuz retry döngülerini önlemek için bir end condition eklemek; ve IAM'i (least privilege, kısıtlı function'dan function'a erişim, dedicated runtime service account'lar), memory/CPU/timeout'u, concurrency'yi (Cloud Run functions'ın izolasyon sağlamadığının farkında olarak), ve kontrollü rollout'lar için scaling/revisions/traffic splitting'i yapılandırmak.

---

# Önemli Noktalar

- Idempotent function'lar retry'ları güvenli kılar; HTTP function'lar her zaman bir response döndürmelidir, aksi halde timeout'a kadar çalışma (ve faturalandırılma) riski vardır.
- Invocation sonlandıktan sonra hiçbir arka plan işi hayatta kalmamalıdır; önce async işlemlerin bittiğinden emin olun, ve bellek tükenmesini ve cold start'ları önlemek için temp dosyaları açıkça silin.
- `process.exit()`/`sys.exit()` çağırmayın; uncaught exception fırlatmayın — bunlar gelecekteki invocation'lar için de cold start'a zorlar. Hataları Error Reporting'e raporlayın; `stdout`/`stderr` logging kutudan çıktığı gibi çalışır.
- Cold start'lar dependency'leri yükler ve execution environment'ı initialize eder; kullanılmayan dependency'leri azaltarak, pahalı nesneleri global scope'ta cache'leyerek, nadiren kullanılan global'leri lazy initialize ederek, ve bir minimum instance sayısı belirleyerek azaltın.
- Persistent HTTP bağlantılarını ve Google API client'larını global scope aracılığıyla yeniden kullanın; yalnızca internal trafik için Serverless VPC Access kullanın.
- Otomatik retry yalnızca event-driven function'lar için mevcuttur, varsayılan olarak kapalıdır, ve en fazla 7 gün retry eder — transient hatalar için kullanın, etkinleştirmeden önce bug'ları düzeltin, ve sonsuz döngüleri önlemek için bir end condition ekleyin.
- Function erişimine ve function'dan function'a çağrılara least privilege uygulayın; production'da default yerine dedicated bir runtime service account kullanın.
- Memory tahsisi, CPU tahsisini belirler; timeout'u gerçek execution süresinin biraz üzerinde ayarlayın.
- Concurrency, cold start'ları azaltmak için tek bir instance'ın birden fazla isteğe hizmet vermesini sağlar, ama istekler arasında izolasyon olmadığı için eşzamanlı çalıştırmaya güvenli kod gerektirir.
- Minimum/maksimum instance sayıları scaling'i kontrol eder (ve geçici olarak aşılabilir); her deployment immutable bir revision oluşturur, ve trafik, bir custom configuration ile split edilmediği ya da rollback yapılmadığı sürece varsayılan olarak en son revision'a gider.
