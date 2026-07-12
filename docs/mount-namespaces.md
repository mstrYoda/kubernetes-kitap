# Mount Namespaces

## Mount Nedir?

Linux'ta her şey bir dosyadır. Bir diski, bir USB'yi ya da bir ağ klasörünü kullanabilmek için önce onu dosya sistemindeki bir noktaya **bağlamanız** gerekir. Bu bağlama işlemine **mount** denir.

Örneğin elinizde bir USB flash bellek var. Linux'ta bu belleği takar takmaz otomatik kullanıma girmez; önce sisteme tanıtılması yani mount edilmesi gerekir. Mount işlemi sayesinde disk, dosya ağacındaki bir dizine bağlanır ve oradan erişilebilir hale gelir.

```
STORAGE DEVICE
──────────────────────────────────────────
┌────────┐   ┌────────┐   ┌────────┐
│ Part 1 │   │ Part 2 │   │ Part 3 │
│ (sda1) │   │ (sda2) │   │ (sda3) │
│  ext4  │   │  swap  │   │ btrfs  │
└───┬────┘   └───┬────┘   └───┬────┘
    │             │             │
    ▼             ▼             ▼
  [/root]      [swap]        [/home]
                   │
                   ▼
            Kernel Magic ✨
                   │
                   ▼
UNIFIED FILE SYSTEM
───────────────────
📁 /              ← sda1 buraya bağlandı
├── bin/
├── etc/
├── home/         ← sda3 buraya bağlandı
├── var/
└── usr/
```

---

## Mount Türleri

### 1. Cihaz Mount (Device Mount)

Fiziksel bir aygıtı ya da disk bölümünü sisteme tanıtma işlemi. USB, harici disk veya partition'ları bu yöntemle bağlarsınız.

```bash
# USB flash belleği /mnt/usb dizinine bağlama
mount /dev/sdb1 /mnt/usb

# Disk bölümünü /home dizinine bağlama
mount /dev/sda3 /home

# Bağlı aygıtları listeleme
lsblk
```

Mount işleminden sonra dosya sistemi şöyle görünür:

```
FİZİKSEL AYGIT          DOSYA SİSTEMİ
──────────────           ──────────────────────
/dev/sdb1      ───────►  /mnt/usb/
                              ├── foto.jpg
                              └── belge.pdf

/dev/sda3      ───────►  /home/
                              ├── ali/
                              └── ayse/
```

---

### 2. Bind Mount

Var olan bir klasörü, başka bir konuma **aynı anda** erişilebilir kılma işlemi. Dosyalar kopyalanmaz; aynı dosyaya iki farklı yoldan bakılır. Docker'da `-v /host/klasör:/container/klasör` dediğinizde arka planda tam olarak bu çalışır.

```bash
# /source/project klasörünü /workspace/active üzerinden de erişilebilir yap
mount --bind /source/project /workspace/active

# Salt okunur bind mount (container'a config dosyası vermek için idealdir)
mount --bind /etc/nginx /container/etc/nginx
mount -o remount,ro,bind /container/etc/nginx
```

```
DİSK ÜZERİNDE GERÇEK YER        ERİŞİLEBİLİR NOKTALAR
─────────────────────────        ─────────────────────────────
/source/project/          ◄───── /source/project/
   ├── app.py                    /workspace/active/
   └── config.yaml       ◄─────  (aynı dosyalar, farklı yol)
```

---


## Mount Namespace Nedir?

Mount namespace, bir process grubunun **hangi mount noktalarını göreceğini** izole eder. Aynı anda iki farklı namespace, birbirinden tamamen farklı dosya sistemi görünümlerine sahip olabilir.

```
İZOLASYON OLMADAN                    İZOLASYON İLE
──────────────────                    ──────────────────────────────────────

📁 /                                  WebApp NS         Database NS
├── bin/                              ┌──────────┐      ┌──────────┐
├── etc/                              │ 📁 /     │      │ 📁 /     │
├── var/                              │ ├── bin/ │      │ ├── bin/ │
└── sensitive-data/                   │ ├── lib/ │      │ ├── lib/ │
    ├── webapp-secrets/  ← herkes     │ └── data/│      │ └── data/│
    ├── database-config/ ← herkes     │  └─ app/ │      │  └─ db/  │
    └── api-keys/        ← herkes     └──────────┘      └──────────┘

 Tüm uygulamalar her şeyi görür   Her uygulama yalnızca kendi verisini görür
```

Bu izolasyonun pratikte nasıl kurulduğunu adım adım görelim. Hedef: WebApp ve Database uygulamalarının birbirinin verisini görememesi.

**Host tarafında veri dizinlerini hazırla:**

```bash
# Her uygulama için ayrı veri klasörleri
mkdir -p /data/webapp
mkdir -p /data/database

echo "webapp-secret-key=abc123" > /data/webapp/secrets.conf
echo "db-password=hunter2"      > /data/database/db.conf
```

---

**WebApp Namespace — Sadece kendi verisini görür:**

```bash
# Yeni bir mount namespace içinde bash başlat
sudo unshare --mount /bin/bash

# Sadece webapp datasını erişilebilir yere bağla
mkdir -p /mnt/app-data
mount --bind /data/webapp /mnt/app-data

# Namespace içinden kontrol et
ls /mnt/app-data
```

```
secrets.conf
```

```bash
# Database verisine doğrudan erişmeye çalış
ls /data/database
```

```
ls: cannot access '/data/database': No such file or directory
```

> **Not:** Bind mount sadece belirli bir mount noktasını etkiler. Bu örnekte `/data/database` dizini namespace içinde ayrıca mount edilmediği için erişilemez. Ancak `../` ile üst dizine çıkış gibi yol manipülasyonları bind mount ile engellenemez. Tam dosya sistemi izolasyonu için `pivot_root` veya `chroot` mekanizmaları gereklidir — bu konu [Chroot ve Pivot_root](chroot-vs-pivot-root.md) sayfasında detaylı olarak ele alınmaktadır.

```bash
exit  # WebApp namespace'inden çık
```

---

**Database Namespace — Sadece kendi verisini görür:**

```bash
# Yeni ve bağımsız bir mount namespace
sudo unshare --mount /bin/bash

# Sadece database datasını bağla
mkdir -p /mnt/db-data
mount --bind /data/database /mnt/db-data

# Namespace içinden kontrol et
ls /mnt/db-data
```

```
db.conf
```

```bash
# WebApp datasına doğrudan erişmeye çalış
ls /data/webapp
```

```
ls: cannot access '/data/webapp': No such file or directory
```

```bash
exit  # Database namespace'inden çık
```

---

**Host tarafından her şey görünür — ama namespace'ler birbirini görmez:**

```bash
# Host üzerinde her iki veri dizini de açık
ls /data/webapp
ls /data/database
```

```
# /data/webapp:
secrets.conf

# /data/database:
db.conf
```

```
SONUÇ
──────────────────────────────────────────────────────
              Host Namespace
              ┌───────────────────────────┐
              │ /data/webapp/secrets.conf │  ← host her ikisini de görür
              │ /data/database/db.conf    │
              └───────────┬───────────────┘
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
  WebApp Namespace            Database Namespace
  ┌──────────────────┐        ┌──────────────────┐
  │ /mnt/app-data/   │        │ /mnt/db-data/    │
  │  secrets.conf ✅  │        │  db.conf      ✅  │
  │                  │        │                  │
  │ /data/database ❌ │        │ /data/webapp  ❌  │
  └──────────────────┘        └──────────────────┘
```

