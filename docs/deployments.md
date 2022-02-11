# Deploymentlar

## Deployment Nedir?

Deploymentlar podların hizmetin devamlılığı ve esnekliğini gerçekleştirmesi için tanımları içerir. Deployment ReplicaSet’in yanında rolling update yapma imkanı veren ilgili pod'u ve replica sayılarını belirttiğimiz bir Kubernetes objesidir. Deployment, arka tarafta ReplicaSet kullanmaktadır. Birden fazla ReplicaSet kullanabildiği için de rolling update yapma şansına sahiptir. Aynı şekilde bir önceki versiyona rollback yapma imkanı da verir. Bir nevi Blue/Green deployment yeteneği kazandırır.
Örneğin; yeni bir versiyona geçerken bütün pod’ları öldürüp downtime yaratarak yeni versiyona sahip pod'ları ayağa kaldırmaz. Ayarlanabilen rolling update oranı ile yeni versiyon’a sahip pod'ları ayağa kaldırır ve eski versiyona sahip olan pod'ları öldürür.


Podlar kesinlikle Deployment'lar kullanılarak çalıştırılmalıdır. Kubernetes Deployment Tutorial with Example YAML


``` my-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

.metadata.name alanıyla gösterilen, nginx-deployment adlı bir deployment oluşturulur.

.spec.replicas alanıyla belirtilen üç Pod oluşturur.

.spec.selector alanı, deployment'ın hangi Pod'ları yöneteceğini nasıl bulacağını tanımlar. Bu durumda, Pod şablonunda (app: nginx) tanımlanan bir pod varsa deployment onları yönetir ve kontrol eder. Pod şablonunun kendisi kuralı karşıladığı sürece daha karmaşık seçim kuralları mümkündür.

template alanı aşağıdaki alt alanları içerir:

    Podlar,  .metadata.labels alanını kullanarak app:nginx olarak etiketlenir.
    .template.spec alanı podda çalışan konteyner bilgilerini tanımlar. Kullanılan image adı, konteyner adı, kullanılan portlar vb.


kubectl komutunu kullanarak komut satırından bir deploymentı kurabiliriz.

```
kubectl apply -f my-deployment.yaml

kubectl get deployments

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```
Yukarıdaki başlıkları incelediğimizde

NAME, deployment  adlarını listeler.
READY, uygulamanın kaç kopyası olduğunu görüntüler. ready/desired şeklinde okunur.
UP-TO-DATE, geçiş aşamasında istenen duruma ulaşmak için güncellenen kopyaların sayısını görüntüler.
AVAILABLE, kullanıcılarınız için uygulamanın kaç kopyasının mevcut olduğunu görüntüler.
AGE, uygulamanın çalıştığı süreyi görüntüler.


Deployment drumunu görmek için rollout status komutunu çalıştırıyoruz.

```
kubectl rollout status deployment/nginx-deployment

Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

Sonrasında tekrar baktığımızda uygulamamızını tüm kopyalarıyla hazır olduğunu görürüz.

```
kubectl get deployments nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
```

Her deployment kopya sayısını replicasetler ile gerçekleştirdiğinden  aşağıdaki komutla replicaset tarafından da istenilen durumu görebiliriz. 

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```

Her podun üretilen etiketlerini görmek istersek

```
kubectl get pods --show-labels

NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
```

Önemli Notlar:

* Bir deploymentta uygun bir seçici ve pod template etiketleri belirtmek zorundayız. Yoksa deployment podu bulamaz. (bu durumda app: nginx).
* Etiketleri veya seçicileri diğer denetleyicilerle (diğer deployments ve StatefulSets dahil) çakıştırmayın. Kubernetes, çakışmanızı engellemez ve birden çok denetleyicinin çakışan seçicileri varsa, bu denetleyiciler çakışabilir ve beklenmedik şekilde davranabilirler. 

## Deployment Güncelleme

```
--record ile yapılan işlemlerde komutu deployment geçmişinde görebilirsiniz. 

kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record

deployment.apps/nginx-deployment image updated
```

* varsayılan editör ile deployment tanımını açar ve buradan konteyner imajıyla ilgili ayarları değiştirebiliriz.

```
kubectl edit deployment.v1.apps/nginx-deployment
```

* Eğer deployment detaylarını görmek istersek

```
kubectl describe deployments

Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
   Containers:
    nginx:
      Image:        nginx:1.16.1
      Port:         80/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
  Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
    Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
    Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```

### Rollover (canlı güncelleme)

Deploymentta her güncelleme yaptığımızda replicaset tanımı .spec.selector tanımına bakar ,ve .spec.template tanımları bir öncekinden farklı ise eski replicaset azaltılır ve yenisi istenen sayıya getirilir ve en son eski replicaset 0 a indirilir kurumun tamamlanmış olur.  Her güncellemede yeni bir replicaset oluşturulur.

Her deployment varsayılan olarak rollingUpdate tanımını oluşturur ve aşağıdaki ön tanımlı değerleri verir.

* **maxSurge**: Eğer % 25 olarak ayarlandığında, rollingUpdate başlayınca yeni ReplicaSet hemen sistemi büyütmeye başlar ama eski ve yeni podların toplam sayısı tüm podların % 130'unu geçemez. 

* **maxUnavailable**: Güncelleme işlemi sırasında kullanılamayacak duruma gelmesine izin verdiğimiz maksimum pod sayısını belirtir. Sayı da olabilir, istenen pod sayısının yüzdesi de olabilir. Varsayılan %25'tir. 

```
kubectl get deployment nginx-deployment -o yaml | grep strategy -A 4

  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```



