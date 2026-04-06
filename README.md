# Otonom DevSecOps Master Pipeline

Bu repository, **"Shift-Left"**  prensiplerini merkeze alarak sıfırdan inşa edilmiş, %100 otonom çalışan uçtan uca bir DevSecOps CI/CD laboratuvar ortamıdır. 

Sistem, bilerek zafiyetli bırakılmış **OWASP Juice Shop** uygulamasını hedef alır. Kaynak kodun taranmasından uygulamanın canlı ortamda hacklenmesine kadar tüm güvenlik testleri insan müdahalesi olmadan gerçekleştirilir ve sonuçlar merkezi bir zafiyet panosunda (DefectDojo) toplanır.

<img width="2816" height="1536" alt="Gemini_Generated_Image_7yzb8o7yzb8o7yzb" src="https://github.com/user-attachments/assets/f09507e8-097b-4bdc-aff0-f80b8bde07ea" />

---

## Kullanılan Teknolojiler ve Güvenlik Araçları

Sistemimiz, siber güvenlik endüstrisinde standart olarak kabul edilen açık kaynaklı araçların birbiriyle entegre edilmesinden oluşur:

 **Orkestrasyon:** Jenkins (Docker-in-Docker mimarisi ile)
<img width="1342" height="414" alt="image" src="https://github.com/user-attachments/assets/8f0cf4c2-6c99-4155-81ef-6020743a0c8d" />


 **Zafiyet Yönetim Merkezi:** OWASP DefectDojo
<img width="1883" height="834" alt="image" src="https://github.com/user-attachments/assets/7cf82c2b-71c7-4911-8c50-96b7c6cd3516" />


 **Secret Scanning:** Gitleaks
<img width="668" height="261" alt="1_Ben-0Jp8xsszbJR1JS9q8g" src="https://github.com/user-attachments/assets/2872317d-5e02-444c-b8ee-f7ef4317595c" />


 **SAST (Statik Kod Analizi):** SonarQube
<img width="1915" height="784" alt="image" src="https://github.com/user-attachments/assets/f1a52b0f-3880-4629-a708-98860592900c" />


 **SCA:** Trivy
