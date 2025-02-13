# Secret Kullanımı

Secret, parola, key, token gibi hassas bilgileri _cluster_ içerisinde saklamamıza yarayan bir Kubernetes objesidir. Örneğin bir uygulamamızın kullanacağı veri tabanı bağlantı bilgileri, bir API Key gibi bilgiler cluster'da bu şekilde saklanmalıdır.

Kullanımı ve görünüşü temelde Configmap'e çok benzese de farklılaştığı üç temel nokta vardır:

1. Configmap'in aksine, __namespace dışından erişilemezler.__
2. İçerisindeki veriyi base64 encoded olarak saklar, herhangi bir şeyi saklamaya imkan tanır (binary vs.)
3. Farklı secret tipleri vardır

Aşağıda `kubectl` ile bir secret oluşturacağız. `kubectl` ile secret oluşturmanın çok çeşitli yolları var. En çok kullanılanlar dosyadan oluşturmak ve doğrudan değerlerle oluşturmak.

İlk örnekte key-value şeklinde bir secret objesi oluşturalım:

```bash
kubectl create secret -n default generic my-secret --from-literal=password=abc123 --from-literal=username=user1
```
```bash
secret/my-secret created
```
Bu komutla kullanıcı adı "user1" olan ve parolası "abc123" olan, "my-secret" isminde bir Kubernetes _secret objesi_ oluşturuyoruz.

>  `kubectl`'i dry-run olarak çalıştırmak için parametre sonunda `--dry-run=client` ekleyebilirsiniz.

Benzer şekilde, secret objelerini bir dosyadan da oluşturabiliriz.

```bash
echo "my-secret-password" > secret.txt
kubectl create secret -n default generic file-secret --from-file=secret.txt
```
İlk komut ile secret.txt isimli bir dosyaya "my-secret-password" yazdık ve ikinci komutla da bu dosyanın içeriğinden bir secret objesi oluşturduk.

Şimdi oluşturduğumuz secret objesini görüntüleyelim:
```bash
kubectl get secrets -n default
```
```bash
NAME          TYPE     DATA   AGE
file-secret   Opaque   1      3m25s
my-secret     Opaque   2      2m39s
```
Bir tanesi `file-secret` öbürü de ilk oluşturduğumuz `my-secret` objeleri. Fakat bu halde pek bir şey ifade etmiyor.
Bunların detaylarını `json` veya `yaml` olarak görüntüleyebiliriz.
```bash
kubectl get secret -n default file-secret -o yaml
```
> `kubectl` kullanırken bir objenin çıktısını belirli bir formatta almak için `-o` parametresini kullanabilirsiniz. `-o yaml` veya `-o json` gibi.
```yaml
apiVersion: v1
data:
  secret.txt: bXktc2VjcmV0LXBhc3N3b3JkCg==
kind: Secret
metadata:
  creationTimestamp: "2025-02-09T20:41:36Z"
  name: file-secret
  namespace: default
  resourceVersion: "199568"
  uid: 47110152-2b9f-4408-a1bc-25c479624ed8
type: Opaque
```
Burada secret.txt içerisine yazdığımız "my-secret-password"ün base64 encoded halini görüyoruz. Bunu herhangi bir şekilde decode edebilirsiniz.

Dosya olarak uygulamak içinse:
```bash
kubectl apply -f secret.yaml
```
> Secret'ı YAML halde oluştururken değerleri base64 encode edip girmek istemiyorsanız `data` scopeunu `stringData` olarak değiştirebilirsiniz. (örnek: `stringData.password: abc123`)

Eğer bilgisayarınızda `yq` veya `jq` yüklüyse şu şekilde parse edebilirsiniz:
```bash
kubectl get secret -n default file-secret -o yaml | yq '.data."secret.txt"' | base64 -d

## veya
kubectl get secret -n default file-secret -o json | jq -r '.data["secret.txt"]' | base64 -d
```
Bu komut size doğrudan decoded base64 değeri verecek, yazılan metni okumanızı sağlayacaktır.
> Ne olduğunu bilmediğiniz base64 kodları terminalinizde çalıştırmayın.

## Secret Tipleri
Secretları kullanırken, "type" alanında Secret'ın tipini belirtebiliriz:

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: my-secret
  namespace: default
data:
  password: YWJjMTIz
  username: dXNlcjE=
