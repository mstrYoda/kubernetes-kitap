Bu bölümde, containerized bir uygulamanın sağlıklı olup olmadığını tanımlayabilmesi ve buna göre hareket edebilmesi için health check yöntemlerinin nasıl yaptığını inceleyeceğiz.

# Kubernetes Probe Nedir?

Kubernetes probları, bir konteynerin durumunu incelemek ve belli durumlara göre farklı aksiyonlar alması için bir araç olarak tanımlanabilir. Kubernetes, bir konteynerin sağlıksız olup olmadığını belirlemek için bu araştırma işlevlerini kullanır ve eğer bir sorun varsa, Kubertenes kendi kendine yapabileceği iyileştirme adımlarını uygular. Bu iyileştirme adımları konteynerin tekrardan başlatmak veya bir önceki replikaya dönmek gibi olabilir.

Kubernetes, yeni bir konteynerin düzgün bir şekilde çalışıp çalışmadığını belirlemek için sıralı bir inceleme adımları uygular. Ancak bu adımlar bittikten ve başarılı durumu bildirildikten sonra eski konteyner kaldırılacaktır.

Kubernetes, üç tür probe a sahiptir:

- Liveness Probe 
    Bağımlılıklardan bağımsız olarak podun çalıştığını varsayalım. Ancak bazı nedenlerden dolayı -bu nedenler bellek sorunu, cpu kullanımı vb. olabilir- podda çalışan uygulamanız yanıt vermemesi durumunda Liveness Probe belirlediğiniz durumlar kontrolü ışığında podu yeniden başlatır.

- Readiness Probe
    Podun trafiği kabul etmeye hazır olup olmadığını belirlemek için kullanılır. Dış bağımlılıklar başarısız olduğunda konteyner hazır değil durumuna girebilir. Örneğin: bir veritabanına bağlantı başarılı olmadığında, pod canlı olarak belirlenir, ancak bağlantı yeniden kurulana kadar trafiği kabul etmeye hazır değildir. Bu durumda trafik sorunsuz çalışan diğer podlara yönlendirilmeye çalışılır.

- Startup Probe
    Podun başlatılıp başlatılmadığını belirlemek için kullanılır. Yapılandırıldığında, canlılık ve hazırlık araştırmaları düzgün başlatılana kadar devre dışı bırakılır. Bu, yavaş başlayan konteynerlerin düzgün bir şekilde başlatılmadan önce öldürülmesini önlemek içindir. Bu prob, K8S sürüm 1.16'dan beri ve Alfa durumunda mevcuttur.
