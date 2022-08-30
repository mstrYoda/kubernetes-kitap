Bu bölüm, Kubernetes de containerized bir uygulamanın sağlıklı olup olmadığını tanımlayabilmesi ve buna göre hareket edebilmesi için health check yöntemlerinin nasıl yaptığını inceleyeceğiz.

# Kubernetes Probe Nedir?

Kubernetes probeları, bir containerin durumunu incelemek ve belli durumlara göre farklı aksiyonlar alması için bir araç olarak tanımlanabilir. Kubernetes, bir containerin sağlıksız olup olmadığını belirlemek için bu araştırma işlevlerini kullanır ve eğer bir sorun varsa, Kubernetes kendi kendine yapabileceği iyileştirme adımlarını uygular. Bu iyileştirme adımları containerin tekrardan başlatmak veya bir önceki replikaya dönmek gibi olabilir.

Kubernetes, yeni bir containerin düzgün bir şekilde çalışıp çalışmadığını belirlemek için sıralı bir inceleme adımları uygular. Ancak bu adımlar bittikten ve başarılı durumu bildirildikten sonra eski container kaldırılacaktır.

## Kubernetes, üç tür probe a sahiptir:
- Liveness Probe 
    Bağımlılıklardan bağımsız olarak podun çalıştığını varsayalım. Ancak bazı nedenlerden dolayı -bu nedenler bellek sorunu, cpu kullanımı vb. olabilir- podda çalışan uygulamanız yanıt vermemesi durumunda Liveness Probe belirlediğiniz durumlar kontrolü ışığında podu yeniden başlatır.

- Readiness Probe
    Podun trafiği kabul etmeye hazır olup olmadığını belirlemek için kullanılır. Dış bağımlılıklar başarısız olduğunda container hazır değil durumuna girebilir. Örneğin: bir veritabanına bağlantı başarılı olmadığında, pod canlı olarak belirlenir, ancak bağlantı yeniden kurulana kadar trafiği kabul etmeye hazır değildir. Bu durumda trafik sorunsuz çalışan diğer podlara yönlendirilmeye çalışılır.

- Startup Probe
    Podun başlatılıp başlatılmadığını belirlemek için kullanılır. Yapılandırıldığında, Liveness ve Readiness probeları düzgün başlatılana kadar devre dışı bırakılır. Bu, yavaş başlayan containerlerin düzgün bir şekilde başlatılmadan önce öldürülmesini önlemek içindir. Bu probe, K8S sürüm 1.16'dan beri ve Alfa durumunda mevcuttur.

## Bu probeların da uygulamaları incelemek için desteklediği başlıca yollar:
Bunlar http, exec, tcp ve gRPC -Kubernetes v1.23 itibaren- oluyorlar.

- http
Uygulama da /health endpointinin başarılı bir status dönmesi beklenmektedir.
HTTP Probe has additional options to configure.
    * host: Host/IP (default: pod IP)
    * scheme: İstekte kullanılacak protokol (default: HTTP)
    * path: Path
    * httpHeaders: İstekte kullanılacak bir header/value map
    * port: Bağlanılacak port
```
httpGet:
  path: /health
  port: 8080
```

- exec
"cat /usr/share/myApp/html/index.html" komutunun status u kontrol edilir. (0) ise başarılı gerçekleşmiştir.
```
exec:
  command:
  - cat
  - /usr/share/myApp/html/index.html
```

- tcp
Başarılı bir şekilde uygulamaya bu port üzerinden bağlantı sağlanması beklenmektedir.
```
tcpSocket:
  port: 8080
```

- gRPC
Uygulama da kendimiz oluşturduğumuz health service bağlantı sağlayabilmek ve duruma göre belli status dönülmesi beklenmektedir.
```
grpc:
  port: 2379
```

## Herhangi bir probe için ortak yapılandırma ayarları:
- initialDelaySeconds: Container çalışmaya başladıktan sonra "initialDelaySeconds" kadar bekledikten sonra probelar başlar. (default: 0)
- periodSeconds: Probe ne sıklıkla çalışacak bunu belirtir. (default: 10)
- timeoutSeconds: Probe timeout (default: 1)
- successThreshold: Container sağlıklı/hazır diye onaylamak için probe kaç kere doğrulaması gerektiğidir. (default: 1)
- failureThreshold: "successThreshold" tam tersi olarak container sağlıklı/hazır değil diyebilmek için probe kaç kere başarısız olması gerektiğidir. (default: 3)