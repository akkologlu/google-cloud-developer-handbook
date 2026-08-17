# Modül 17 — Introduction to Containers: Pratik Sorular

Bu set, container ve container image kavramlarını (arşiv-vs-runtime-instance ayrımı, bir container çalıştığında dosya sistemine ve network'e ne olduğu, dile göre gereksinimler, sistem bağımlılıkları, ve varsayılan root kullanıcısı dahil container configuration), Docker ile image build etmeyi ("sahnelenmiş image'ın içinde build etme" modeli, `FROM`/`COPY`/`RUN` talimatları, ve `CMD`/`ENTRYPOINT` ilişkisi), Buildpacks ile image build etmeyi (detect ve build fazları, builder'lar, ve `pack` CLI'ı), CI/CD araçlarını (Skaffold'un workflow'u ve `skaffold.yaml`, Cloud Build'in otomatik doğası ve `cloudbuild.yaml` step'leri, Cloud Build trigger türleri, ve Artifact Registry), ve best practice'leri (base image'ların neden şişirilmiş olduğu ve Distroless multi-stage build'lerin bunu nasıl çözdüğü, PID 1 ve signal handling, Docker'ın build cache'i, ve Container Analysis ile vulnerability scanning) kapsar. Bu modül, "Developing Containerized Applications on Google Cloud" kursunun Modül 1'idir — handbook'taki yeni bir kurs, 16 modülden oluşan "Developing Applications with Cloud Run Functions on Google Cloud" kursunun ardından geliyor.

Sorular, insanları gerçekten tuzağa düşüren ayrımlara ağırlık verir: bir container'ın neden yalnızca process'leri çalışırken var olduğu, açıkça değiştirilmedikçe varsayılan container kullanıcısının neden root olduğu, `RUN`'ın neden yalnızca sahnelenmiş image'da zaten mevcut olan programları çalıştırabildiği, bir builder'ın neden hem bir detect hem bir build fazına ihtiyaç duyduğu, Cloud Build'in otomasyonunun neden tetiklendikten sonra doğrudan bir girdi gerektirmediği, şişirilmiş bir base image'ın neden sadece bir boyut rahatsızlığı değil bir güvenlik riski olduğu, neden yalnızca PID 1'in sonlandırma sinyalleri alabildiği, ve Docker'ın build cache'inin neden önceki tek bir adım değiştiğinde bozulduğu.

Önce tüm soruları yanıtlamayı deneyin, ardından cevaplarınızı aşağıdaki [Cevap Anahtarı ve Açıklamalar](#cevap-anahtarı-ve-açıklamalar) bölümüyle kontrol edin.

---

## Sorular

**1.** Bir ekibin Artifact Registry'de hiç çalıştırılmamış bir container image'ı var. Ayrı olarak, bunu bir kere deploy ettiler ve deployment daha sonra sıfır çalışan instance'a kadar scale down oldu. Bu durumların hangisinde gerçekten bir "container" (bir container image'ın aksine) var olur?

A. Her iki durumda da bir container vardır, çünkü bir container image ile bir container, farklı isimler altında aynı şeydir.
B. Hiçbir durumda container yoktur — bir container image, dosyalardan oluşan bir şablon/arşivdir, bir container ise o image'ın, çalışan process'leri temsil eden bir runtime örneğidir; çalışan process yoksa, container da yoktur.
C. Container yalnızca ilk durumda vardır (registry'de kullanılmadan durduğunda), çünkü container'lar bir image push edildiği anda oluşturulur, çalıştırıldığında değil.
D. Container yalnızca ikinci durumda vardır (sıfıra scale olduktan sonra), çünkü sıfıra scale olmak image'ı kalıcı bir container nesnesine dönüştürür.

**2.** Bir container, bir container image'dan gerçekten çalışmaya başladığında, dosya sistemine ve network trafiği alma yeteneğine ne olur?

A. Container, host makinenin dosya sistemini hiçbir izolasyon olmadan doğrudan paylaşır, ve network erişimi önce elle bir public IP atanmasını gerektirir.
B. Hiçbir şey değişmez — bir container'ın bağımsız bir dosya sistemi ya da network interface'i yoktur; host'unkini doğrudan kullanır.
C. Çalışan container'ın dosya sisteminden yeni bir container image üretilir, ve network erişimi başlangıçtan sonra yapılandırılana kadar devre dışı kalır.
D. Container image'ın içeriği, container için private bir dosya sistemi seed eder, ve container'ın process'leri, uygulamanın bağlanıp (bind) gelen trafik için bir portu dinleyebilmesi için local IP'li bir sanal network interface'e erişim kazanır.

**3.** Bir Dockerfile hiçbir zaman bir `USER` talimatı ayarlamıyor. Sonuçtaki container, varsayılan olarak gerçekten hangi kullanıcı olarak çalışır, ve modül bunun neden dikkat edilmesi gereken bir şey olarak işaret ediyor?

A. Varsayılan olarak root kullanıcısı (sistem yöneticisi) kullanılır — modül bunun güvenlik açısından bir best practice olmadığını söylüyor.
B. Her container'a varsayılan olarak, otomatik olarak üretilen, ayrıcalıksız bir kullanıcı ID'si atanır.
C. Dockerfile'a açıkça bir `USER` talimatı eklenene kadar container tamamen başlamayı reddeder.
D. Kullanıcı, o anda Docker'ı çalıştıran host makineye giriş yapmış olan hesaptan miras alınır.

**4.** Bir ekip, bir Java uygulaması ile bir Go uygulaması için container image'a ne girmesi gerektiğini karşılaştırıyor. Modül bu ikisini birbirinden neyin ayırdığını söylüyor?

A. Her iki dil de runtime'da tam olarak aynı dosya kümesine ihtiyaç duyar — kaynak kod, bir runtime, ve ayrı olarak kurulan kütüphane bağımlılıkları.
B. Go uygulamaları, Java'nın bir JVM'e ihtiyaç duymasıyla tamamen aynı şekilde, image'a her zaman ayrı bir runtime kurulmasını gerektirir.
C. Java kaynak kodu önce compile edilmelidir, ve runtime'da yalnızca compiled binary'ler artı Java Virtual Machine gerekir — kaynak kodun kendisi gerekmez; buna karşın bir Go uygulaması, bağımlılıklarını ve kaynak kodunu tek bir binary'e birlikte compile eder — bu binary'ye statik varlıklar bile embed edilebilir.
D. Java uygulamaları, container image içinde hiçbir zaman bir runtime'a ihtiyaç duymaz, sadece compiled class dosyalarına ihtiyaç duyar.

**5.** Bir uygulama, HTML'i PDF'e dönüştürmek için headless bir browser'a, ve ayrı olarak görselleri işlemek için ImageMagick'e ihtiyaç duyuyor. Modül, bunun gibi ihtiyaçları nasıl kategorize ediyor?

A. Bunlar, tıpkı bir npm paketi ya da bir Python pip paketi gibi, sıradan uygulama kütüphanesi bağımlılıkları olarak deklare edilmelidir.
B. Container'lar bunun gibi ihtiyaçları hiç desteklemez — bunlar container'ın tamamen dışında, ayrı bir sanal makine gerektirir.
C. Bunlar, belirli bir uygulamanın gerçekten ihtiyaç duyup duymadığından bağımsız olarak, varsayılan olarak her base image'a otomatik olarak dahil edilir.
D. Bunlar, bir uygulama kütüphanesi bağımlılığı olarak ifade edilemeyen ihtiyaçlar olan sistem bağımlılıklarıdır — ve modül tam olarak bu türden örnekler listeler: headless bir browser, curl/tar/zip gibi araçlar, ek sistem fontları, ImageMagick, ve OpenOffice.

**6.** Modül, Docker ile uygulamanı container image'ın "içinde" build ettiğini söylerken tam olarak neyi kastediyor, ve `FROM` talimatı bu süreçte ne rol oynuyor?

A. Bir container image'ı bir sahneye koyarsın, ve sonraki her Dockerfile talimatı o sahnelenmiş image'ı değiştirir; `FROM` bu süreci, bir registry'den — yazılım build etmek ve çalıştırmak için gereken araçlarla önceden kurulmuş — bir base image indirip bu sahneye koyarak başlatır.
B. Uygulama, herhangi bir image'ın dışında, host makinede tamamen build edilir, ve bitmiş binary yalnızca en sonda, `FROM` kullanılarak aksi halde boş olan bir image'a kopyalanır.
C. `FROM`, build tamamlandığında tamamlanmış container image'ını bir registry'e yükler — başlangıçta hiçbir şey indirmez.
D. Her Dockerfile talimatı, kendinden önceki talimatlarla hiçbir bağlantısı olmayan, tamamen ayrı, ilişkisiz bir image üretir.

**7.** `COPY` talimatı bir Dockerfile'da tam olarak ne yapar, ve Docker, çekebileceği dosya kümesi için hangi terimi kullanır?

A. `COPY`, sahnelenmiş image'ın, sadece yedekleme amacıyla, ikinci, birebir aynı bir image olarak tam bir kopyasını oluşturur.
B. `COPY`, `FROM` talimatıyla işlevsel olarak aynı şekilde, bir registry'den yeni bir base image indirir.
C. `COPY`, dosyaları kaynak kod dizininden sahnelenmiş image'a çeker — Docker bu dizine "build context" der.
D. `COPY`, sahnelenmiş image'da zaten mevcut olan bir programı, image'ın dışından hiçbir şey çekmeden, içindeki dosyaları değiştirmek için çalıştırır.

**8.** Modül, `RUN` talimatının hangi programları çalıştırabildiği, ve bu programların hangi dosyalara erişebildiği hakkında ne söylüyor?

A. `RUN`, container image içinde olsun ya da olmasın, host makinenin dosya sisteminde herhangi bir yerde bulunan herhangi bir programı çalıştırabilir.
B. `RUN`, image'daki bir programı, dosyaları güncellemek için image üzerinde çalıştırmanı sağlar — ama çalıştırdığın program dosyası container image'da zaten mevcut olmalıdır, ve o programın erişebileceği tek dosyalar container image'da zaten var olan dosyalardır.
C. `RUN` yalnızca environment variable'lar tanımlamak için vardır ve hiçbir zaman gerçekten bir program çalıştırmaz.
D. `RUN`, image'da bunu yapacak bir araç mevcut olmasa bile, internetten herhangi bir dosyayı indirmek için otomatik olarak sınırsız network erişimine sahiptir.

**9.** Modülün tanımladığı şekliyle `CMD` ile `ENTRYPOINT` talimatları arasındaki ilişki nedir?

A. `ENTRYPOINT`, container'ı bir executable olarak başlatıp çalıştırmak için program dosyasına işaret eder, `CMD` ise çalışan container için, başladığında çalıştırılacak komut dahil, varsayılanlar sağlar; başka türlü bir executable komut belirtilmemişse, `ENTRYPOINT` zorunludur.
B. `CMD` ve `ENTRYPOINT`, tamamen aynı talimatın iki farklı ismidir, ve her iki talimatı da kullanan bir Dockerfile her zaman build edilemez.
C. `ENTRYPOINT`, environment variable'ları ayarlamak için kullanılır, `CMD` ise container'ın working directory'sini ayarlamak için kullanılır.
D. `CMD` yalnızca Buildpacks ile build edilen image'lara uygulanır, ve `ENTRYPOINT` yalnızca Docker ile build edilen image'lara uygulanır.

**10.** Bir builder bir kaynak kod dizinini işlediğinde, detect fazı ve build fazı gerçekte her biri ne yapar?

A. Detect fazı kaynak kodu doğrudan compile eder, ve build fazı yalnızca sonuçtaki binary'nin sözdizimsel olarak geçerli olduğunu doğrular.
B. Her iki faz da tam olarak aynı işi yapar — süreç, ilk geçiş herhangi bir nedenle başarısız olursa sadece yedeklilik amacıyla iki kez çalışır.
C. Detect fazı, belirli bir buildpack'in uygulanıp uygulanmadığını belirlemek için kaynak kodun üzerinde çalışır (örneğin bir Python buildpack'in bir `requirements.txt` dosyası kontrol etmesi) — detection başarısız olursa, o buildpack için build fazı skip edilir; build fazı ardından build/run-time ortamını kurar, bağımlılıkları indirir, gerekirse kaynak kodu compile eder, ve uygun bir entry point ayarlar.
D. Detect fazı, proje başına değil, bir builder'ın tüm ömrü boyunca yalnızca bir kez çalışır, ve sonucu, daha sonra hangi kaynak kod sağlanırsa sağlansın sonsuza kadar cache'lenir.

**11.** `pack` komut satırı aracı nedir, ve bir kaynak dizinini bir container image'a dönüştürmek için neye ihtiyaç duyar?

A. `pack`, bir Docker Build çalışmadan önce Dockerfile sözdizimini doğrulayan bir Dockerfile linter'ıdır, ve buildpack'lerle hiçbir ilişkisi yoktur.
B. `pack`, Cloud Native Buildpacks projesi tarafından bakımı yapılan bir komut satırı aracıdır; bir kaynak dizinini bir container image'a dönüştürmek için bir builder'a (bir ya da daha fazla buildpack içeren) ihtiyaç duyar, ve Google Cloud's buildpacks, Paketo Buildpacks, ya da Heroku Buildpacks gibi birden fazla projeden builder'larla çalışabilir.
C. `pack`, buildpack'lerin sıradan Dockerfile build'lerinin sadece ince bir sarmalayıcısı olduğu belirtildiği için, bir builder'a ek olarak bir Dockerfile'a da ihtiyaç duyar.
D. `pack`, yalnızca Google Cloud's buildpacks ile çalışacak şekilde kısıtlanmıştır ve mimari olarak başka herhangi bir projeden bir builder kullanamaz.

**12.** Skaffold'un çok aşamalı workflow'u boyunca, ve bir `skaffold.yaml` dosyasında, `build` ve `deploy` bölümleri her biri neyi yapılandırır?

A. Skaffold'un yalnızca container image'ları build edebildiği, hiçbir deployment yeteneği olmadığı belirtiliyor — deployment her zaman tamamen ayrı, ilişkisiz bir araçla ele alınmalıdır.
B. `skaffold.yaml`'ın `build` bölümü, image'ın nereye deploy edileceğini tanımlar, `deploy` bölümü ise onu hangi Dockerfile ile build edeceğini tanımlar.
C. Skaffold, tamamen local, yalnızca Docker'lı bir workflow için bile, herhangi bir container image build edebilmeden önce zaten çalışan bir Kubernetes cluster'ına ihtiyaç duyar.
D. Skaffold, projedeki kaynak kod değişikliklerini tespit eder, artifact'leri (Dockerfile'lar ya da Buildpacks gibi araçlarla, local'de ya da Cloud Build üzerinden) build eder, test edip tag'ler, ardından manifestleri Kubernetes, Docker, ya da Cloud Run gibi bir hedefe render edip deploy eder; `skaffold.yaml`'ın `build` bölümü container image'ın nasıl build edileceğini tanımlar, `deploy` bölümü ise (örneğin Kustomize ile) nasıl deploy edileceğini tanımlar.

**13.** Modül, başladıktan sonra Cloud Build sürecinin kendisinin ne kadar manuel girdi gerektirdiği hakkında ne söylüyor, ve bir `cloudbuild.yaml` dosyasındaki her step ne içermelidir?

A. Cloud Build, her bireysel build step'inin, bir sonrakine geçmesine izin verilmeden önce console'da manuel olarak onaylanmasını gerektirir.
B. Cloud Build'in yalnızca Docker tabanlı build'leri desteklediği belirtildiği için, `cloudbuild.yaml`'daki her step, bir `name` alanı yerine bir `dockerfile` alanı içermelidir.
C. Bir container image'ı Cloud Build ile build etme süreci, tetiklendikten sonra tamamen otomatiktir ve senden doğrudan bir girdi gerektirmez; kullanılan tüm kaynaklar kendi user projende çalışır, ve build log'larına Cloud Logging üzerinden erişebilirsin; her step, ortak bir aracı (örneğin docker çalıştıran bir image) çalıştıran bir container image olan bir cloud builder belirten bir `name` alanı içermelidir.
D. Cloud Build her zaman kendi Google Cloud projenin tamamen dışında, log'larına hiç erişemeyeceğin bir altyapıda çalışır.

**14.** Bir ekip üç ayrı şeyin otomatikleştirilmesini istiyor: GitHub push'larında otomatik olarak tetiklenen build'ler, Artifact Registry'de image push/tag/delete event'lerine yanıt olarak tetiklenen build'ler, ve Cloud Build'in doğal olarak entegre olmadığı harici bir kaynak kod sisteminden tetiklenen build'ler. Modül her biri için hangi trigger türünü öneriyor?

A. Manual trigger'lar herhangi bir event türünde otomatik olarak tetiklenecek şekilde yapılandırılabildiği için, tek bir manual trigger'ın her üç durum için de yeterli olduğu belirtiliyor.
B. İlk durum için GitHub repository'sine bağlı bir repository trigger (push/tag/PR event'lerinde tetiklenen), ikinci durum için bir Pub/Sub trigger (çünkü Artifact Registry ve Cloud Storage event'leri Pub/Sub'a publish edebilir), ve üçüncü durum için, GitLab ya da Bitbucket.com gibi harici sistemleri Cloud Build'e bağlamak üzere özel bir URL'e gelen webhook event'lerini kimlik doğrulayıp işleyen bir webhook trigger.
C. Bu üç senaryodan hiçbiri Cloud Build trigger'larıyla otomatikleştirilemez — harici ya da Pub/Sub odaklı senaryolar için yalnızca tamamen manuel build'lerin mümkün olduğu belirtiliyor.
D. Pub/Sub ve webhook trigger'larının yalnızca Cloud Build'in kendi dahili logging'i için kullanıldığı belirtildiği için, bir repository trigger'ın her üç durum için de doğru seçim olduğu söyleniyor.

