```
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

Gördüğünüz gibi, bir TCP kontrolünün konfigürasyonu HTTP kontrolüne oldukça benzer. Bu örnekte hem readiness probe hem de liveness probe kullanılmaktadır. Konteyner başladıktan 5 saniye sonra ilk hazır olma bildirimini gönderecektir. Bu, goproxy 8080 numaralı bağlantı noktasındaki konteynera bağlanmayı deneyecektir. Araştırma başarılı olursa, bölme hazır olarak işaretlenecektir. Bu denetim her 10 saniyede bir çalıştırmaya devam edecektir.

Readiness probuna ek olarak, bu konfigürasyon bir liveness probu içerir. Konteyner başladıktan 15 saniye sonra ilk liveness probu çalıştıracaktır. Readiness probu gibi, bu da goproxy 8080 numaralı bağlantı noktasındaki konteynere bağlanmaya çalışacaktır. Liveness probu başarısız olursa, konteyner yeniden başlatılacaktır.

