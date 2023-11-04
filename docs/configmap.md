# ConfigMap Kullanımı

Bir uygulamaya çeşitli konfigürasyon değerleri geçmemiz gerekebilir. Bu noktada ConfigMap Kubernetes objesi ile konfigürasyon değerlerimizi Kubernetes üzerinde saklayabiliriz.

Bu konfigürasyon değerlerini tutan ConfigMap objelerini Deployment veya Pod gibi objeler ile uygulamamıza geçebiliriz.

Aşağıdaki örnekte `kubectl` kullanarak key-value bir ConfigMap objesi oluşturup daha sonrasında bunu Deployment objesine bağlayacağız.

```console
kubectl create configmap --from-literal=myKEY=myValue app-config
```

Oluşan ConfigMap objemizin değerine Kubernetes üzerinden bakalım:

```console
kubectl get configmap -n default app-config -o yaml`
```

```yaml
apiVersion: v1
data:
  myKEY: myValue
kind: ConfigMap
metadata:
  creationTimestamp: "2023-11-03T19:56:15Z"
  name: app-config
  namespace: default
  resourceVersion: "4771480"
  uid: 6cdfd7be-a546-4cd1-8727-30b9f70a0f63
```

## Deployment Objesine ConfigMap Mount Etmek

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "3"
  labels:
    app: nginx
  name: nginx
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /tmp/configs
          name: api-config
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: app-config
        name: api-config
```

Yukarıdaki Deployment objesinde iki önemli kısım bulunuyor. Birinci kısım Deployment objesine volume mount eklediğimiz

```yaml
volumes:
- configMap:
    defaultMode: 420
    name: app-config
  name: api-config
```

bölümü.

Diğer kısım ise ilgili volume tanımını container üzerinde bağlama işlemi:

```yaml
volumeMounts:
- mountPath: /tmp/configs
  name: api-config
```

Deployment objemize ait bir Pod objesine girip `/tmp/configs` yoluna gittiğimizde aşağıdaki şekilde ConfigMap objemizin içerisindeki key değerlerinin dosya olarak oluştuğunu görüyoruz:

```console
root@nginx-d984f49d7-cjblq:/# ls /tmp/configs/
myKEY
```

```console
root@nginx-d984f49d7-cjblq:/# cat /tmp/configs/myKEY
myValue
```


## ConfigMap Değerinin Güncellenmesi Senaryosu

Eğer hali hazırda bir Pod objesine bağlanan bir ConfigMap objesi güncellenir ise, bu değişiklik Pod dosya sistemine yansır.

Az önceki örnekte sürekli olarak myKEY değerini ekrana yazdıran bir komut yazalım ve kodu çalıştırdıktan sonra ConfigMap objemizde myKEY değerini newValue olarak güncelleyelim ve komutun çıktısına bakalım:

```console

root@nginx-d984f49d7-cjblq:/# while true; do cat /tmp/configs/myKEY; sleep 1; done;

myValue
myValue
myValue
myValue
myValue
myValue
myValue
myValue
myValue
myValue
myValue
newValue
newValue
newValue
newValue
newValue
newValue
newValue
newValue
```

Çıktıda ConfigMap objesindeki değer değişikliğinin Pod dosya sistemine yansıdığını görebiliyoruz.

*Not:* Değişiklik sadece Pod dosya sistemine yansır. Eğer sizin uygulama içinde de değişiklik olmasını istiyorsanız, konfigürasyon dosyasını izleyen bir kod yazmanız veya dosya değişikliklerini dinleyen bir kütüphane kullanmanız gerekebilir.
