# Chroot ve Pivot_root

Linux sistemlerde bir process'in **root dizinini (/) değiştirmeye** yarayan iki temel mekanizma vardır: `chroot` ve `pivot_root`. İkisi de benzer bir amaca hizmet eder — bir process'in dosya sistemi görünümünü kısıtlamak — ancak çalışma prensipleri, güvenlik düzeyleri ve kullanım alanları birbirinden farklıdır. Konteyner teknolojilerinin dosya sistemi izolasyonunu nasıl sağladığını anlamak için bu iki mekanizmayı iyi bilmek gerekir.

---

## Chroot Nedir?

`chroot` (change root), bir process'in **kök dizinini (/) başka bir dizine işaret ettiren** bir sistem çağrısıdır. `chroot` çağrısından sonra process, belirtilen dizini kendi kök dizini olarak görür ve bu dizinin üstüne çıkamaz.

```
CHROOT ÖNCESİ                          CHROOT SONRASI
───────────────                          ──────────────────────────────
                                         Process'in gördüğü dosya sistemi:
📁 / (gerçek root)                       📁 / (aslında /srv/container)
├── bin/                                 ├── bin/
├── etc/                                 ├── etc/
├── home/                                ├── lib/
├── srv/                                 └── home/
│   └── container/   ← chroot noktası        └── app/
│       ├── bin/
│       ├── etc/                         Process artık /srv/container'ın
│       ├── lib/                         üstünü GÖRMEZ (teoride).
│       └── home/
│           └── app/
└── var/
```

### Chroot Nasıl Çalışır?

`chroot` sistem çağrısı, kernel'de process'in `fs_struct` yapısındaki `root` alanını değiştirir. Bu sayede process'in tüm mutlak yol (`/...`) çözümlemeleri yeni root dizininden başlar.

Ancak **sadece dosya yolu çözümlemesi** değişir — process'in açık dosya tanımlayıcıları (file descriptor), mevcut çalışma dizini (cwd) ve diğer kaynakları değişmez. Bu durum güvenlik açıklarına yol açabilir.

---

## Adım Adım: Chroot ile Basit Bir İzole Ortam Oluşturma

### 1. Chroot ortamı için dizin yapısını hazırla

İlk olarak, izole ortamda çalışacak temel dosya sistemini oluşturmamız gerekiyor. Bir shell çalıştırabilmek için en azından `/bin/bash` ve bağımlı kütüphaneler bulunmalıdır:

```bash
# Chroot ortamının kök dizini
mkdir -p /srv/chroot-demo

# Temel dizinleri oluştur
mkdir -p /srv/chroot-demo/{bin,lib,lib64,etc,home,proc}
```

### 2. Shell ve bağımlılıklarını kopyala

`bash` binary'sini ve ihtiyaç duyduğu paylaşımlı kütüphaneleri chroot ortamına kopyalıyoruz:

```bash
# bash binary'sini kopyala
cp /bin/bash /srv/chroot-demo/bin/

# bash'in bağımlı olduğu kütüphaneleri bul
ldd /bin/bash
```

```
linux-vdso.so.1 (0x00007ffd5c7fe000)
libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
/lib64/ld-linux-x86-64.so.2
```

```bash
# Bağımlı kütüphaneleri kopyala
cp /lib/x86_64-linux-gnu/libtinfo.so.6 /srv/chroot-demo/lib/
cp /lib/x86_64-linux-gnu/libdl.so.2 /srv/chroot-demo/lib/
cp /lib/x86_64-linux-gnu/libc.so.6 /srv/chroot-demo/lib/
cp /lib64/ld-linux-x86-64.so.2 /srv/chroot-demo/lib64/

# ls komutunu da ekleyelim (dosya listeleme için)
cp /bin/ls /srv/chroot-demo/bin/

# ls'in bağımlılıklarını kopyala
ldd /bin/ls | grep -o '/lib[^ ]*' | while read lib; do
    dir=$(dirname "$lib")
    mkdir -p "/srv/chroot-demo$dir"
    cp "$lib" "/srv/chroot-demo$dir/"
done
```

