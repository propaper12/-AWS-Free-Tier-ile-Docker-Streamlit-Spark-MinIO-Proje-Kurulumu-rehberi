# AWS Free Tier ile Docker + Streamlit + Spark + MinIO Proje Kurulumu rehberi
Bu rehberi guncel projelerimde kullanıyorum sizlerinde kullanması ıcın boyle bir rehber hazırladım tesekkür ederim şimdiden.
Bu doküman, AWS Free Tier üzerinde **Docker tabanlı veri uygulamalarını stabil ve performanslı şekilde ayağa kaldırmak** için hazırlanmış profesyonel bir deployment rehberidir.

Spark, PostgreSQL, MinIO ve Streamlit gibi RAM tüketimi yüksek servislerin **aynı anda 1 GB RAM’e sahip t2.micro instance üzerinde sorunsuz çalıştırılmasını** hedefler.

---

## Kritik Mülakat Notu

AWS Free Tier `t2.micro` makineleri yalnızca **1 GB RAM** içerir.

Eğer aşağıdaki servisler **aynı anda** çalıştırılırsa:

- PostgreSQL  
- MinIO  
- Streamlit  
- PySpark  

makine **çok büyük ihtimalle Out of Memory (OOM)** hatası vererek kilitlenir.

### Profesyonel Çözüm

Bu problem **Linux Swap (Sanal RAM)** oluşturularak aşılmıştır.

> **Mülakat ifadesi:**  
> *“AWS Free Tier’ın 1 GB RAM limitini Linux üzerinde Swap File oluşturarak aştım ve sistemi stabil hale getirdim.”*

Bu ifade, güçlü bir **altyapı ve sistem farkındalığı** gösterir.

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
|--------|--------|--------------|
| SSH | 22 | Terminal erişimi |
| Custom TCP | 8501 | Streamlit |
| Custom TCP | 9001 | MinIO Console |

**Source:** `0.0.0.0/0`

### Storage

- Varsayılan **8 GiB → 30 GiB** yapılmalıdır.

---

## 2. Sunucuya SSH ile Bağlanma

```bash
chmod 400 geci-key.pem
ssh -i "geci-key.pem" ubuntu@IP_ADRESIN

---
##3. Swap (Sanal RAM) Oluşturma

Bu adım kritik öneme sahiptir.

sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

Kontrol:

free -h
4. Docker ve Docker Compose Kurulumu
sudo apt update && sudo apt install docker.io docker-compose -y
sudo usermod -aG docker ubuntu
exit

Tekrar bağlan:

ssh -i "geci-key.pem" ubuntu@IP_ADRESIN
5. Proje Dosyalarının Hazırlanması (Lokal)

Aşağıdaki dosyaları .zip haline getir:

docker-compose.yml

Home.py

Veri_Setleri/

.env

requirements.txt

Dosya adı:

proje.zip
6. Projeyi AWS Sunucuya Gönderme (SCP)
scp -i "geci-key.pem" proje.zip ubuntu@IP_ADRESIN:~
7. Dosyayı Sunucuda Açma
sudo apt install unzip -y
unzip proje.zip -d geci_projesi
cd geci_projesi
ls -a
8. Docker Servislerini Başlatma
sudo docker-compose up --build -d
sudo docker ps
9. Tarayıcıdan Erişim
http://IP_ADRESIN:8501
