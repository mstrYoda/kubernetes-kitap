Bazen, ilk başlatıldıklarında ek bir başlatma süresi gerektirebilecek eski uygulamalarla uğraşmanız gerekir. Bu tür durumlarda, liveness probu parametrelerini, böyle bir durumu düzenleyek ve kilitlenmelere hızlı tepkiden ödün vermeden ayarlamak zor olabilir. İşin püf noktası, aynı komutla, HTTP veya TCP kontrolüyle, failureThreshold * periodSeconds daha kötü durumdaki başlatma süresini kapsayacak kadar uzun bir başlangıç ​​araştırması kurmaktır.

```
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

Startup probu sayesinde, uygulamanın başlangıcını bitirmesi için maksimum 5 dakika (30 * 10 = 300s) olacaktır. Başlangıç ​​araştırması bir kez başarılı olduktan sonra, liveness probu konteyner kilitlenmelerine hızlı bir yanıt sağlamak için devreye girer. Startup probu hiçbir zaman başarılı olmazsa, konteyner 300 saniye sonra öldürülür ve yeniden başlatma politikası uygulanır.