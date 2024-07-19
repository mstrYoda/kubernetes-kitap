# Sidecar Container

Sidecar container'ları, aynı Pod içinde ana uygulama container'ı ile birlikte çalışan ikincil container'lardır. Bu container'lar, ana uygulama container'ının işlevselliğini genişletmek veya artırmak için ek hizmetler veya işlevler (örneğin, günlük kaydı, izleme, güvenlik veya veri senkronizasyonu) sağlar, ana uygulama kodunu doğrudan değiştirmeden.

Genellikle, bir Pod'da sadece bir uygulama container'ı bulunur. Örneğin, yerel bir web sunucusu gerektiren bir web uygulamanız varsa, yerel web sunucusu bir sidecar olarak kabul edilir ve web uygulamasının kendisi uygulama container'ıdır.

## Kubernetes'te Sidecar Container'ları

Kubernetes, sidecar container'larını, init container'larının özel bir durumu olarak uygular; sidecar container'ları, Pod başlatıldıktan sonra çalışmaya devam eder. Bu belgede, yalnızca Pod başlatma sırasında çalışan container'ları açıkça belirtmek için düzenli init container'ları terimi kullanılır.

Kümenizde SidecarContainers özelliği etkinleştirilmişse (bu özellik Kubernetes v1.29'dan itibaren varsayılan olarak etkindir), bir Pod'un initContainers alanında listelenen container'lar için bir restartPolicy belirleyebilirsiniz. Bu yeniden başlatılabilir sidecar container'ları, aynı pod içindeki diğer init container'larından ve ana uygulama container'larından bağımsızdır. Bu container'lar, ana uygulama container'ını ve diğer init container'larını etkilemeden başlatılabilir, durdurulabilir veya yeniden başlatılabilir.

Ayrıca, init veya sidecar container'ları olarak işaretlenmemiş birden fazla container ile bir Pod çalıştırabilirsiniz. Bu, Pod içindeki container'ların Pod'un genel olarak çalışması için gerekli olduğu, ancak hangi container'ların önce başlatılıp durdurulması gerektiğini kontrol etmenize gerek olmadığı durumlarda uygundur. Bu, container düzeyinde restartPolicy alanını desteklemeyen daha eski Kubernetes sürümlerini desteklemeniz gerektiğinde de yapılabilir.

## Uygulama Container'larından Farkları

Sidecar container'ları, aynı pod içindeki uygulama container'larının yanında çalışır. Ancak, ana uygulama mantığını yürütmezler; bunun yerine ana uygulamaya destekleyici işlevler sağlarlar.

Sidecar container'larının kendi bağımsız yaşam döngüleri vardır. Uygulama container'larından bağımsız olarak başlatılabilir, durdurulabilir ve yeniden başlatılabilirler. Bu, sidecar container'larını güncelleyebileceğiniz, ölçeklendirebileceğiniz veya bakımını yapabileceğiniz anlamına gelir, bu da ana uygulamayı etkilemez.

Sidecar container'ları, ana container ile aynı ağ ve depolama ad alanlarını paylaşır. Bu birlikte yerleştirme, onların yakın etkileşimde bulunmasına ve kaynakları paylaşmasına olanak tanır.

## Init Container'larından Farkları

Sidecar container'ları, ana container ile birlikte çalışarak işlevselliğini genişletir ve ek hizmetler sağlar.

Sidecar container'ları, ana uygulama container'ı ile eşzamanlı olarak çalışır. Pod'un yaşam döngüsü boyunca aktiftirler ve ana container'dan bağımsız olarak başlatılabilir ve durdurulabilirler. Init container'larının aksine, sidecar container'ları yaşam döngülerini kontrol etmek için probları destekler.

Sidecar container'ları, ana uygulama container'ları ile doğrudan etkileşime geçebilir, çünkü init container'ları gibi her zaman aynı ağı paylaşırlar ve isteğe bağlı olarak aynı hacimleri (dosya sistemlerini) de paylaşabilirler.

Init container'ları, ana container'lar başlamadan önce durur, bu nedenle init container'ları bir Pod'daki uygulama container'ı ile mesaj alışverişinde bulunamaz. Herhangi bir veri aktarımı tek yönlüdür (örneğin, bir init container'ı boş bir emptyDir hacmine bilgi koyabilir).

## Container'lar Arasında Kaynak Paylaşımı

Init, sidecar ve uygulama container'larının yürütme sırası göz önüne alındığında, kaynak kullanımı için aşağıdaki kurallar geçerlidir:

- Tüm init container'larında tanımlanan belirli bir kaynak isteği veya limitinin en yükseği, etkili init isteği/limiti olarak kabul edilir. Herhangi bir kaynak için limit belirtilmemişse, bu en yüksek limit olarak kabul edilir.
- Pod'un bir kaynak için etkili isteği/limiti, pod overhead'in ve aşağıdakilerin toplamıdır:
  - Tüm non-init container'larının (uygulama ve sidecar container'ları) bir kaynak için isteği/limitinin toplamı
  - Bir kaynak için etkili init isteği/limiti
- Zamanlama, etkili isteklere/limitlere göre yapılır, bu da init container'larının Pod'un yaşam süresi boyunca kullanılmayan kaynakları ayırabileceği anlamına gelir.
- Pod'un QoS (hizmet kalitesi) seviyesi, tüm init, sidecar ve uygulama container'ları için geçerli olan QoS seviyesiyle aynıdır.
- Kota ve limitler, etkili Pod isteği ve limitine göre uygulanır.
