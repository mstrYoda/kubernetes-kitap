## Secret tiplerine göre kullanım örnekleri:

### `Opaque`
Kullanıcı tanımlı herhangi bir key-value eşini tanımlamak için kullanılır:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-opaque-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

### `kubernetes.io/dockerconfigjson`
İsminden de anlaşılacağı üzere, private registrylere login olurken clusterın kullanmasını istediğimiz kullanıcı adı ve parolayı buraya yazıyoruz.
Çoğunlukla `imagePullSecret` olarak karşımıza çıkar:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-docker-config-json
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwOi8vZXhhbXBsZS5jb20iOnsidXNlcm5hbWUiOiAidXNlciIsICJwYXNzd29yZCI6ICJwYXNzIn19fQ==

```

### `kubernetes.io/dockercfg`
`kubernetes.io/dockerconfigjson`'ın eski tip olanı. Genellikle `dockerconfigjson` kullanırsınız ancak bazı eski tip repolar hala bu yöntemi kullanabiliyor.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-docker-cfg
type: kubernetes.io/dockercfg
data:
  .dockercfg: eyJodHRwOi8vZXhhbXBsZS5jb20iOnsidXNlcm5hbWUiOiAidXNlciIsICJwYXNzd29yZCI6ICJwYXNzIn19
```

### `kubernetes.io/basic-auth`
Kendi endpointinizde veya dışarıdaki bir endpointe erişirken "basic auth" yapmak istediğinizde kullanabileceğiniz secret objesi tipidir.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: myuser
  password: mypassword
```

### `kubernetes.io/ssh-auth`
Bir pod private bir Git reposundan read/write gibi faaliyetleri yürütebilmek için SSH doğrulama yöntemi kullanıyorsa ihtiyaç duyacağınız ssh private key'inizi saklamanıza yarayan secret objesi.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-ssh-auth
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

### `kubernetes.io/tls`
TLS sertifikalarını tutmaya yarayan secret tipi. Örneğin `cert-manager` kullanıyorsanız, domain için imzalanan sertifikalar bu secret tipinde tutulur.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

### `bootstrap.kubernetes.io/token`
Cluster'a yeni bir node eklediğinizde, node'un doğrulaması için kullanılan tokendır.
__Bu secret'ı genellikle elle oluşturmayız. Kubeadm zaten bunu okur, doğrulamasını yapar. Ancak `kubeadm` dışında başka bir bootstrap işlemi eklendiyse veya ekstra token ihtiyacı duyuluyorsa o zaman elle oluşturma yapılabilir__
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-abc123
type: bootstrap.kubernetes.io/token
stringData:
  token-id: abc123
  token-secret: def456
```

### `kubernetes.io/service-account-token` 
Podlar, Kubernetes API Server'a erişebilmek için `ServiceAcocunt` kullanırlar. Bu token, ServiceAccount kullanıldığında pod içerisinde otomatik olarak mount edilir, böylece uygulama Kubernetes kaynaklarını bu token ile kullanabilir.

__Bu secret'ı genellikle elle oluşturmaya gerek kalmaz, çoğu zaman otomatik olarak oluşturulacak ve Kubernetes tarafından halledilecektir__
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-service-account-token
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
```