**15.** Basit bir `FROM node:16` tabanlı Dockerfile ile build edilen bir Node.js uygulamasının container image'ı, Node.js runtime'ı ve uygulamanın kendi bağımlılıkları birlikte 100 MB'ın altında olsa bile, yaklaşık 950 MB'a ulaşıyor. Modül, kalanından neyin sorumlu olduğunu ve çözüm olarak neyi önerdiğini söylüyor?

A. Ekstra boyutun, modülün her Node.js projesinde doğası gereği büyük olduğunu söylediği, uygulamanın kendi kaynak kodundan ve statik varlıklarından geldiği belirtiliyor.
B. Bunun, image'ın boyutunu ölçmek için kullanılan aracın bir görüntüleme hatası olduğu, gerçek runtime ayak izinin zaten minimal olduğu belirtiliyor.
C. Ekstra boyutun, hangi base image seçilirse seçilsin, Docker'ın kendisinin her image'a her zaman eklediği kaçınılmaz bir overhead olduğu belirtiliyor.
D. `node` base image'ı, çalıştırmak için değil yazılım build etmek için tasarlanmış ekstra sistem paketleri (Debian package manager `apt-get`, compiler'lar, ve version control araçları gibi) taşır — bu, production'da bir güvenlik riskidir; çözüm, `FROM` talimatını tekrarlayarak minimal bir Distroless image'ı sahneleyen, ardından `COPY --from=<önceki sahne>` ile yalnızca uygulamayı ve bağımlılıklarını kopyalayan bir multi-stage build'dir.

