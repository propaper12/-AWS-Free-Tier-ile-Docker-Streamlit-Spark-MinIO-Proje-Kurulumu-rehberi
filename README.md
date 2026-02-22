
# AWS Free Tier ile Docker + Streamlit + Spark + MinIO Proje Kurulumu Rehberi

Bu rehberi, güncel projelerimde aktif olarak kullanıyorum. Benzer mimarileri kurmak isteyen herkesin faydalanabilmesi için açık ve uygulanabilir şekilde dokümante ettim.

Bu doküman, AWS Free Tier üzerinde **Docker tabanlı veri uygulamalarını stabil, güvenli ve performanslı şekilde ayağa kaldırmak** amacıyla hazırlanmış profesyonel bir deployment rehberidir.

Spark, PostgreSQL, MinIO ve Streamlit gibi RAM tüketimi yüksek servislerin **aynı anda yalnızca 1 GB RAM’e sahip `t2.micro` instance üzerinde sorunsuz çalıştırılmasını** hedefler.

---

## Kritik Mülakat Notu

AWS Free Tier `t2.micro` makineleri yalnızca **1 GB RAM** içerir.

Aşağıdaki servisler **aynı anda** çalıştırıldığında:

- PostgreSQL  
- MinIO  
- Streamlit  
- PySpark  

makine **yüksek ihtimalle Out of Memory (OOM)** hatası vererek kilitlenir.

### Profesyonel Çözüm

Bu problem, Linux üzerinde **Swap (Sanal RAM)** oluşturularak aşılmıştır.

> **Mülakat ifadesi:**  
> *“AWS Free Tier’ın 1 GB RAM limitini Linux üzerinde Swap File oluşturarak aştım ve sistemi stabil hale getirdim.”*

Bu ifade, güçlü bir **altyapı bilgisi ve sistem farkındalığı** gösterir.

---

# Kurulum Adımları

---

## 1. AWS EC2 Makinesi Oluşturma

AWS Console → EC2 → **Launch Instance**

### Temel Ayarlar

- **Name:** `GECI-Junior-Project`
- **AMI:** Ubuntu 22.04 veya 24.04 LTS *(Free tier eligible)*
- **Instance Type:** `t2.micro`
- **Key Pair:**  
  - Create new key pair  
  - Name: `geci-key`  
  - Format: `.pem`

### Security Group Ayarları

| Type | Port | Açıklama |
|--------|--------|-------------|
| SSH | 22 | Terminal erişimi |
| Custom TCP | 8501 | Streamlit |
| Custom TCP | 9001 | MinIO Console |

**Source:** `0.0.0.0/0`

### Storage

Varsayılan **8 GiB → 30 GiB** olarak değiştirilmelidir.  
*sag taraftakı Launch instance butonuna basın (Docker, Spark ve MinIO için disk alanı kritik öneme sahiptir.)*

---

## 2. Sunucuya SSH ile Bağlanma
* IP_ADRESIN bulmak için ilk önce ec2 girin instance kısmında olsutrudgumuz kısımından intance ID ye basın orada  Public IPv4 address adresini goreceksınız bunu kopyalıyıp yapıstrıın*
```bash
chmod 400 geci-key.pem
ssh -i "geci-key.pem" ubuntu@IP_ADRESIN
```
## 3. Swap (Sanal RAM) Oluşturma

Bu adım **kritik öneme sahiptir.** AWS Free Tier `t2.micro` makineleri yalnızca **1 GB RAM** içerdiği için, servislerin stabil çalışabilmesi adına **2 GB Swap alanı** oluşturulur.

```bash
sudo fallocate -l 2G /swapfile  
sudo  chmod  600 /swapfile  
sudo mkswap /swapfile  
sudo swapon /swapfile
```

### Kontrol

```free -h```

Swap alanı listeleniyorsa işlem başarıyla tamamlanmıştır.

## 4. Docker ve Docker Compose Kurulumu

sudo apt update && sudo apt install docker.io docker-compose -y  
sudo usermod -aG docker ubuntu  
exit

Tekrar bağlan:

ssh  -i  "geci-key.pem" ubuntu@IP_ADRESIN

----------

## 5. Proje Dosyalarının Hazırlanması (Lokal)

Aşağıdaki dosya ve klasörleri `.zip` haline getirin:

-   `docker-compose.yml`
    
-   `Home.py`
    
-   `Veri_Setleri/`
    
-   `.env`
    
-   `requirements.txt`
    

Zip dosyasının adı:

proje.zip

----------
### 5.1 GitHub Repository Hazırlığı (Lokal)  
  
Proje klasöründe aşağıdaki dosyaların bulunduğundan emin olun:  
  
- `docker-compose.yml`  
- `Home.py`  
- `Veri_Setleri/`  
- `.env.example`  
- `requirements.txt`  
  