### 3. Chroot ortamına gir

```bash
# Chroot ile yeni root'a geçiş yap
sudo chroot /srv/chroot-demo /bin/bash
```

### 4. Chroot ortamının içinden dosya sistemini kontrol et

```bash
# Yeni root dizinindeki dosyaları listele
ls /
```

```
bin  etc  home  lib  lib64  proc
```

```bash
# Üst dizine çıkmayı dene
ls /../../
```

```
bin  etc  home  lib  lib64  proc
```

`/../../` yazarak üst dizine çıkmaya çalışsak bile, kernel yol çözümlemesinde root sınırını aşmamıza izin vermez ve bizi yine aynı kök dizine yönlendirir.

```bash
# Chroot ortamından çık
exit
```

---

## Chroot'un Güvenlik Zafiyetleri

`chroot` izolasyonu **tam bir güvenlik mekanizması değildir**. Aşağıdaki durumlarda bir process chroot'tan kaçabilir (chroot escape):

### 1. Açık File Descriptor ile Kaçış

Eğer process chroot çağrısından **önce** kök dizine (`/`) ait bir file descriptor açmışsa, `fchdir()` ile o dizine geri dönebilir:

```c
// chroot-escape.c — Kavramsal örnek (root yetkisi gerektirir)
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>

int main() {
    // 1. Chroot ÖNCESİNDE gerçek root'a bir fd aç
    int real_root_fd = open("/", O_RDONLY);

    // 2. Chroot uygula
    chroot("/srv/chroot-demo");

    // 3. Önceden açtığımız fd ile gerçek root'a geri dön
    fchdir(real_root_fd);

    // 4. Artık chroot dışındayız, gerçek root'a ulaşabiliriz
    chroot(".");

    // 5. Tüm dosya sistemine erişim var
    execl("/bin/bash", "bash", NULL);
    return 0;
}
```

```
CHROOT ESCAPE MEKANİZMASI
──────────────────────────────────────────────────────
  ┌─────────────────────────────┐
  │ Process (chroot içinde)     │
  │                             │
  │  fd=3 ──► gerçek root (/)   │  ← chroot öncesi açılmış fd
  │                             │
  │  fchdir(fd=3)               │  ← fd üzerinden gerçek root'a atla
  │  chroot(".")                │  ← yeni root olarak gerçek root'u ata
  │                             │
  │  💥 KAÇIŞ BAŞARILI          │
  └─────────────────────────────┘
```

### 2. Root Yetkisi ile Kaçış

`chroot` içindeki process **root yetkisine** sahipse, `mknod` ile cihaz dosyaları oluşturabilir, `mount` ile dosya sistemi bağlayabilir veya yukarıdaki fd tekniğini kullanabilir. Bu nedenle chroot, güvenilmeyen processleri izole etmek için **tek başına yeterli değildir**.

> **Önemli:** `chroot` bir güvenlik aracı değil, bir dosya sistemi görünümü kısıtlama aracıdır. Linux man sayfasında açıkça belirtilir: *"This call does not change the current working directory, so that after the call '.' can be outside the tree rooted at '/'."*

---

## Pivot_root Nedir?

`pivot_root` sistem çağrısı, **tüm mount namespace'inin kök dosya sistemini (root filesystem) değiştiren** daha güçlü bir mekanizmadır. `chroot`'tan farklı olarak sadece process'in yol çözümlemesini değil, **mount hiyerarşisinin kendisini** değiştirir.

`pivot_root` iki parametre alır:
- **new_root**: Yeni kök dosya sistemi olacak mount noktası
- **put_old**: Eski kök dosya sisteminin taşınacağı dizin