**16.** Modül, bir container'ın uygulama process'inin neden bir wrapper shell script yerine doğrudan `CMD` ya da `ENTRYPOINT` ile başlatılması gerektiğini söylüyor, ve uygulama kodunda ek olarak ne yapman gerektiğini söylüyor?

A. Docker, Kubernetes, ve Cloud Run gibi container platformları, sinyalleri (örneğin bir process'i sonlandırmak için) yalnızca container içindeki PID 1'e sahip process'e gönderebilir; process'i doğrudan `CMD`/`ENTRYPOINT` ile başlatmak, onun bu sinyalleri almasını ve düzgün bir şekilde kapanmasını (graceful shutdown) sağlar, ve signal handler'lar PID 1 için otomatik olarak kaydedilmediğinden, bunları uygulama kodunda kendin implement edip kaydetmelisin.
B. Bunun, sinyallerle hiçbir ilgisi olmayan, saf bir performans optimizasyonu olduğu belirtiliyor — wrapper shell script'lerinin her workload için ölçülebilir şekilde her zaman daha yavaş çalıştığı söyleniyor.
C. Bunun yalnızca HTTP uygulamalarıyla ilgili olduğu belirtiliyor; event-driven ya da arka planda işlem yapan container'ların, hangi process'in PID 1'e sahip olduğundan tamamen etkilenmediği söyleniyor.
D. PID 1'e sahip process için signal handler'ların container runtime tarafından otomatik olarak kaydedildiği belirtiliyor, bu yüzden hiçbir zaman uygulama kodu değişikliği gerekmiyor.

**17.** Docker, her Dockerfile talimatı için build cache'inde yeniden kullanılabilir katmanlar arar. Modül, bu cache'in gerçekten kullanılabilmesi için hangi koşulun sağlanması gerektiğini söylüyor, ve sonuç olarak bir Dockerfile'da neyi geç konumlandırmayı öneriyor?

A. Build cache'inin her talimat için tamamen bağımsız olduğu belirtiliyor, bu yüzden erken bir talimatı değiştirmek, sonraki bir talimatın cache'inin yeniden kullanılıp kullanılamayacağını hiçbir zaman etkilemez.
B. Build cache, bir talimat için ancak Dockerfile'da ondan önceki tüm talimatlar da cache'i kullandıysa kullanılabilir; kaynak kod genellikle her yeni build'de değişen şey olduğundan, modül onu image'a Dockerfile'da mümkün olduğunca geç eklemeyi önerir.
C. Build cache'inin yalnızca bir Dockerfile'daki ilk talimata (`FROM`) uygulandığı, ondan sonra gelen hiçbir talimatı etkilemediği belirtiliyor.
D. Modül, kaynak kodu Dockerfile'da mümkün olduğunca erken eklemeyi önerir, çünkü build cache'inin sık değişen talimatlar önce geldiğinde en iyi çalıştığı belirtiliyor.

**18.** Modül, container image'ları zafiyetler için taramak üzere hangi servisi tanımlıyor, ne zaman otomatik olarak tetikleniyor, ve bir image'ı manuel olarak taramanı ne sağlıyor?

A. Vulnerability scanning'in, Google Cloud entegrasyonu olmayan, tamamen ayrı, üçüncü taraf bir ürün gerektirdiği belirtiliyor.
B. Manuel taramanın imkansız olduğu belirtiliyor — yalnızca push'ta otomatik tarama desteklenir, hiçbir on-demand seçenek yoktur.
C. Zafiyet metadata'sının herhangi bir API üzerinden kalıcı olarak kullanılamaz olduğu, yalnızca bir insan tarafından Google Cloud console içinde görüntülenebildiği belirtiliyor.
D. Container Analysis, Artifact Registry'de saklanan image'ları zafiyetler için tarar ve sonuç metadata'yı, bir API üzerinden kullanılabilir hale getirerek saklar; Artifact Registry'e her yeni image push edildiğinde otomatik olarak tetiklenebilir, ve on-demand scanning API'si ek olarak, örneğin `gcloud artifacts docker images scan` komutuyla, bir registry'de ya da local'de saklanan image'ları manuel olarak taramanı sağlar.

**19.** Distroless ile bir multi-stage build kullanmanın ötesinde, modül bir container image'ı güvenli ve verimli tutmak için başka hangi best practice'leri öneriyor, ve neden?

A. Uygulamayı non-root bir kullanıcı olarak çalıştır (saldırganların bir package manager kullanarak root'a ait dosyaları değiştirmesini önlemek, ve `sudo`'yu devre dışı bırakmayı ya da read-only container'lar kullanmayı mümkün kılmak için), saldırı yüzeyini azaltmak için gereksiz araçları ve utility'leri kaldır, yükleme/indirme süresini azaltmak için mümkün olan en küçük image'ı build et, ve bir ekip genelinde ortak, standart base image'larda standartlaş — böylece her base image sadece bir kez indirilir.
B. Uygulamayı production'da her zaman root kullanıcısı olarak çalıştır, çünkü modül bunun uygulamanın kendi dosya sistemine tam erişime sahip olması için gerekli olduğunu belirtiyor.
C. Mevcut her geliştirme ve hata ayıklama aracını production image'ına dahil et, çünkü modül bunun, image boyutu pahasına, incident response'u hızlandırdığını belirtiyor.
D. Bir ekip genelinde ortak base image'larda standartlaşmaktan kaçın, çünkü modül her projenin izolasyon için kendi benzersiz, bağımsız olarak indirilen base image'ına ihtiyaç duyduğunu belirtiyor.

**20.** Artifact Registry nedir, ve modülün tanımına göre Cloud Build ile ilişkisi nasıldır?

A. Artifact Registry'nin, yalnızca bir geliştiricinin kendi makinesinde var olan, local-only bir cache olduğu, Cloud Build ile hiçbir ilişkisi olmadığı belirtiliyor.
B. Artifact Registry'nin yalnızca Docker container image'larını saklayabildiği, açıkça başka hiçbir yazılım paketi türünü saklayamadığı belirtiliyor.
C. Artifact Registry, container image'lar ve yazılım paketleri de dahil olmak üzere, yazılım artifact'lerini private repository'lerde saklamak ve yönetmek için kullanılan bir Google Cloud servisidir; Google Cloud için önerilen container registry'sidir, ve Cloud Build'in build'lerinden üretilen paketleri ve container image'ları saklamak için Cloud Build ile entegre olur.
D. Artifact Registry'nin, ayrı bir registry servisine olan ihtiyacı tamamen ortadan kaldıran, gerçek zamanlı bir vulnerability scanner olduğu belirtiliyor.

---

## Cevap Anahtarı ve Açıklamalar

**1. Doğru cevap: B.**
Bir container image, runtime'da bir container instance'ının nasıl gerçekleneceğini tanımlayan bir şablondur — pratikte, dosyalardan oluşan bir arşivdir; bir container ise, o image'ın, uygulamanın çalışan process'lerini temsil eden bir runtime örneğidir. Çalışan process yoksa, container da yoktur — bu yüzden ne registry'de kullanılmadan duran bir image'da, ne de sıfır çalışan instance'a scale olmuş bir deployment'ta, o anda gerçek bir container vardır.

**2. Doğru cevap: D.**
Bir container çalıştığında, bir programı çalıştırmanın ötesinde iki şey daha olur: container image'ın içeriği, container için private bir dosya sistemi seed etmek için kullanılır (uygulamanın process'lerinin görebileceği tüm dosyalar), ve container'daki process'ler, local IP'li bir sanal network interface'e erişim kazanır — bu da uygulamanın bu interface'e bağlanıp gelen trafik için bir portu dinlemesini sağlar.

**3. Doğru cevap: A.**
Container configuration, programı hangi kullanıcıyla çalıştıracağını içerir; ayarlanmazsa, varsayılan olarak root kullanıcısı (sistem yöneticisi) kullanılır — modül bunun güvenlik nedenleriyle bir best practice olmadığını açıkça belirtiyor.

**4. Doğru cevap: C.**
Java'da yazılmış bir uygulama önce compile edilmelidir — uygulamayı çalıştırmak için kaynak kod artık gerekmez, ama compiled binary'ler (artı Java Virtual Machine) gerekir. Buna karşın bir Go uygulamasında, bağımlılıklar ve kaynak kod birlikte tek bir binary'e compile edilir, ve statik varlıklar bu binary'ye doğrudan embed edilebilir.

**5. Doğru cevap: D.**
Bazı uygulamalar, bir uygulama kütüphanesi bağımlılığı olarak ifade edilemeyen sistem araçlarına bağımlıdır; modülün kendi örnekleri arasında headless bir browser (örn. HTML'i PDF'e dönüştürmek için), curl/tar/zip gibi araçlar, ek sistem fontları, ImageMagick (görselleri işlemek için), ve OpenOffice (belge formatlarını dönüştürmek için) yer alır.

**6. Doğru cevap: A.**
Docker ile, uygulamanı container image'ın içinde build edersin: bir container image'ı bir sahneye koyarsın, ve her Dockerfile talimatı o sahnelenmiş image'ı değiştirir. `FROM` talimatı bu süreci, yazılım build etmek ve çalıştırmak için gereken araçlarla önceden kurulmuş bir base image'ı bir registry'den indirip bu sahneye koyarak başlatır — sonraki talimatlar tarafından değiştirilmek üzere.

**7. Doğru cevap: C.**
`COPY` talimatı kaynak kodu çeker; Docker, kaynak kod dizinindeki dosya kümesine "build context" der, ve `COPY`, bu dosyaları `FROM` talimatıyla indirilen sahnelenmiş image'a getirmek için kullanılır.

**8. Doğru cevap: B.**
`RUN` talimatı, image'daki bir programı, image üzerinde çalıştırmanı sağlar — bu şu anlama gelir: çalıştırdığın program dosyası container image'da zaten mevcut olmalıdır, ve o programın erişebileceği tek dosyalar container image'da zaten var olan dosyalardır — örneğin bir sistem paketi kurmak, kütüphane bağımlılıklarını indirmek, ya da kaynak kodu binary'lere compile etmek için.

**9. Doğru cevap: A.**
`ENTRYPOINT`, container'ı bir executable olarak başlatıp çalıştırmak için program dosyasına işaret eder. `CMD`, çalışan bir container için, container başladığında çalıştırılacak komut dahil, varsayılanlar sağlar; executable komut `CMD` ile belirtilmemişse, `ENTRYPOINT` talimatı zorunludur.

**10. Doğru cevap: C.**
Bir builder bir kaynak dizinini işlemeye başlarsa, bir buildpack'in iki fazını çalıştırır. Detect fazı, kaynak kodun üzerinde çalışarak bir buildpack'in uygulanabilir olup olmadığını belirler (örn. bir Python buildpack'in bir `requirements.txt` ya da `setup.py` dosyası araması, ya da bir Node buildpack'in `package-lock.json` araması); detection başarısız olursa, o spesifik buildpack için build fazı skip edilir. Build fazı ardından, kaynak kodun üzerinde çalışarak build-time ve run-time ortamını kurar, bağımlılıkları indirir ve gerekirse kaynak kodu compile eder, ve uygun bir entry point ile başlangıç script'leri ayarlar.

**11. Doğru cevap: B.**
`pack`, Cloud Native Buildpacks projesi tarafından bakımı yapılan bir komut satırı aracıdır. Bir kaynak dizinini bir container image'a dönüştürmek için — bir ya da daha fazla buildpack içerebilen, bir OCI image olarak dağıtılan — bir builder'a ihtiyaç duyar, ve buildpacks standardını kullanan birden fazla projeden builder'larla çalışabilir; Google Cloud's buildpacks, Paketo Buildpacks, ve Heroku Buildpacks dahil.

**12. Doğru cevap: D.**
Skaffold çok aşamalı bir workflow kullanır: projedeki kaynak kod değişikliklerini tespit eder, tercih edilen araçla (Dockerfile'lar, Cloud Native Buildpacks, ve diğerleri; local'de ya da Cloud Build ile uzaktan build edilir) artifact'leri build eder, test edip tag'ler, ardından manifestleri Kubernetes, Docker, ya da Cloud Run gibi hedeflere render edip deploy eder. `skaffold.yaml`'da, `build` bölümü container image'ın nasıl build edileceğine dair configuration'ı tanımlar (örn. hangi Dockerfile'ın kullanılacağı), `deploy` bölümü ise container'ın nasıl deploy edileceğine dair configuration'ı tanımlar (örn. Kustomize kullanarak).

