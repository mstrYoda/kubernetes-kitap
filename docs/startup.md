Bazen, ilk ayağa kaldırılmaya çalışıldıklarında ek bir başlatma süresi gerektirebilecek eski uygulamalarımız olabilir. Bu tür durumlarda, liveness probe parametrelerini, böyle bir durumu düşünerek ve yaşam döngüsünde hızlı tepkiden ödün vermeden ayarlamak zor olabilir. İşin püf noktası, aynı komutla, http, tcp veya exec kontrolüyle, failureThreshold * periodSeconds daha kötü durumdaki başlatma süresini kapsayacak kadar uzun bir Startup Probe kurmaktır.

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
-Önemli bir not; yukarıdaki örnekte kurgulanan en başta tanımlı ports tanımı gRPC de desteklenmiyor.-

Startup probe sayesinde, uygulamanın başlangıcını bitirmesi için maksimum 5 dakika (30 * 10 = 300s) olacaktır. Startup probe bir kez başarılı olduktan sonra, liveness probe container yaşam döngüsünde hızlı bir yanıt sağlamak için devreye girer. Startup probe hiçbir zaman başarılı olmazsa, container 300 saniye sonra öldürülür ve yeniden başlatma politikası uygulanır.