<img width="329" height="153" alt="images" src="https://github.com/user-attachments/assets/1c56b927-1654-4071-8093-191d41bdc235" />

  
 **DAST (Dinamik Analiz):** [OWASP ZAP](https://www.zaproxy.org/) & [ProjectDiscovery Nuclei](https://github.com/projectdiscovery/nuclei)
<img width="860" height="277" alt="1586766458581" src="https://github.com/user-attachments/assets/485664f6-1bec-4a76-8786-7ffabd528a84" /> <img width="498" height="178" alt="1_U5MxN6VKGkqL_RbHaUC5tg" src="https://github.com/user-attachments/assets/33b3d108-ebea-4666-949d-2848c00c5550" />


---

## Kurulum ve Kullanım Rehberi

### Ön Koşullar
* Docker ve Docker Compose'un bilgisayarınızda kurulu olması.
* Sistemde minimum 8 GB (Tercihen 16 GB) boş RAM bulunması.

### 1. Ana Projeyi Klonlayın
Öncelikle bu repoyu bilgisayarınıza indirin:

* git clone https://github.com/TA2ZTD/devsecops-lab.git

* cd devsecops-lab

### 2. DefectDojo Kurulumu ve Başlatılması
Zafiyetleri görselleştireceğimiz merkezi panomuzu ayağa kaldırıyoruz:

* git clone https://github.com/DefectDojo/django-DefectDojo

* cd django-DefectDojo

* ./dc-build-local-images.sh

* ./dc-up-d.sh postgres-redis

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
<img width="1007" height="196" alt="image" src="https://github.com/user-attachments/assets/93504dc2-58cd-481a-a19f-8b7acefc1688" />

---

## Örnek Zafiyet Haritası
Bu pipeline'ı OWASP Juice Shop üzerinde çalıştırdığınızda DefectDojo panosunda yaklaşık olarak şu tabloyla karşılaşacaksınız:
* **Secret Scanning:** 4000+ ifşa olmuş token ve parola.
* **SCA:** 90+ CVE kütüphane zafiyeti.
* **DAST:** Güvenlik başlığı (CSP) eksiklikleri ve davranışsal zafiyetler.
<img width="1274" height="866" alt="image" src="https://github.com/user-attachments/assets/3c1b75ce-8796-4519-933b-f2eaabd0cdf2" />

---
---

ENGLISH DESCRIPTION

# Autonomous DevSecOps Master Pipeline

This repository is a fully autonomous, end-to-end DevSecOps CI/CD laboratory environment built from scratch, centering on "Shift-Left" security principles.

The system targets the intentionally vulnerable OWASP Juice Shop application. From source code scanning to dynamic application security testing in a live environment, all security tests are performed without human intervention, and the results are aggregated in a central vulnerability dashboard (DefectDojo).

<img width="2816" height="1536" alt="Gemini_Generated_Image_7yzb8o7yzb8o7yzb" src="https://github.com/user-attachments/assets/f09507e8-097b-4bdc-aff0-f80b8bde07ea" />

---

## Technologies and Security Tools Used

Our system consists of industry-standard open-source tools integrated with each other:

 **Orchestration:** Jenkins (utilizing Docker-in-Docker architecture)
<img width="1342" height="414" alt="image" src="https://github.com/user-attachments/assets/297415a4-c1dd-46f6-85a3-5d1492f84081" />


 **Vulnerability Management Center:** OWASP DefectDojo
*<img width="1883" height="834" alt="image" src="https://github.com/user-attachments/assets/776baaae-837e-48d8-9aa8-aa72d417cfda" />


 **Secret Scanning:** Gitleaks
*<img width="668" height="261" alt="1_Ben-0Jp8xsszbJR1JS9q8g" src="https://github.com/user-attachments/assets/2872317d-5e02-444c-b8ee-f7ef4317595c" />


 **SAST (Static Application Security Testing):** SonarQube,
*<img width="1915" height="784" alt="image" src="https://github.com/user-attachments/assets/4205c0c5-3310-4478-baa6-4b1f2b291036" />


 **SCA (Software Composition Analysis):** Trivy
*<img width="329" height="153" alt="images" src="https://github.com/user-attachments/assets/1c56b927-1654-4071-8093-191d41bdc235" />


 DAST (Dynamic Application Security Testing): [OWASP ZAP](https://www.zaproxy.org/) & [ProjectDiscovery Nuclei](https://github.com/projectdiscovery/nuclei)
*<img width="860" height="277" alt="1586766458581" src="https://github.com/user-attachments/assets/485664f6-1bec-4a76-8786-7ffabd528a84" />
*<img width="498" height="178" alt="1_U5MxN6VKGkqL_RbHaUC5tg" src="https://github.com/user-attachments/assets/33b3d108-ebea-4666-949d-2848c00c5550" />
---

## Installation and Usage Guide

### Prerequisites
* Docker and Docker Compose installed on your machine.
* Minimum 8 GB (Preferably 16 GB) of free RAM.

### 1. Clone the Main Project
First, download this repository to your computer:

* git clone https://github.com/TA2ZTD/devsecops-lab.git

* cd devsecops-lab

### 2. DefectDojo Setup and Initialization
Launch our central dashboard where vulnerabilities will be visualized:

* git clone https://github.com/DefectDojo/django-DefectDojo

* cd django-DefectDojo

Note: Before starting, you must change the target port to 8081 in the docker-compose.yml file (it defaults to 8080, which will conflict with Jenkins). 

After that, run the following commands:

* ./dc-build-local-images.sh

* ./dc-up-d.sh postgres-redis

*Note: Once the installation is complete, change the target port to 8081 in the docker-compose.yml file (it defaults to 8080, but Jenkins is already running on that port) and make a note of the admin password displayed in the terminal. Log in to Dojo at http://localhost:8081. Go to the "API v2 Key" tab in the profile menu (top right) to copy your key.*

### 3. Start Jenkins and SonarQube
Return to the main directory (where the original docker-compose.yml is located) and start the infrastructure:

* docker-compose up -d

### 4. Pipeline Configuration
Open the Jenkinsfile in the cloned directory with a text editor and update the following variables in the environment block:
* DOJO_URL (Default: http://host.docker.internal:8081)
* DOJO_API_KEY (The key you copied above)
* PRODUCT_NAME (The project name as you want it to appear in DefectDojo, e.g., "OWASP Juice Shop")

### 5. Launch the DevSecOps Pipeline
1. Access the Jenkins interface at http://localhost:8080.
2. Click "New Item" from the left menu.
3. Give the project a name, select "Pipeline", and click OK.
4. Scroll down to the Pipeline section.
5. Keep "Pipeline script" as the definition and paste the entire contents of your edited Jenkinsfile into the script box.
6. Save and click the "Build Now" button!

The system will sequentially spin up Gitleaks, SonarQube, Trivy, ZAP, and Nuclei as ephemeral containers, conduct the scans, and upload all results into your DefectDojo dashboard automatically.
<img width="1007" height="196" alt="image" src="https://github.com/user-attachments/assets/93504dc2-58cd-481a-a19f-8b7acefc1688" />
---

## Sample Vulnerability Map
When you run this pipeline against OWASP Juice Shop, you will see a result similar to this in DefectDojo:
* Secret Scanning: 4000+ exposed tokens and passwords.
* SCA: 90+ CVE library vulnerabilities.
* DAST: Missing security headers (CSP) and behavioral vulnerabilities.
<img width="1274" height="866" alt="image" src="https://github.com/user-attachments/assets/dd9992c5-01a9-46a7-8ad9-8ad01e5d90b0" />