**13. Doğru cevap: C.**
Bir container image'ı Cloud Build ile build etme süreci, başladıktan sonra tamamen otomatiktir ve senden doğrudan bir girdi gerektirmez; build sürecinde kullanılan tüm kaynaklar kendi user projende çalışır, ve tüm build log'larına Cloud Logging üzerinden erişebilirsin. Bir `cloudbuild.yaml` dosyasındaki talimatlar bir dizi step olarak yazılır, ve her step, ortak araçları çalıştıran bir container image olan bir cloud builder belirten bir `name` alanı içermelidir — örneğin docker çalıştıran bir image.

**14. Doğru cevap: B.**
Bir Cloud Build trigger'ı, bağlı bir Google Cloud Source repository, GitHub, ya da Bitbucket repository'sindeki kaynak koda herhangi bir değişiklik yapıldığında otomatik olarak bir build başlatır — ilk senaryoyla eşleşir. Cloud Build Pub/Sub trigger'ları, Pub/Sub event'lerine yanıt olarak build'leri etkinleştirir — kullanım örnekleri arasında Artifact Registry ve Cloud Storage'daki image push, tag, ya da delete gibi event'ler vardır — ikinci senaryoyla eşleşir. Webhook trigger'ları, GitLab ya da Bitbucket.com gibi harici kaynak kod yönetim sistemlerinin Cloud Build'e bağlanmasını sağlamak için, özel bir URL'e gelen webhook event'lerini kimlik doğrulayıp işler — üçüncü senaryoyla eşleşir.