```

| Tip                                   | Kullanım                                   |
| ------------------------------------- |---------------------------------------- |
| `Opaque`                              | Kullanıcı tanımlı veri            |
| `kubernetes.io/service-account-token` | ServiceAccount token                    |
| `kubernetes.io/dockercfg`             | serialized `~/.dockercfg` dosyası          |
| `kubernetes.io/dockerconfigjson`      | serialized `~/.docker/config.json` dosyası |
| `kubernetes.io/basic-auth`            | basic-auth kimlik doğrulaması için    |
| `kubernetes.io/ssh-auth`              | SSH kimlik doğrulaması için   |
| `kubernetes.io/tls`                   | Server veya client TLS sertifikası         |
| `bootstrap.kubernetes.io/token`       | bootstrap token data                    |

Secret tipleri, nerede nasıl kullanmak istediğinize göre değişkenlik gösterebilir. Her birisi için usecase ve kullanımlarını anlatan bir sayfayı da aşağıda bulabilirsiniz.
#### [Secret Tipleri Örnekleri](secret_ornek.yaml)

## Deployment'ta secret kullanımı
Secretları oluşturduktan kullanabilmek için bunları Podlara bağlamamız gerekiyor.
Pod'un secret'a erişilebilmesi için ya bir _environment variable_ olarak eklememiz ya da bir dosya olarak bağlamamız gerekiyor.

Tabii içerideki uygulamanın secretları kullanım mekanizması da önemli. Bazı uygulamalar environment variable olarak tercih ederken bazıları dosya olarak tercih ediyor. İki örneği de yapalım.

İki örnekte de loglarda verdiğimiz değerleri görebileceğiz.

Environment variable örneği:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: env-secret-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: env-secret-app
  template:
    metadata:
      labels:
        app: env-secret-app
    spec:
      containers:
        - name: myapp
          image: busybox
          command: ["sh", "-c", "env && sleep 3600"]
          envFrom:
            - secretRef:
                name: my-secret
          resources: {}
```

Volume olarak bağlanacaksa da bu şekilde kullanılabilir. Sertifikaları, userlist.txt gibi dosyaları bu şekilde bağlayabilirsiniz:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: volume-secret-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: volume-secret-app
  template:
    metadata:
      labels:
        app: volume-secret-app
    spec:
      containers:
        - name: myapp
          image: busybox
          command:
            [
              "sh",
              "-c",
              "echo 'Mounted Secret:' && ls -la /etc/secret-volume && cat /etc/secret-volume/secret.txt && sleep 3600",
            ]
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/secret-volume
              readOnly: true
          resources: {}
      volumes:
        - name: secret-volume
          secret:
            secretName: file-secret
```


Şimdi bu deploymentların çıktılarını alalım.
```bash
kubectl get pods -n default

NAME                                        READY   STATUS    RESTARTS   AGE
env-secret-deployment-687b8f4b77-4q6xc      1/1     Running   0          5m16s
volume-secret-deployment-5596f86499-6gl6f   1/1     Running   0          5m13s
```
Şimdi loglarına bakalım:
```bash
kubectl logs -n default volume-secret-deployment-5596f86499-6gl6f


Mounted Secret:
total 4
drwxrwxrwt    3 root     root           100 Feb 13 20:20 .
drwxr-xr-x    1 root     root          4096 Feb 13 20:20 ..
drwxr-xr-x    2 root     root            60 Feb 13 20:20 ..2025_02_13_20_20_21.3116789438
lrwxrwxrwx    1 root     root            32 Feb 13 20:20 ..data -> ..2025_02_13_20_20_21.3116789438
lrwxrwxrwx    1 root     root            17 Feb 13 20:20 secret.txt -> ..data/secret.txt
my-secret-password
```
En altta gördüğümüz gibi __my-secret-password__ görünür halde, base64 decoded şekilde bağlanmış.

Env variable olana bakalım:
```bash
kubectl logs -n default env-secret-deployment-687b8f4b77-4q6xc 


KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.152.183.1:443
HOSTNAME=env-secret-deployment-687b8f4b77-4q6xc
SHLVL=1
username=user1
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.152.183.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
password=abc123
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.152.183.1:443
KUBERNETES_SERVICE_HOST=10.152.183.1
```
Burada da __username__ ve __password__ değerlerinin, secret'tan okunup, pod içerisinde base64 decoded olarak env variable olarak tanımlandığını görebiliyoruz.