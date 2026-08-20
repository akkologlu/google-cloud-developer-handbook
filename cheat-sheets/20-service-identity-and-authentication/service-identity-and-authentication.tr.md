# Modül 20 – Service Identity and Authentication

> Bu, `deep-dive/19-fundamentals-of-cloud-run/` ile aynı kursun (Cloud Run odaklı kurs) ikinci modülüdür. Modül 19'un kısa IAM girişinin üzerine, service account'lara, resource hierarchy'ye, least privilege'a, ve secret'lar/environment variable'lara derinlemesine bakar.

---

# Genel Bakış

```text
Service account & identity → Resource hierarchy → Principle of least privilege → Secrets & environment variables
```

---

# Google Cloud API'leri ve Yetkilendirme

Google Cloud, bir API koleksiyonudur; bunları Cloud Run kodundan client kütüphaneleriyle (Go, Java, Node.js, Python, Ruby, PHP, C#, C++ için mevcuttur) çağırırsın. Örnek: Pub/Sub'a `pubsub_v1.PublisherClient()` ile publish etmek, `pubsub.googleapis.com`'a yapılan çağrıyı handle eder.

**Her API çağrısı IAM tarafından yetkilendirilir:** IAM, istekteki credential'lardan çağıranı tanımlar, ardından hedef kaynağa attach edilmiş IAM policy'deki policy binding'leri kontrol ederek identity'nin gereken role'e sahip olup olmadığını görür.

**Policy binding** = bir ya da daha fazla member'ın (identity) tek bir role'e bağlanması. Bir role, izinleri bir araya toplar (örn. Pub/Sub Publisher → `pubsub.topics.publish`). Bir member, birden fazla binding aracılığıyla birden fazla role'e sahip olabilir.

| Identity türü | Açıklama |
| --- | --- |
| Human | Google hesabın (bir grup/domain'e ait olabilir) |
| Machine — service account | Makineler/uygulamalar tarafından kullanılır: bir VM, bir Cloud Run servisi, bir Cloud Run function'ı, vb. |
| All users | Public/anonim erişim için özel bir tanımlayıcı |

---

# Service Account'lar

Bir **service account**, benzersiz bir e-posta adresiyle tanımlanan, makineler için özel bir identity'dir. User account'lardan farklı olarak: şifresi yoktur, browser/cookie ile giriş yapamaz; diğer kullanıcılar/service account'lar onun adına hareket edebilir; Workspace domain'inin üyesi değildir (gruplara katılabilse de).

Çalıştırdığın her kod (VM, Cloud Run servisi, Cloud Build build'i), client kütüphaneleri tarafından kimlik doğrulama için otomatik olarak kullanılan bir **built-in service account'a** erişim kazanır — ama bunu kendi **user-managed** service account'unla değiştirebilirsin (ve değiştirmelisin).

**Cloud Run'da service account'lar:** her servis/job, "service identity"si olan bir service account'a bağlıdır. Varsayılan olarak bu, **Editor role'üne sahip default Compute Engine service account'udur.** Best practice: servis başına, gereken minimal izinlere sahip bir user-managed service account.

**Runtime akışı (bir Google API'sini çağırmak, örn. Pub/Sub):** uygulama kodu → client kütüphanesi → servisin service account'uyla authenticate eder → bir **OAuth 2.0 access token** elde eder → API'yi çağırır → IAM token'ı doğrular ve hedef kaynak üzerindeki policy binding'i kontrol eder.

**Service-to-service iletişim:**

| Desen | Mekanizma |
| --- | --- |
| Asenkron | Cloud Tasks, Pub/Sub, Cloud Scheduler, Eventarc |
| Senkron | Diğer servisin endpoint URL'ine doğrudan HTTP çağrısı — best practice: her çağıran servis için ayrı bir service identity, alıcı servis üzerinde `roles/run.invoker` verilmiş: `gcloud run services add-iam-policy-binding RECEIVING_SERVICE --member='serviceAccount:CALLING_SERVICE_IDENTITY' --role='roles/run.invoker'` |

Senkron çağrılar için, istek, kimliğin kanıtı olarak **Google tarafından imzalanmış bir OpenID Connect (OIDC) ID token'ı** sunmalıdır (çağıran servisteki Google authentication client kütüphaneleriyle elde edilir; alıcı serviste aynı kütüphanelerle parse edilir/doğrulanır) — Google API çağrıları için kullanılan OAuth 2.0 access token'ından farklıdır.

---

# Resource Hierarchy

```text
Organization (kök) → Folder (opsiyonel) → Project (temel seviye varlık) → kaynaklar
```

- **Organization:** kök düğüm; altındaki her şey üzerinde merkezi görünürlük/kontrol.
- **Folder:** opsiyonel gruplama (departmanlar, ekipler, iş birimleri).
- **Project:** temel seviye varlık — kaynak oluşturmak, API'leri/servisleri kullanmak, izinleri yönetmek, billing'i etkinleştirmek için gereklidir.
- Her kaynağın tam olarak bir ebeveyni vardır (organization düğümü hariç).

**Hiyerarşideki her kaynağın kendi IAM policy'si vardır.** Bir policy binding, bir identity'ye o belirli kaynak üzerinde bir role verir — örn. bir topic üzerinde "Pub/Sub Publisher."

**Policy binding inheritance:** daha yüksek seviyeli bir kaynağa (örn. proje) eklenen bir binding, tüm daha düşük seviyeli kaynaklar tarafından (örn. içindeki her Pub/Sub topic'i) miras alınır — birçok kaynağa aynı anda erişim vermek için kullanışlıdır.

Bir kaynak üzerindeki **effective IAM policy** = kendi binding'leri + ebeveyninden ve atalarından miras alınan tüm binding'ler. **Daha yüksek bir seviyede verilen izinler, daha düşük bir seviyede geri alınamaz.**

---

# Least Privilege Prensibi

| Role türü | Kapsam | Örnekler |
| --- | --- | --- |
| **Basic** | Çok güçlü, tüm servisleri kapsar — production'da varsayılan olarak verilmesinden kaçın | Owner, Editor, Viewer |
| **Predefined** | Belirli bir servise granular erişim, Google tarafından yönetilir | Cloud Run Admin, Pub/Sub Publisher, Cloud Tasks Enqueuer |
| **Custom** | Kullanıcının belirttiği bir izin listesinden granular erişim | — |

**Default service account riski:** birini belirtmezsen, Cloud Run geniş **Editor** role'üne sahip default Compute Engine service account'unu kullanır — bu da, policy binding inheritance nedeniyle, projedeki çoğu kaynak üzerinde okuma/yazma verir. Kullanışlıdır, ama doğal bir güvenlik riskidir (kaynaklar bunun aracılığıyla oluşturulabilir/değiştirilebilir/silinebilir).

**Azaltma (3 adım):**
1. Cloud Run servisi için yeni bir service account oluştur.
2. Onu servisin identity'si olarak yapılandır (oluşturma/güncelleme anında, ya da yeni bir revision'da ayarlanabilir — console, gcloud CLI, YAML, ya da Terraform).
3. Bu identity için, servisin ihtiyaç duyduğu sadece kaynaklara, predefined/custom role'lerle policy binding ekle.

Yeni oluşturulan bir service account, **sıfır izinle** başlar — sen açıkça erişim vermeden, ondan yapılan herhangi bir Google Cloud API çağrısı IAM tarafından reddedilir:

```shell
gcloud pubsub topics add-iam-policy-binding my-topic \
  --member="my-service-account-email" --role="roles/pubsub.publisher"
```

---

# Environment Variable'lar ve Secret'lar

**Environment variable'lar:** container'a inject edilen, runtime'da uygulama koduna erişilebilir olan key-value çiftleri. Servis/job oluşturma, güncelleme, ya da yeni revision deploy anında ayarlanır (console, gcloud CLI, YAML, ya da Terraform); belirli, rezerve edilmiş isimler ayarlanamaz (container runtime contract'a bakın). Bir Dockerfile `ENV` varsayılanı, aynı isimli bir Cloud Run değişkeni tarafından override edilir.

```shell
gcloud run deploy my-service --image my-container-image-url --update-env-vars FOO=bar,BAZ=boo
gcloud run jobs create my-job --image my-container-image-url --update-env-vars FOO=bar,BAZ=boo
```

| Dil | Erişim |
| --- | --- |
| Python | `os.environ.get("key")` |
| Node.js | `process.env.key` |
| Java | `System.getenv("key")` |

**Secret'lar (Secret Manager):** hassas configuration için (API anahtarları, şifreler). Bir **secret** nesnesi, metadata (replikasyon konumları, etiketler, izinler) artı **secret version'lar** tutar, her biri gerçek veriyi (metin/binary) saklar.

| Erişim yöntemi | Davranış | En uygun kullanım |
| --- | --- | --- |
| Volume olarak mount etmek | Container'a bir dosya olarak sunulur; okumada her zaman Secret Manager'dan en güncel değeri getirir | `latest` kullanmak |
| Environment variable olarak geçirmek | Instance başlangıcında bir kez çözülür | Belirli bir version'a pinlemek |

```shell
# volume olarak mount edilmiş
gcloud run deploy my-service --image my-container-image --update-secrets=SECRET_FILE_PATH=my_secret:VERSION
# environment variable olarak
gcloud run deploy my-service --image my-container-image --update-secrets=ENV_VAR_NAME=my_secret:VERSION
```

Herhangi bir configuration değişikliği (bir secret'ı güncellemek dahil), sonraki revision'ların da miras alacağı yeni bir revision oluşturur. Erişim vermek için, service account'a **Secret Manager Secret Accessor** role'ünü ver:

```shell
gcloud secrets add-iam-policy-binding my-secret-id \
  --member="my-service-account-email" --role="roles/secretmanager.secretAccessor"
```

---

# Modül Özeti

Bir Cloud Run servisi, çalışan koddan fazlasıdır — aynı zamanda Google Cloud API'lerini ve diğer servisleri çağırmak için kullandığı bir identity'dir (service account). Bu modül, bu identity'nin nasıl kimlik doğruladığını (Google API'leri için access token, service-to-service çağrılar için OIDC ID token), izinlerin resource hierarchy boyunca (Organization → Folder → Project → kaynaklar, aşağı doğru geri alınamayan binding inheritance ile) nasıl yayıldığını, default service account'un neden bir güvenlik riski olduğunu ve bunun yerine least privilege'ın nasıl uygulanacağını, ve hassas olmayan (environment variable) ile hassas (Secret Manager) configuration'ın nasıl yönetileceğini kapsar.

---

# Anahtar Noktalar

- Bir policy binding, member + role'dür; bir member birden fazla binding aracılığıyla birden fazla role'e sahip olabilir.
- Service account'ların şifresi yoktur ve bir browser üzerinden giriş yapamazlar; benzersiz bir e-posta ile tanımlanırlar.
- Her Cloud Run servisinin/job'ının bir service identity'si (service account) vardır — default'u, minimal izinli, user-managed biriyle değiştir.
- OAuth 2.0 access token'ları Google Cloud API'lerine yapılan çağrıları yetkilendirir; OIDC ID token'ları doğrudan service-to-service HTTP çağrıları için kimliği kanıtlar.
- Kaynaklar, her birinin tam olarak bir ebeveyni olduğu sıkı bir hiyerarşi oluşturur (Organization → Folder → Project → kaynak).
- Policy binding'ler hiyerarşi boyunca miras alınır, ve daha yukarıda verilen izinler daha aşağıda geri alınamaz.
- Basic role'ler (Owner/Editor/Viewer) tehlikeli derecede geniştir — production'da predefined ya da custom role'leri tercih et.
- Default Compute Engine service account'u (Editor role'ü), Cloud Run'ın default identity'sidir ve inheritance nedeniyle proje genelinde geniş erişime sahiptir — bu gerçek bir risktir, sadece bir kolaylık takası değil.
- Yeni oluşturulan bir service account varsayılan olarak sıfır izne sahiptir; policy binding'lerle sadece ihtiyaç duyulanı ver.
- Bir Dockerfile'ın `ENV` varsayılanı, aynı isimli bir Cloud Run environment variable'ı tarafından override edilir.
- Secret'lar, düz environment variable'lar değil, versiyonlanmış Secret Manager nesneleridir — her zaman taze `latest` değerler için volume olarak mount et, ya da belirli bir version'a pinlenmiş bir env var olarak geçir.
- Bir secret'a erişmek, service account'a Secret Manager Secret Accessor role'ünün verilmesini gerektirir — IAM boyunca kullanılan aynı policy-binding deseni.
