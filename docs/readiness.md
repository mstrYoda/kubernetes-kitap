Podun trafiği kabul etmeye hazır olup olmadığını belirlemek için kullanılır. Dış bağımlılıklar başarısız olduğunda container hazır değil durumuna girebilir. Örneğin: bir veritabanına bağlantı başarılı olmadığında, pod canlı olarak belirlenir,ancak bağlantı yeniden kurulana kadar trafiği kabul etmeye hazır değildir. Buna nasıl karar verdiğini ve nasıl yapabileceğimizi inceleyeceğiz.

LivenessProbe sonrasında başarılı bir istek gerçekleştirip gerçekleştiremediğimizi kontrol edebilir ve buna göre uygulamamızın gerçekten sağlıklı çalıştığından emin olabilir.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: readiness-http
spec:
  containers:
  - name: liveness
    image: myApp
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        host:
        scheme: HTTP
        path: /
        httpHeaders:
        - name: Host
          value: myapplication1.com
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

Uygulamamız ve bağımlılıklarımız için doğru çalıştığını doğrulayacak bir komut ile bunu doğrulayabilir.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: readiness-exec
spec:
  containers:
  - name: liveness
    image: myApp
    ports:
    - containerPort: 8080
    readinessProbe:
      exec:
        command:
        - cat
        - /etc/myApp/configs/config.yml
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      exec:
        command:
        - cat
        - /usr/share/myApp/html/index.html
      initialDelaySeconds: 15
      periodSeconds: 20
```

Http benzer şekilde uygulamamız ve bağımlılıklarımız için doğru çalıştığını doğrulayacak bir port ile kontrol gerçekleştirebilirsiniz.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: readiness-tcp
spec:
  containers:
  - name: readiness
    image: myApp
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        host:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

Port olması health-check yöntemleri başlığı altında belirtildiği gibi zorunludur.
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
    - containerPort: 5000
    - containerPort: 8080
    readinessProbe:
      grpc:
        port: 5000
    livenessProbe:
      grpc:
        port: 8080
      initialDelaySeconds: 10
```

Readiness probe, ek olarak bir de liveness probe içerir. Container başladıktan sonra belirtilen saniye boyunca ilk liveness probe çalıştıracaktır. Başarılı olması durumunda readiness probe kontrolü gerçekleştirilir.

