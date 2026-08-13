# Modül 14 – Securing Cloud Run Functions

> Bu, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 3. Modülü'dür, Modül 2'yi (Modül 13: Calling and Connecting Cloud Run Functions) takip eder.

---

# Genel Bakış

Bu modül, Cloud Run functions'ı güvence altına almak için üç katmanı ele alır:

```text
Identity-based erişim kontrolü (kim çağırabilir) → Network-based erişim kontrolü (trafik nereden gelebilir/nereye gidebilir) → CMEK ile encryption (at rest verisini kim okuyabilir)
```

---

# Identity-Based Erişim Kontrolü

Function'lar **varsayılan olarak private'dır ve authentication gerektirir**; isteğe bağlı olarak bir function'ı public olarak deploy edebilirsiniz.

| Adım | Detay |
| --- | --- |
| 1. Authentication | Identity credential'ı doğrula — requestor'ın gerçekten iddia ettiği kişi/servis olduğunu teyit et |
| 2. Authorization | Kimliği doğrulanmış identity'nin izinlerini değerlendir |

**İki kimlik türü:** service account (kişi olmayan kimlikler — bir function, application ya da VM) ve user account (bireysel Google Account sahipleri ya da Google Groups).

**Token tabanlı authentication:** istemciler bir service/user account credential'ından bir token oluşturur; token sınırlı bir ömre sahiptir, istekle birlikte gönderilir, ve altta yatan credential sızarsa oluşacak zararı sınırlar.

| Token türü | Ne için kullanılır |
| --- | --- |
| OAuth 2.0 access token | API çağrılarını authenticate etmek için |
| ID token | Geliştirici tarafından yazılmış koda yapılan çağrıları (örn. bir function'ın başka bir function'ı çağırması) authenticate etmek için |

İkisi de OAuth 2.0 framework'ü ve OpenID Connect (OIDC) kullanılarak oluşturulur.

**IAM rolleri (Cloud Run functions predefined rolleri):**

| Rol | Ne sağlar |
| --- | --- |
| Cloud Functions Admin | Tam administrative yönetim |
| Cloud Functions Developer | Function'ları geliştirme ve deploy etme |
| Cloud Functions Invoker | Yalnızca function'ı invoke etme |
| Cloud Functions Viewer | Salt-okunur görünürlük |

Principal'ları (user ya da service account email'leri) console ya da gcloud CLI ile bu rollerle authorize edersiniz.

> **Sınav tuzağı:** Bir function, varsayılan olarak private'dır ve authentication gerektirir — public erişim, başlangıç durumu değil, isteyerek seçtiğiniz bir şeydir.

---

# Function'ı Kim Çağırabilir

| Function türü | Kim invoke edebilir |
| --- | --- |
| Event-driven function | Yalnızca abone olduğu event kaynağı |
| HTTP function | Farklı identity türleri (function'ı test eden bir developer, function'ı kullanan bir servis), her biri `Authorization` header'ında uygun izinlere sahip bir ID token sağlayarak |

**Developer test akışı:** uygun izinleri veren bir role sahip bir user account edin → o hesaptan bir ID token oluştur → token'ı isteğin `Authorization` header'ında geçir.

---

# Runtime Service Account

Her function, başka Google Cloud kaynaklarına erişirken kullandığı kimlik olan bir **runtime service account** ile ilişkilendirilir.

| Ayar | Detay |
| --- | --- |
| Varsayılan | Compute Engine default service account (ya da 1st gen için App Engine default service account) — yalnızca test/geliştirme için |
| Production | Yalnızca gereken minimum izinlerin verildiği, dedicated bir runtime service account belirtin |

---

# Function'dan Function'a Çağrılar

Bir function'ın başka bir function'ı çağırmasını sağlamak ve hangi function'ın hangisini çağırabileceğini kısıtlamak için:

| Nesil | Alıcı function üzerinde verilecek rol |
| --- | --- |
| Cloud Run functions | `roles/run.invoker`, çağıran function'ın service account'una verilir |
| Cloud Run functions (1st gen) | `roles/cloudfunctions.invoker`, çağıran function'ın service account'una verilir |

