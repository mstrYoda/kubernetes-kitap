Health-check yöntemleri başlığı altında kısaca "olsa bazı nedenlerden dolayı -bu nedenler bellek sorunu, cpu kullanımı vb. olabilir- podda çalışan uygulamanız yanıt vermemesi durumunda Liveness Probe belirlediğiniz durumlar kontrolü ışığında podu yeniden başlatır." diye bahsetmiştik. Şimdi bunu hangi configuration ile nasıl yaptığına bakacağız.

Aşağıdaki config örneğinde görüldüğü üzere sağlıklı olup olmadığını kontrol etmek için spec.containers.livenessProbe.httpGet.path yolunda belirtildiği adresin GET isteği sonucunda httpStatus 200 dönmesi beklenir. HttpStatus 200 gelmemesi durumunda hazır olmadığı düşünülür.
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
    image: myApp
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
```

Exec ile yapılabilecek kontrol örneklerinden biri bu şekildedir. Pod düzgün bir şekilde çalışır ve sorun yaşamaz. index.html silerseniz podun restart etmeye başladığını görürsünüz. Sizde uygulamanın doğru çalıştığını doğrulayacak komut ile commandı güncelleyebilirsiniz.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: myApp
    livenessProbe:
      exec:
        command:
        - cat
        - /usr/share/myApp/html/index.html
      initialDelaySeconds: 5
      periodSeconds: 5
```

TCP ile kontrolde kubelet sizin uygulamanıza belirtilen port ile bir soket açmaya çalışacaktır. Başarılı bir şekilde bağlantı kurabiliyorsa pod sağlıklı, kuramazsa arızalı kabul edilir.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-tcp
spec:
  containers:
  - name: liveness
    image: myApp
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

gRPC örneği de bu şekilde oluşturulabilir. Port olması health-check yöntemleri başlığı altında belirtildiği gibi zorunludur.
```
apiVersion: v1
kind: Pod
metadata:
  name: etcd-with-grpc
spec:
  containers:
  - name: etcd
    image: myApp
    command: ["/agnhost", "grpc-health-checking"]
    ports:
    - containerPort: 8080
    livenessProbe:
      grpc:
        port: 8080
      initialDelaySeconds: 10
```