**15. Doğru cevap: D.**
Node.js runtime'ı kabaca 80 MB'dır ve uygulama artı kütüphane bağımlılıkları 1 MB'ın altındadır, ama `node` base image'ından doğrudan build edilen image yaklaşık 950 MB'a ulaşır — çünkü bu base image, çalıştırmak için değil yazılım build etmek için gereken sistem paketleriyle doludur: Debian package manager `apt-get`, GCC compiler ve make gibi build araçları, version control araçları, ve daha fazlası — bu, production'da bir güvenlik riskidir çünkü bu ekstra paketler zafiyetler içerebilir. Docker'ın sunduğu çözüm bir multi-stage build'dir: Distroless projesinden (yalnızca runtime, build-time değil, bağımlılıkları içeren) yeni bir image'ı sahnelemek için `FROM` talimatını tekrarlamak, ardından yalnızca uygulamayı ve bağımlılıklarını getirmek için `COPY --from=<önceki sahne>` kullanmak.

**16. Doğru cevap: A.**
Bir container'da başlatılan ilk process PID 1'i alır, ve Docker, Kubernetes, ve Cloud Run gibi container platformları, sinyalleri — en önemlisi process'leri sonlandırmak için — yalnızca container içindeki PID 1'e sahip process'e gönderebilir. Bu yüzden, container'ının uygulama process'ini Dockerfile'ındaki `CMD` ya da `ENTRYPOINT` talimatıyla başlatmalısın — bu, o process'in sinyalleri almasını ve sonlandırıldığında düzgün bir şekilde kapanmasını (graceful shutdown) sağlar. Ayrıca, signal handler'lar PID 1'e sahip process için otomatik olarak kaydedilmediğinden, bu handler'ları kendi uygulama kodunda implement edip kaydetmelisin.