Çağıran function ayrıca, `audience` (`aud`) alanı alıcı function'ın URL'ine ayarlanmış, **Google-signed bir ID token** sağlamalı ve bunu `Authorization` header'ında göndermelidir.

> **Sınav tuzağı:** Invoker rolü, **alıcı function üzerinde**, **çağıran function'ın kimliğine** verilir — tersi değil. Ve ID token'ın `aud` alanı, onu tek bir belirli hedefe bağlar; B function'ı için verilmiş bir token, C function'ına sunulursa reddedilir.

---

# Network-Based Erişim Kontrolü

**Ingress settings**, bir function'ın Google Cloud projenizin ya da VPC Service Controls perimeter'ınızın dışındaki kaynaklar tarafından invoke edilip edilemeyeceğini kısıtlar:

| Ingress seçeneği | Anlamı |
| --- | --- |
| Allow all traffic | Kısıtlama yok |
| Allow internal traffic only | Yalnızca aynı proje/perimeter içindeki Workflows ve VPC network'leri |
| Allow internal traffic and traffic from Cloud Load Balancing | Internal trafik artı Cloud Load Balancing |

**Egress settings**, outbound HTTP isteklerinin routing'ini kontrol eder — bunun için function'ı bir **Serverless VPC Access connector** aracılığıyla bir VPC network'e bağlamanız gerekir:

| Egress seçeneği | Anlamı |
| --- | --- |
| Route all outbound traffic through the connector | Her şey connector üzerinden geçer |
| Route only requests to private IPs through the connector | Yalnızca private IP'lere giden trafik connector üzerinden geçer |

**VPC Service Controls**, ek bir katman ekler: bir service perimeter oluşturun, proje(ler) ekleyin (Shared VPC için host + service projeleri), ve organization policy'lerle Cloud Functions API'sini kısıtlayın. Bu policy'ler yerinde olduğunda:

- HTTP functions yalnızca perimeter içindeki bir VPC network'ten gelen trafiği kabul eder.
- Tüm function'lar bir Serverless VPC Access connector kullanmak zorundadır.
- Tüm function'lar tüm egress trafiğini VPC network üzerinden yönlendirmek zorundadır.

> **Sınav tuzağı:** Ingress/egress ayarları, function başına isteğe bağlı seçimlerdir; VPC Service Controls organization policy'leri ise ikisinin de en katı versiyonunu perimeter genelinde **zorunlu** kılar — sadece bir seçenek olarak sunmaz.

---

# Cloud KMS ve CMEK ile Veriyi Korumak

**Cloud KMS**, Cloud Run functions'ı ve ilişkili verisini at rest korumak için encryption key — **customer-managed encryption keys (CMEK)** — oluşturmanıza ve yönetmenize izin verir. Anahtarlar size aittir, Google'a değil, ve software key, HSM destekli ya da harici olabilir.

**CMEK'in koruduğu veri:**

| Veri türü | Detay |
| --- | --- |
| Function source code | Deployment için upload edilen, Cloud Storage'da saklanan, build'de kullanılan kod |
| Build sonuçları | Build edilen container image, ve deploy edilen her function instance'ı |
| Internal event transport channel'lar | Bunların at-rest verisi |

Anahtar devre dışı bırakılır ya da yok edilirse, **hiç kimse** — sizi de dahil ederek — koruduğu veriye erişemez.

**Kurulum adımları:**

| Adım | Detay |
| --- | --- |
| 1. Bir anahtar oluştur | Single-region bir encryption key |
| 2. CMEK etkin bir Artifact Registry repository'si oluştur | Function ile **aynı anahtarı** kullanmalı |
| 3. Service account erişimi ver | Cloud Run Functions, Artifact Registry ve Cloud Storage service account'larının her birine, anahtarın principal'ı olarak eklenip **Cloud KMS CryptoKey Encrypter/Decrypter** rolü verilmeli |
| 4. Function'da CMEK'i etkinleştir | Deploy anında anahtarı ve repository'yi belirt |

**Kullanım senaryosu — Cloud Storage:** nesneler CMEK'i tek tek ya da bucket varsayılanı olarak kullanabilir; bir Cloud Storage değişikliğinde Eventarc üzerinden tetiklenen bir function decrypted nesneyi alabilir, ya da bir function nesneleri upload etmeden önce encrypt edebilir.

