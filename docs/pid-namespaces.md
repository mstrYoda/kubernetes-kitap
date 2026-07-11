# Pid-Namespaces

PID namespace, en temel namespace'lerden biridir ve diğer namespace'lerle çakışma problemi olmadan aynı PID değerlerine sahip olabilmesini sağlar.

## Pid Nedir?

Pid (Process ID) numarası, Linux işletim sisteminde bir işlemin benzersiz kimlik numarasıdır.

## Pid Namespaces Özellikleri

### PID'lerin İzolasyonu:
   - Her namespace kendi içinde ayrı bir PID hiyerarşisine sahiptir.
   - Bir namespace içindeki 1 numaralı PID her zaman o namespace'in "init" sürecidir.
   - Host (global) PID namespace'inde ise 1 numaralı PID genellikle sistemin init sürecidir.



## Pid Namespaces Oluşturma

Öncelikle namespace oluşturmadan **önce** host üzerinde çalışan processlere bakalım:

```bash
# Host üzerinde processleri listeleme (namespace oluşturmadan önce)
ps aux
```

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169316 13092 ?        Ss   Jun20   0:03 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Jun20   0:00 [kthreadd]
root       312  0.0  0.1  47580  8204 ?        Ss   Jun20   0:01 /usr/lib/systemd/systemd-journald
root       573  0.0  0.0   6212  3064 ?        Ss   Jun20   0:00 /usr/sbin/cron
root       812  0.0  0.0   8540  3724 pts/0    Ss   10:15   0:00 -bash
root       950  0.0  0.0   7488  3312 pts/0    R+   10:18   0:00 ps aux
...
```

Gördüğünüz gibi host üzerinde onlarca process çalışıyor ve PID değerleri 1'den başlayarak yüzlere, binlere kadar çıkıyor.

Şimdi yeni bir PID namespace oluşturalım ve içine bir kabuk atalım:

```bash
# Yeni PID namespace oluşturma ve içine bir kabuk atma
sudo unshare --pid --fork --mount-proc /bin/bash
```

Artık yeni namespace içindeyiz. Şimdi tekrar processleri listeleyelim:

```bash
# Yeni PID namespace içinde PID'leri listeleme
ps aux
```

```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   8540  3724 pts/0    S    10:20   0:00 /bin/bash
root         2  0.0  0.0   7488  3312 pts/0    R+   10:20   0:00 ps aux
```

Aynı iki process, bakış açısına göre çok farklı görünür:

| Process | Host (root) perspektifinden PID | Namespace içinden PID |
|---------|----------------------------------|------------------------|
| `/bin/bash` | 812 | **1** |
| `ps aux` | 950 | **2** |

> **Aynı process, iki farklı kimlik.** Host, bu processleri sıradan birer process olarak yüksek PID numaralarıyla görür. Namespace içindeki bash ise kendini sistemin ilk ve tek processi sanır — tıpkı bir container'ın içinde `/bin/bash`'in PID 1 olduğu gibi.

Farka dikkat edin: Namespace içinde **sadece 2 process** var — başlattığımız `/bin/bash` kabuğu (PID 1) ve `ps aux` komutu (PID 2). Host üzerindeki yüzlerce process artık görünmüyor!

---

**`--fork`** bayrağı neden gerekli? `unshare` komutu yeni bir izole ortam (namespace) yaratır, ancak bu ortamın içinde çalışacak bir process olması gerekir. `--fork` ile oluşan **child process** yeni namespace içinde kalırken, **parent process** host namespace'inde kalır. PID namespace'inde PID değerleri 1'den başlayarak yeniden numaralandırılır. Bu sayede host'taki processlerden tamamen bağımsız yeni bir PID listesi elde etmiş oluruz.

**/proc** dizini ise ilgili namespace'e ya da host'a ait process bilgilerini tutar. Örneğin ilk `ps aux` işlemi host namespace'i içinde çalışırken oluşturulan `/proc` dizini host'a ait process bilgilerini tutarken, ikinci `ps aux` işlemi PID namespace'i içinde çalışırken oluşturulan `/proc` dizini sadece o namespace'e ait process bilgilerini tutar.


**`--mount-proc`** bayrağı neden gerekli? Bu bayrak `/proc` dizinini yeni namespace'e özel olarak yeniden mount eder. `ps aux` komutu process bilgilerini `/proc` dizininden okur. Eğer `--mount-proc` kullanmazsak, eski (host) `/proc` dizini hâlâ bağlı kalır ve namespace içinde olsak bile **tüm host processlerini** görürüz. Bu bayrak sayesinde sadece ilgili namespace'e ait processleri görürüz.

> **Önemli**: `--mount-proc` bayrağı her zaman `unshare --pid` ile birlikte kullanılmalıdır. Aksi takdirde izolasyon tam sağlanmaz.
