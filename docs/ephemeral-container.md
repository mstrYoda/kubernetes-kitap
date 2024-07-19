# Ephemeral Container

## Ephemeral Container'ları Anlamak
Pod'lar, Kubernetes uygulamalarının temel yapı taşıdır. Pod'lar geçici ve değiştirilebilir olarak tasarlandığından, bir Pod oluşturulduktan sonra ona bir container ekleyemezsiniz. Bunun yerine, genellikle Pod'ları kontrollü bir şekilde siler ve değiştirirsiniz.

Ancak bazen mevcut bir Pod'un durumunu incelemek gerekebilir, örneğin, tekrarlanması zor bir hatayı gidermek için. Bu durumlarda, mevcut bir Pod'da geçici bir container çalıştırarak durumunu inceleyebilir ve rastgele komutlar çalıştırabilirsiniz.

## Ephemeral Container Nedir?
Ephemeral container'lar, kaynaklar veya yürütme için garantiler içermediklerinden ve asla otomatik olarak yeniden başlatılmayacaklarından diğer container'lardan farklıdır, bu nedenle uygulama oluşturmak için uygun değillerdir. Ephemeral container'lar, normal container'larla aynı ContainerSpec kullanılarak tanımlanır, ancak birçok alan uyumsuzdur ve ephemeral container'lar için izin verilmez.

- Ephemeral container'lar portlara sahip olamaz, bu nedenle ports, livenessProbe, readinessProbe gibi alanlara izin verilmez.
- Pod kaynak tahsisleri değiştirilemez olduğundan, kaynak ayarlamak izin verilmez.
- İzin verilen alanların tam listesi için EphemeralContainer referans dökümantasyonuna bakın.
- Ephemeral container'lar, API'deki özel ephemeralcontainers işleyicisi kullanılarak oluşturulur, bu nedenle doğrudan pod.spec'e eklenerek eklenmeleri mümkün değildir, bu yüzden kubectl edit kullanarak ephemeral container eklemek mümkün değildir.

Normal container'lar gibi, bir Pod'a ekledikten sonra ephemeral container'ı değiştiremez veya kaldıramazsınız.

Not:
Ephemeral container'lar statik pod'lar tarafından desteklenmez.

## Ephemeral Container Kullanım Alanları
Ephemeral container'lar, bir container çöktüğünde veya bir container görüntüsü hata ayıklama araçları içermediğinde kubectl exec yetersiz olduğunda etkileşimli hata ayıklama için kullanışlıdır.

Özellikle, distroless görüntüler, saldırı yüzeyini ve hata ve güvenlik açıklarına maruz kalmayı azaltan minimal container görüntüleri dağıtmanıza olanak tanır. Distroless görüntüler bir kabuk veya herhangi bir hata ayıklama aracı içermediğinden, distroless görüntüleri yalnızca kubectl exec kullanarak hata ayıklamak zordur.

Ephemeral container'ları kullanırken, diğer container'lardaki işlemleri görebilmeniz için işlem ad alanı paylaşımını etkinleştirmek faydalıdır.