**Kısıtlamalar ve hata davranışı:**

- Cloud Run functions yalnızca anahtarın **primary versiyonunu** kullanır — belirli bir key versiyonu sabitleyemezsiniz.
- Anahtar yok edilir/devre dışı bırakılırsa, ya da gerekli izinler geri alınırsa: aktif instance'lar **kapatılmaz**, devam eden execution'lar **devam eder**, ama yeni execution'lar ve yeni instance gerektiren execution'lar **başarısız olur**.

> **Sınav tuzağı:** Bir CMEK anahtarını yok etmek/devre dışı bırakmak, çalışan function'ları anında durdurmaz — yalnızca *yeni* erişimi (yeni execution'lar, yeni instance'lar) engeller, hâlihazırda çalışan iş tamamlanır.

---

# Modül Özeti

Cloud Run functions, iki cephede güvence altına alınır: identity-based erişim kontrolü (OAuth 2.0 access token'lar ya da ID token'lar ile authentication, Cloud Functions Admin/Developer/Invoker/Viewer IAM rolleri ile authorization, least privilege'a göre kapsamlandırılmış function başına runtime service account'lar, ve function'dan function'a çağrılar için `roles/run.invoker` artı audience-scoped bir ID token) ve network-based erişim kontrolü (kimin içeri çağırabileceğini kısıtlayan ingress ayarları, bir Serverless VPC Access connector üzerinden outbound trafiği yönlendiren egress ayarları, ve ikisinin de en katı versiyonunu perimeter genelinde zorunlu kılan VPC Service Controls). Ayrıca, Cloud KMS customer-managed encryption keys (CMEK), function source code'unu, build sonuçlarını ve internal event transport verisini at rest korur — yalnızca müşteriye ait, yalnızca anahtarın primary versiyonunu kullanarak, yok etme/devre dışı bırakma yalnızca yeni erişimi engelleyip hâlihazırda çalışan işi durdurmadan.

---

# Önemli Noktalar

- Function'lar varsayılan olarak private'dır ve authentication gerektirir; public erişim isteğe bağlıdır.
- Authentication (kimlik doğrulama) her zaman authorization'dan (izin değerlendirmeden) önce gelir.
- OAuth 2.0 access token'lar API çağrılarını, ID token'lar ise geliştirici tarafından yazılmış koda yapılan çağrıları (function'dan function'a çağrılar dahil) authenticate eder.
- Cloud Functions Admin/Developer/Invoker/Viewer, predefined IAM rolleridir; Invoker yalnızca çağırma erişimi verir.
- Event-driven function'lar yalnızca abone oldukları event kaynağı tarafından invoke edilebilir; HTTP function'lar, herhangi bir çağıran identity'den `Authorization` header'ında bir ID token gerektirir.
- Varsayılan runtime service account (Compute Engine/App Engine default) yalnızca test içindir — production, minimum gerekli izinlere sahip dedicated bir service account gerektirir.
- Function'dan function'a çağrılar, alıcı function üzerinde çağıranın service account'una verilen `roles/run.invoker` (ya da 1st gen için `roles/cloudfunctions.invoker`) artı `aud`'u alıcı function'ın URL'ine ayarlanmış Google-signed bir ID token gerektirir.
- Ingress ayarları (all / internal only / internal + Cloud Load Balancing) ve egress ayarları (tüm trafik / yalnızca private IP'ler, bir Serverless VPC Access connector üzerinden) network seviyesinde erişimi kontrol eder; VPC Service Controls, ikisinin de en katı versiyonunu perimeter genelinde zorunlu kılabilir.
- CMEK, müşteriye ait bir anahtarla function source code'unu, build sonuçlarını ve internal event transport verisini at rest korur; function ve Artifact Registry repository'si aynı anahtarı paylaşmalıdır, ve Cloud Run Functions/Artifact Registry/Cloud Storage service account'larının her birine Cloud KMS CryptoKey Encrypter/Decrypter rolü gerekir.
- CMEK için yalnızca anahtarın primary versiyonu kullanılır; anahtarı yok etmek ya da devre dışı bırakmak, hâlihazırda devam eden işi durdurmadan yeni execution'ları ve yeni instance'ları engeller.
