# Otonom DevSecOps Master Pipeline

Bu repository, **"Shift-Left"**  prensiplerini merkeze alarak sıfırdan inşa edilmiş, %100 otonom çalışan uçtan uca bir DevSecOps CI/CD laboratuvar ortamıdır. 

Sistem, bilerek zafiyetli bırakılmış **OWASP Juice Shop** uygulamasını hedef alır. Kaynak kodun taranmasından uygulamanın canlı ortamda hacklenmesine kadar tüm güvenlik testleri insan müdahalesi olmadan gerçekleştirilir ve sonuçlar merkezi bir zafiyet panosunda (DefectDojo) toplanır.

---

## Kullanılan Teknolojiler ve Güvenlik Araçları

Sistemimiz, siber güvenlik endüstrisinde standart olarak kabul edilen açık kaynaklı araçların birbiriyle entegre edilmesinden oluşur:

* **Orkestrasyon:** Jenkins (Docker-in-Docker mimarisi ile)
* **Zafiyet Yönetim Merkezi:** OWASP DefectDojo
* **Secret Scanning:** Gitleaks
* **SAST (Statik Kod Analizi):** SonarQube
* **SCA:** Trivy
* **DAST (Dinamik Analiz):** OWASP ZAP & ProjectDiscovery Nuclei

---

## Kurulum ve Kullanım Rehberi

### Ön Koşullar
* Docker ve Docker Compose'un bilgisayarınızda kurulu olması.
* Sistemde minimum 8 GB (Tercihen 16 GB) boş RAM bulunması.

### 1. Ana Projeyi Klonlayın
Öncelikle bu repoyu bilgisayarınıza indirin:
git clone https://github.com/TA2ZTD/devsecops-lab.git
cd devsecops-lab

### 2. DefectDojo Kurulumu ve Başlatılması
Zafiyetleri görselleştireceğimiz merkezi panomuzu ayağa kaldırıyoruz:
git clone https://github.com/DefectDojo/django-DefectDojo
cd django-DefectDojo
./dc-build-local-images.sh
./dc-up-d.sh postgres-redis

*Not: Kurulum tamamlandığında docker-compose.yml dosyasından target portu 8081 olarak değiştirin (default port 8080 olarak geliyor ancak o portta jenkins çalışıyor.) ve terminalde beliren `admin` parolasını not alın. `http://localhost:8081` adresinden Dojo'ya giriş yapın. Sağ üstteki profil menüsünden "API v2 Key" sekmesine giderek anahtarınızı kopyalayın.*

### 3. Jenkins ve SonarQube'un Başlatılması
Ana dizine (docker-compose.yml dosyasının olduğu yere) geri dönün ve altyapıyı başlatın:
docker-compose up -d

### 4. Pipeline Konfigürasyonu
Klonladığınız klasördeki `Jenkinsfile` dosyasını bir metin editörüyle açın ve en üstteki `environment` bloğunda bulunan şu değişkenleri kendi Dojo sunucunuza göre düzenleyin:
* `DOJO_URL` (Varsayılan: http://host.docker.internal:8081)
* `DOJO_API_KEY` (Yukarıda kopyaladığınız anahtar)
* `PRODUCT_NAME` (DefectDojo'da görünmesini istediğiniz proje adı, örn: "OWASP Juice Shop")

### 5. DevSecOps Hattının Başlatılması
1. Tarayıcınızdan Jenkins arayüzüne (`http://localhost:8080`) erişin.
2. Sol menüden **"New Item"** seçeneğine tıklayın.
3. Projeye bir isim verin ve **"Pipeline"** türünü seçerek OK tuşuna basın.
4. Karşınıza çıkan ayar sayfasında en alta inerek **"Pipeline"** sekmesini bulun.
5. "Definition" kısmından "Pipeline script" seçeneğini aktif tutarak, düzenlediğiniz `Jenkinsfile` dosyasının tüm içeriğini buradaki kutuya yapıştırın.
6. Kaydedin ve sol menüden **"Build Now"** butonuna basın!

Sistem sırasıyla Gitleaks, SonarQube, Trivy, ZAP ve Nuclei'yi konteynerler halinde ayağa kaldırıp taramaları yapacak ve saniyeler içinde tüm sonuçları DefectDojo panelinize aktaracaktır.

---

## Örnek Zafiyet Haritası
Bu pipeline'ı OWASP Juice Shop üzerinde çalıştırdığınızda DefectDojo panosunda yaklaşık olarak şu tabloyla karşılaşacaksınız:
* **Secret Scanning:** 4000+ ifşa olmuş token ve parola.
* **SCA:** 90+ CVE kütüphane zafiyeti.
* **DAST:** Güvenlik başlığı (CSP) eksiklikleri ve davranışsal zafiyetler.