```
PIVOT_ROOT İŞLEMİ
──────────────────────────────────────────────────────────────────

ÖNCESİ:                                SONRASI:
📁 / (eski root filesystem)            📁 / (yeni root filesystem)
├── bin/                                ├── bin/
├── etc/                                ├── etc/
├── var/                                ├── lib/
└── mnt/                                ├── home/
    └── newroot/  ← yeni root           │   └── app/
        ├── bin/                        └── .old_root/  ← eski root buraya taşındı
        ├── etc/                            ├── bin/
        ├── lib/                            ├── etc/
        └── home/                           └── var/
            └── app/
                                        Eski root umount edilebilir → tam izolasyon!
```

### Pivot_root Neden Daha Güvenli?

| Özellik | `chroot` | `pivot_root` |
|---------|----------|-------------|
| Ne değiştirir? | Process'in root yolu çözümlemesini | Mount namespace'in root filesystem'ini |
| Eski root'a erişim | File descriptor ile mümkün | Umount edilerek tamamen kaldırılabilir |
| Escape riski | Yüksek (fd, mknod, mount ile) | Çok düşük (eski root umount edilirse) |
| Mount namespace gerektirir mi? | Hayır | Evet |
| Konteyner runtime kullanımı | Kullanılmaz | Docker, containerd, runc tarafından kullanılır |
| Kernel seviyesi | Sadece yol çözümleme | Mount hiyerarşisi değişimi |

---

> **Not:** `pivot_root` çağrısı eski root filesystem'i `.old_root` gibi bir dizine **taşır** ama otomatik olarak kaldırmaz. Bu dizin umount edilmediği sürece namespace içindeki processler eski root'a `/.old_root/` üzerinden erişebilir — bu da izolasyonu zafiyete uğratır. Bu nedenle `pivot_root` sonrasında eski root **mutlaka umount edilmelidir**.

## Adım Adım: Pivot_root ile Gerçek İzolasyon

### 1. Yeni root filesystem'i hazırla

Pivot_root için yeni kök olacak dosya sistemini oluşturuyoruz. Bu ortam minimal bir Linux root filesystem olmalıdır:

```bash
# Yeni root filesystem dizini
mkdir -p /srv/pivot-demo

# Temel dizin yapısını oluştur
mkdir -p /srv/pivot-demo/{bin,lib,lib64,etc,home,proc,sys,dev,tmp}

# Shell ve temel araçları kopyala
cp /bin/bash /srv/pivot-demo/bin/
cp /bin/ls /srv/pivot-demo/bin/
cp /bin/mount /srv/pivot-demo/bin/
cp /bin/umount /srv/pivot-demo/bin/
cp /bin/cat /srv/pivot-demo/bin/

# Bağımlı kütüphaneleri kopyala
for cmd in bash ls mount umount cat; do
    ldd /bin/$cmd 2>/dev/null | grep -o '/lib[^ ]*' | while read lib; do
        dir=$(dirname "$lib")
        mkdir -p "/srv/pivot-demo$dir"
        cp -n "$lib" "/srv/pivot-demo$dir/" 2>/dev/null
    done
done
```

### 2. Yeni mount namespace oluştur

`pivot_root` sadece bir **mount namespace** içinde çalışır. Bu nedenle önce yeni bir mount namespace oluşturuyoruz:

```bash
# Yeni mount namespace içinde bir shell başlat
sudo unshare --mount /bin/bash
```

### 3. Yeni root'u bir mount noktası haline getir

`pivot_root`, yeni root'un bir **mount noktası** olmasını zorunlu kılar. `bind mount` ile bunu sağlıyoruz:

```bash
# pivot_root yeni root'un bir mount noktası olmasını gerektirir
mount --bind /srv/pivot-demo /srv/pivot-demo
```

### 4. Eski root için dizin oluştur ve pivot_root uygula

