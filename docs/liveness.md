```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

Yapılandırma dosyasında, Bölmenin tek bir konteynera sahip olduğunu görebilirsiniz. periodSeconds liveness probun 3 saniyede bir gerçekleşeceğini belirtir. initialDelaySeconds ilk probun gerçekleştirmeden önce 3 saniye beklemesi gerektiğini söyler. Konteyner da çalışan ve 8080 numaralı bağlantı noktasında dinleyen sunucuya bir HTTP GET isteği gönderir. Sunucunun /healthz yolunun işleyicisi bir başarı kodu döndürürse, konteyner canlı ve sağlıklı olarak değerlendirir. İşleyici bir hata kodu döndürürse, kubelet konteyneri öldürür ve yeniden başlatır.

200'den büyük veya eşit ve 400'den küçük herhangi bir kod başarıyı gösterir. Başka herhangi bir kod, başarısızlığı gösterir.