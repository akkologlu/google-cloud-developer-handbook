# Modül 15 – Integrating with Cloud Databases

> Bu, "Developing Applications with Cloud Run Functions on Google Cloud" kursunun 4. Modülü'dür, Modül 3'ü (Modül 14: Securing Cloud Run Functions) takip eder.

---

# Genel Bakış

Cloud Run functions; Firestore, Cloud SQL, Spanner, Bigtable, BigQuery ve Memorystore ile entegre olabilir. Bu modül, iki bağlanma modeline ve destekleyici mekanizmalara derinlemesine giriyor:

```text
Memorystore (function aktif olarak dışarı bağlanır) ↔ Firestore (veritabanı değişiklikleri function'ı tetikler) + environment variables + Secret Manager + BigQuery Remote Functions
```

---

# Memorystore

**Memorystore**, **Redis** ve **Memcached** için fully managed, highly available, scalable ve güvenli bir in-memory cache servisidir. Provisioning, replication, failover ve patching'i otomatikleştirir; IAM (güvenli erişim) ve Cloud Monitoring (gözlemlenebilirlik) ile entegredir.

| Teknoloji | Nedir |
| --- | --- |
| Redis | Veritabanı, cache, message broker ve streaming engine olarak kullanılan açık kaynaklı in-memory veri yapısı deposu |
| Memcached | Açık kaynaklı, dağıtık memory object caching sistemi |

**Bir function'ı bir Redis instance'ına bağlamak** (Serverless VPC Access aracılığıyla):

1. Redis instance'ının authorized VPC network'ünü belirle.
2. Function ile **aynı region'da** bir Serverless VPC Access connector oluştur.
3. Connector'ı Redis instance'ının authorized VPC network'üne bağla (oluştururken network, region, IP address range belirterek).
4. Connector'ın `Ready` durumda olduğunu doğrula.
5. Function'ı, connector'ın path/adını ve Redis host IP/port'u için environment variable'ları belirterek deploy et.
6. Function kodu, bu environment variable'ları kullanarak bir Redis client oluşturur; bir HTTP `GET` isteğiyle invoke edilir.

> **Sınav tuzağı:** Connector'ın region'ı, Redis instance'ının region'ı değil, **function'ın** region'ıyla eşleşmelidir — ardından ayrıca Redis instance'ının authorized network'üne bağlanır (attach edilir).

---

# Environment Variables

Bir Cloud Run function için **deployment zamanında** ayarlanan key-value çiftleri.

| Özellik | Detay |
| --- | --- |
| Erişim | Runtime'da function kodu tarafından okunur, ya da buildpack yapılandırması olarak kullanılır |
| Saklama/kapsam | Cloud Run functions backend'inde saklanır; tek bir function'a bağlıdır; o function'ın lifecycle'ı içinde var olur |
| Nasıl ayarlanır | gcloud CLI, Google Cloud console, ya da source control'deki bir YAML dosyası (adı deploy zamanında sağlanır) |
| Okunması (Python) | `os` modülü |

---

# Firestore

**Firestore**, high availability, bakım penceresi gerektirmeyen ölçeklenebilirlik ve multi-region replication sağlayan, fully managed, serverless bir NoSQL document database'dir. Veri, **document**'lar (key-value çiftleri kümesi) olarak saklanır ve **collection**'lar içinde gruplanır.

**Firestore'u Cloud Run functions ile genişletmek:** Firestore'daki değişikliklerin tetiklediği event'leri işleyen function kodu yaz — bu event'ler **Cloud Run functions for Firebase SDK** aracılığıyla exposed edilir.

| Event türü | Ne zaman tetiklenir |
| --- | --- |
| Created | Document oluşturulduğunda |
| Updated | Document güncellendiğinde |
| Deleted | Document silindiğinde |
| (herhangi biri) | Yukarıdakilerden herhangi biri — genel bir "write" event'i |

> **Sınav tuzağı:** Cloud Run functions için Firestore trigger'ları **yalnızca Native mode'daki Firestore'da** çalışır — **Datastore mode'daki Firestore için kullanılamaz**.