```bash
# Eski root'un taşınacağı dizini oluştur
mkdir -p /srv/pivot-demo/.old_root

# pivot_root: yeni root = /srv/pivot-demo, eski root = /srv/pivot-demo/.old_root
pivot_root /srv/pivot-demo /srv/pivot-demo/.old_root
```

### 5. Yeni root'a geçiş yap ve proc mount et

```bash
# Çalışma dizinini yeni root'a ayarla
cd /

# /proc dosya sistemini mount et (process bilgileri için gerekli)
mount -t proc proc /proc
```

### 6. İzolasyonu doğrula

```bash
# Yeni root dizinini listele
ls /
```

```
bin  dev  etc  home  lib  lib64  proc  sys  tmp  .old_root
```

```bash
# Eski root hâlâ .old_root altından erişilebilir
ls /.old_root/
```

```
bin  boot  dev  etc  home  lib  lib64  mnt  opt  proc  root  run
sbin  srv  sys  tmp  usr  var
```

### 7. Eski root'u umount ederek tam izolasyonu sağla

İşte `pivot_root`'un asıl gücü burada ortaya çıkar — eski root filesystem'i tamamen kaldırabilirsiniz:

```bash
# Eski root'u umount et — artık erişim tamamen kesilir
umount -l /.old_root

# Doğrula: eski root artık erişilemez
ls /.old_root/
```

```
(boş)
```
### Neden Eski Root'u Önce `.old_root` Dizinine Taşıdık, Sonra Umount Ettik?

Bu soruyu sormak çok doğal — madem eski root'u kaldıracağız, neden önce bir dizine bağlıyoruz? Bunun **üç temel sebebi** vardır:

**1. `pivot_root` syscall'u bunu zorunlu kılar:**

`pivot_root(new_root, put_old)` çağrısı iki parametre alır. İkinci parametre olan `put_old`, eski root filesystem'in **nereye taşınacağını** belirler. Kernel bu parametreyi zorunlu tutar çünkü bir mount namespace'in root filesystem'i hiçbir zaman "boşta" kalamaz — eski root ya bir yere taşınmalı ya da swap edilmelidir.

```bash
# pivot_root syscall imzası:
# pivot_root(new_root, put_old)
#
# put_old ZORUNLUDUR — eski root'un taşınacağı yer
pivot_root /srv/pivot-demo /srv/pivot-demo/.old_root
```

**2. Geçiş sırasında çalışan processlerin çökmemesi için:**

Pivot_root çağrıldığı anda namespace içinde hâlâ çalışan processler olabilir. Bu processlerin açık dosya tanımlayıcıları (fd), kütüphane bağımlılıkları veya `cwd` değerleri eski root filesystem'indeki dosyalara işaret ediyor olabilir. Eski root aniden kaldırılırsa bu processler çöker. `.old_root` dizinine taşıyarak **kontrollü bir geçiş** sağlanır:

```
PIVOT_ROOT GEÇİŞ SÜRECİ
──────────────────────────────────────────────────────

  Adım 1: pivot_root çağrılır
  ┌────────────────────────────────────────────┐
  │ Yeni Root (/)                              │
  │  ├── bin/bash  (yeni process'ler burada)   │
  │  ├── lib/                                  │
  │  └── .old_root/                            │
  │       ├── bin/   ← eski root hâlâ erişilir │
  │       ├── etc/   ← çalışan processler      │
  │       └── lib/     bağımlılıklarını         │
  │                    hâlâ okuyabilir          │
  └────────────────────────────────────────────┘

  Adım 2: Yeni ortam hazır, umount -l yapılır
  ┌────────────────────────────────────────────┐
  │ Yeni Root (/)                              │
  │  ├── bin/bash                              │
  │  ├── lib/                                  │
  │  └── .old_root/  ← BOŞ, erişim kesildi     │
  └────────────────────────────────────────────┘
```

**3. Umount için bir referans noktası gereklidir:**