> Güvenlik sebebiyle `.env` dosyası **repo içine eklenmemelidir.**  
> Bunun yerine `.env.example` dosyası oluşturulmalıdır.  
  
---  
  
### 5.2 AWS Sunucu Üzerinde Git Kurulumu  
  
```bash  
sudo apt update && sudo apt install git -y
```
### 5.3 GitHub Repository Klonlama

git clone https://github.com/KULLANICI_ADIN/REPO_ADI.git  
cd REPO_ADI

----------

### 5.4 Ortam Değişkenlerinin Tanımlanması (.env)

Sunucu üzerinde `.env` dosyasını oluştur:

nano .env

### 5.5 Docker Servislerini Başlatma

sudo docker-compose up --build  -d  
sudo docker ps

----------

## Notlar

-   `.env` dosyası **asla GitHub reposuna push edilmemelidir.**
    
-   Bu yöntem, **production deployment için önerilen standart yaklaşımdır.**
    
-   Kod güncellemesi için:
    

git pull  
sudo docker-compose up --build  -d


## 6. Projeyi AWS Sunucuya Gönderme (SCP)
bunun için yeni terminal acın oraya bunu kopyalaıyn
scp -i  "geci-key.pem" proje.zip ubuntu@IP_ADRESIN:~

----------

## 7. Dosyayı Sunucuda Açma

sudo apt install unzip -y  
unzip proje.zip -d geci_projesi  
cd geci_projesi  
ls  -a
cd projenin_adi
`docker-compose.yml`, `Home.py` ve diğer dosyaları görüyorsanız işlem başarılıdır.

----------

## 8. Docker Servislerini Başlatma

sudo docker-compose up --build  -d  
sudo docker ps

----------

## 9. Tarayıcıdan Erişim

http://IP_ADRESIN:8501 buradakı ıp ise publıc ıpv4 ıpniz olacak

Streamlit arayüzü açılıyorsa sistem başarıyla ayağa kalkmıştır.

----------

# Kazanımlar

-   AWS Free Tier üzerinde gerçekçi üretim ortamı simülasyonu
    
-   Docker tabanlı mikroservis mimarisi
    
-   Linux sistem yönetimi
    
-   Bellek yönetimi (Swap)
    
-   Data Science + DevOps entegrasyonu
    

----------

# Notlar

-   Bu rehber tüm veri bilimi ve veri mühendisliği projelerinde tekrar tekrar kullanılabilir.
    
-   Free Tier sınırları içinde maksimum performans elde edilmesini sağlar.
    
-   CV ve teknik mülakatlarda ciddi fark yaratır.
-   AWS Free Tier kapsamında aylık toplam **720 saat** EC2 kullanım hakkımız var. Ama bu süre **her açtığımız sunucu için ayrı ayrı 720 saat değildir.** Tüm EC2 sunucularının açık kaldığı süre **toplamda** hesaplanır.

Bu yüzden yeni bir proje açmadan önce, **eski EC2 sunucumuzu mutlaka silmemiz gerekir.**

Eski EC2 Sunucusunu Silme Adımları
1. AWS Console → EC2 → **Instances** kısmına giriyoruz.
2. Sileceğimiz sunucuyu seçiyoruz.
3. Üst taraftaki **Instance State** menüsüne basıyoruz.
4. En altta bulunan **Terminate instance (Delete)** seçeneğine tıklıyoruz.
5. Onayladıktan sonra sunucu birkaç dakika içinde tamamen siliniyor.

Bu işlemden sonra artık yeni sunucularımızı **rahat rahat** açabiliriz.

---
Neden Silmemiz Gerek?

Çünkü AWS süreyi **sunucu başına değil, toplam kullanım süresine göre** hesaplıyor.
Örnek:
- İlk açtığımız sunucu → **10 saat açık kaldı**
- Yeni açtığımız sunucu → **10 saat açık kaldı**
Toplam kullanım süresi:
Yani her sunucu için ayrı ayrı 720 saat hakkımız yok.  
Hepsi **aynı 720 saatlik limitten düşüyor.**  
  
---  
  
### Kısaca Özet  
  
- İşin bitince EC2 sunucunu **sil**  
- Boş yere açık bırakma  
- Böylece Free Tier süreni **en verimli şekilde kullanırsın**

### 2. Adım: Spark ETL Sürecini Başlat
bu kısım benım kendı projem için kullanmanıza gerek yok
Şimdi MinIO'daki o ham verileri alıp, temizleyip Postgres veritabanına atacağız.

Bash

```
sudo docker exec -it geci_dashboard python ingest_to_s3.py

sudo docker exec -it geci_dashboard python etl_spark_to_db.py

sudo docker restart geci_dashboard