**Gerekli trigger yapılandırması:** bir **document path** (belirli bir document ya da bir wildcard pattern, asla trailing slash ile değil) ve bir **event type**.

| Kavram | Detay |
| --- | --- |
| Snapshot | Event'in verisinin bir snapshot'ı function'a ulaşır; update event'lerinde, hem önceki hem sonraki snapshot mevcuttur |
| `DocumentReference` | `snapshot.ref` (Firestore Node.js SDK'sından) — function'ı tetikleyen document'i değiştirmeni sağlar |
| Firebase Admin SDK | Function'ı tetikleyenin **dışındaki** document'leri okumak/yazmak için gereklidir |

---

# Secrets ve Secret Manager

Hassas credential'lar (DB kullanıcı adı/şifresi, API key'ler), kodun ya da config'in içine gömülmek yerine **Secret Manager**'da saklanır.

| Kavram | Detay |
| --- | --- |
| Secret | Metadata (replication location'lar, label'lar, permission'lar vb.) ve secret version'larını tutan bir nesne |
| Secret version | Gerçek secret verisini (bir text string ya da binary blob) saklar |
| Ön koşul | Secret oluşturmak/yönetmek için Secret Manager API'sini etkinleştir |

**Erişim vermek:** function'ın **runtime service account**'una, secret üzerinde `roles/secretmanager.secretAccessor` rolünü ver.

**Bir secret'ı function'a ulaştırmak:**

| Yöntem | Version davranışı |
| --- | --- |
| Volume olarak mount et | Function onu bir dosyadan okur; her okuma **en son** version'ı alır |
| Environment variable olarak geçir | **Instance startup time**'da çözümlenir — function, yeni bir instance başlayana kadar **belirli** bir version'a sabitlenir |

**Cross-project erişim:** yukarıdaki gibi runtime service account'a erişim ver, ardından secret'ı **project ID**'yi içeren bir resource path ile referansla.

> **Sınav tuzağı:** Function'ın rotate edilmiş bir secret'ı hemen görmesi mi gerekiyor? Environment variable değil, volume mount kullanın — env var'lar yalnızca bir kez, instance startup'ında çözümlenir.

**Kullanım senaryosu akışı (harici API key):** API key bir secret olarak saklanır → runtime service account'a Secret Manager Secret Accessor rolü verilir → deploy zamanında secret adı ve erişim yöntemi (mounted file path ya da environment variable) belirtilir → function kodu dosyayı/env var'ı okuyup değeri kullanarak harici API'yi çağırır.

---

# BigQuery Remote Functions

Bir **BigQuery remote function**, Google Standard SQL sorgularının bir **Cloud Run function**'ı çağırmasına izin verir — SQL'i keyfi kodla entegre eder.

**Kurulum:**

1. BigQuery Connection API'sini etkinleştir.
2. Gerekli IAM rolüne sahip ol (örn. `roles/bigquery.admin`).
3. `CLOUD_RESOURCE` türünde bir connection oluştur (console, `bq` CLI, ya da connection API):
   ```bash
   bq mk --connection --display_name='friendly name' \
     --connection_type=CLOUD_RESOURCE \
     --project_id=my_project_id --location=US my-connection
   ```
4. Connection'ın service account'una, **function üzerinde** doğru Invoker rolünü ver:

| Nesil | Verilecek rol |
| --- | --- |
| 1st gen | Cloud Functions Invoker |
| 2nd gen | Cloud Run Invoker |

5. Dataset ve connection üzerinde gerekli rollere sahip ol (örn. `roles/bigquery.admin`), ardından remote function'ı oluştur:
   ```sql
   CREATE FUNCTION my_project_id.my_dataset.function_name(x INT64, y INT64) RETURNS INT64
   REMOTE WITH CONNECTION `my_project_id.us.my-connection`
   OPTIONS (endpoint = 'https://us-east1-my_gcf_project.cloudfunctions.net/function_name')
   ```
6. Invoke etmek için: dataset üzerinde `roles/bigquery.dataViewer`'a ve connection üzerinde `roles/bigquery.connectionUser`'a sahip ol, ardından bir query içinde çağır:
   ```sql
   SELECT val, my_project_id.my_dataset.function_name(val, 2)
   FROM UNNEST([NULL, 2, 3, 5, 8]) AS val;
   ```

> **Sınav tuzağı:** Connection'ın service account'una verilen Invoker rolü, function'ın nesline bağlıdır — 1st gen için Cloud Functions Invoker, 2nd gen için Cloud Run Invoker — bu, function'dan function'a çağrılarda kullanılan aynı iki rolü yansıtır.

---

# Modül Özeti

Cloud Run functions, veriyle iki şekilde entegre olur: Memorystore'un Redis/Memcached cache'ine, function'ın region'ıyla eşleşen ve instance'ın authorized network'üne attach edilmiş bir Serverless VPC Access connector üzerinden aktif olarak dışarı bağlanarak (bağlantı bilgileri environment variable olarak geçirilir), ve Firestore document değişikliklerine trigger'lar aracılığıyla tepki vererek (create/update/delete/write, yalnızca Native mode, document path ile kapsamlandırılmış, önceki/sonraki snapshot'lar ve tetikleyen document için bir `DocumentReference` sağlar, başka herhangi bir document için Firebase Admin SDK gerekir). Environment variables, tek bir function'ın lifecycle'ına bağlı deployment-zamanı yapılandırmasını taşırken, Secret Manager hassas credential'ları korur — function'a, runtime service account'unun `secretAccessor` rolü aracılığıyla verilir, bir volume mount (her zaman en son version) ya da environment variable (instance startup'ında sabitlenmiş) olarak ulaştırılır. Ayrıca, BigQuery Remote Functions, SQL sorgularının bir `CLOUD_RESOURCE` connection aracılığıyla bir Cloud Run function'ı invoke etmesine izin verir; connection'ın service account'una (nesle bağlı) bir Invoker rolü verilir.