Bir filesystem'i umount edebilmek için onun **nerede mount edildiğini** bilmeniz gerekir. `pivot_root` eski root'u `.old_root` dizinine taşıdığında, artık `umount /.old_root` komutuyla bu mount noktasını hedefleyebiliriz. Eğer eski root hiçbir yere taşınmasaydı, onu adresleyecek bir yolumuz olmazdı.

> **Özet:** `pivot_root` → `.old_root`'a taşı → geçiş tamamlandığında `umount -l` ile kaldır. Bu üç adımlı süreç, hem kernel'in zorunlu kıldığı bir gereklilik, hem de çalışan processlerin çökmesini engelleyen güvenli bir geçiş stratejisidir.

> **Kritik Fark:** `chroot`'ta eski dosya sistemi her zaman bellekte kalır ve file descriptor üzerinden erişilebilir. `pivot_root` + `umount` kombinasyonuyla eski root **tamamen kaldırılır** ve hiçbir şekilde geri dönülemez.

```bash
# Namespace'den çık
exit
```

---

## Container Runtime'lar Hangi Mekanizmayı Kullanır?

Docker, containerd ve runc gibi modern konteyner runtime'ları dosya sistemi izolasyonu için **`pivot_root`** kullanır. Süreç şu adımlarla işler:

```
KONTEYNER BAŞLATMA SÜRECİ (runc)
──────────────────────────────────────────────────────────────────

1. Container image'inden rootfs hazırla
   └─► /var/lib/docker/overlay2/<id>/merged/

2. Yeni mount namespace oluştur
   └─► unshare(CLONE_NEWNS)

3. rootfs'i mount noktası yap
   └─► mount --bind rootfs rootfs

4. pivot_root uygula
   └─► pivot_root(rootfs, rootfs/.old_root)

5. Eski root'u umount et
   └─► umount -l /.old_root
   └─► rmdir /.old_root

6. Konteyner process'ini başlat
   └─► exec /entrypoint.sh

SONUÇ: Process artık SADECE container image içindeki
       dosya sistemini görür. Host dosya sistemine
       erişim tamamen kesilmiştir.
```

### Neden pivot_root Tercih Edilir?

1. **Tam izolasyon**: Eski root umount edildiğinde host dosya sistemine erişim fiziksel olarak kesilir
2. **Escape riski yok**: `chroot` escape teknikleri (`fchdir`, `mknod`) `pivot_root` + `umount` kombinasyonunda çalışmaz
3. **Mount namespace ile birlikte çalışır**: Konteynerın mount tablosu host'tan tamamen bağımsızdır
4. **Kernel seviyesinde güvenlik**: Mount hiyerarşisi değiştirildiğinden, yol manipülasyonları etkisizdir

---

## Özet Karşılaştırma

```
chroot                                  pivot_root
──────────────────────                   ──────────────────────
+ Basit ve hızlı                        + Tam dosya sistemi izolasyonu
+ Root yetkisi yeterli                  + Eski root umount edilebilir
- Escape edilebilir (fd, mknod)         + Container runtime standardı
- Eski root erişilebilir kalır          + Mount namespace ile çalışır
- Sadece yol çözümleme değişir          - Mount namespace gerektirir
- Konteyner güvenliği için yetersiz     - Kurulumu daha karmaşık

Kullanım Alanı:                         Kullanım Alanı:
• Build ortamları                       • Docker, containerd, runc
• Paket derleme (debootstrap)           • Konteyner dosya sistemi izolasyonu
• Kurtarma modu (rescue boot)           • Güvenli sandbox ortamları
```

> **Sonuç:** Konteyner teknolojilerinde dosya sistemi izolasyonu `pivot_root` + `umount` kombinasyonuyla sağlanır. `chroot` ise daha basit senaryolarda (build ortamları, rescue mode) tercih edilir. Kubernetes'te çalışan her konteyner, arka planda bu mekanizma sayesinde host dosya sisteminden tamamen izole bir ortamda çalışır.