**17. Doğru cevap: B.**
Docker, bir image için build cache'ini yalnızca önceki tüm build adımları da onu kullandıysa kullanabilir. Kaynak kodun her yeni sürümü için genellikle yeni bir Docker image build edildiğinden, modül kaynak kodunu image'a Dockerfile'da mümkün olduğunca geç eklemeyi önerir — ve daha genel olarak, sık değişiklik içeren build adımlarını dosyanın altına konumlandırmayı — böylece ilgisiz, değişmemiş önceki adımlar yine de cache'e isabet edebilir.

**18. Doğru cevap: D.**
Container Analysis, container'lar için vulnerability scanning ve metadata depolama sağlayan bir Google Cloud servisidir; tarama servisi, Artifact Registry'deki image'lar üzerinde vulnerability scan'leri yapar, ardından sonuç metadata'yı saklar ve bir API üzerinden kullanılabilir hale getirir. Etkinleştirildiğinde, bir image'ı otomatik olarak tarayabilir ve Artifact Registry'e yeni bir image push edildiğinde tetiklenir. On-demand scanning API'siyle — örneğin `gcloud artifacts docker images scan` komutu üzerinden — bu registry'lerde ya da local'de saklanan container image'ları manuel olarak da tarayabilirsin.

**19. Doğru cevap: A.**
Distroless ile multi-stage build'lerin ötesinde modül şunları önerir: container içinde uygulamayı root kullanıcısı olarak çalıştırmaktan kaçınmak (saldırganların bir package manager kullanarak root'a ait dosyaları değiştirmesini önlemek, ve `sudo`'yu devre dışı bırakmayı ya da container'ı read-only başlatmayı desteklemek için); uygulamanın saldırı yüzeyini azaltmak için gereksiz araçları ve utility'leri kaldırmak; yükleme ve indirme sürelerini azaltmak için mümkün olan en küçük image'ı build etmek; ve bir kuruluş genelinde ortak, standart base image'larla image'lar oluşturmak — böylece her base image yalnızca bir kez indirilmesi gerekir ve sonrasında yalnızca benzersiz katmanların build edilmesi gerekir.

**20. Doğru cevap: C.**
Artifact Registry, container image'lar ve yazılım paketleri de dahil olmak üzere, yazılım artifact'lerini private repository'lerde saklamak ve yönetmek için kullanılan bir Google Cloud servisidir; Google Cloud için önerilen container registry'sidir, ve Cloud Build'in build'lerinden üretilen paketleri ve container image'ları saklamak için Cloud Build ile entegre olur.