---

# Önemli Noktalar

- Memorystore, fully managed bir Redis/Memcached cache'idir; bir function'ı ona bağlamak, function'ın region'ında oluşturulan ve Redis instance'ının authorized VPC network'üne bağlanan bir Serverless VPC Access connector gerektirir.
- Environment variables, tek bir function'a ve onun lifecycle'ına bağlı, deployment-zamanı key-value çiftleridir; gcloud, console ya da bir YAML dosyasıyla ayarlanır; Python bunları `os` modülüyle okur.
- Firestore trigger'ları document create/update/delete/write event'lerinde tetiklenir, bir document path (trailing slash olmadan) ve event type gerektirir, ve yalnızca Native mode'daki Firestore için çalışır — Datastore mode için değil.
- Tetiklenen bir function, bir veri snapshot'ı alır (update'lerde önce ve sonra); `snapshot.ref`, tetikleyen document için bir `DocumentReference` verir, başka document'lere dokunmak için Firebase Admin SDK gerekir.
- Secret Manager, hassas credential'ları secret olarak (metadata + version'lar) saklar; bir secret version, gerçek text/binary veriyi tutar.
- Bir function'ın bir secret'a erişimi, runtime service account'una `roles/secretmanager.secretAccessor` verilmesini gerektirir; volume mount edilen secret'lar her zaman en son version'ı okur, environment variable secret'ları ise instance startup'ında geçerli olan version'a sabitlenir.
- Cross-project secret erişimi, aynı IAM izni artı hedef project ID'yi içeren bir resource path gerektirir.
- BigQuery Remote Functions, bir SQL sorgusunun bir `CLOUD_RESOURCE` connection aracılığıyla bir Cloud Run function'ı çağırmasına izin verir; connection'ın service account'unun function üzerinde Cloud Functions Invoker'a (1st gen) ya da Cloud Run Invoker'a (2nd gen) ihtiyacı vardır, ve caller'ların dataset üzerinde `bigquery.dataViewer`'a artı connection üzerinde `bigquery.connectionUser`'a ihtiyacı vardır.
