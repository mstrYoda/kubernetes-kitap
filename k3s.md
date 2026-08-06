# K3S

Rancher Labs tarafından geliştirilen, hafif bir Kubernetes dağıtımıdır
Daha düşük sistem kaynaklarıyla çalışmak için optimize edilmiş, Linux, ARM cihazlar, edge cihazlar ve IoT ortamlarında çalışabilen, tek komutla hızlı bir şekilde kurulabilen, 100 MB’dan küçük bir binary dosyasıdır.
Varsayılan olarak etcd yerine SQLite kullanır (isteğe bağlı olarak etcd ile de kullanılabilir).

## Gereksinimler 

K3s aşağıdaki mimariler için kullanılabilir:

- x86_64
- armhf
- arm64/aarch64
- s390x

Donanım:

| Node	| CPU	| RAM	|
|-------|-------|-------|
| Server|2 cores| 2 GB  |
| Agent	|1 core	|512 MB |

- Dökümanda yukarıdaki gereksinimler yazsada 1 CPU 512 MB RAM içeren sanal makinama server olarak yine 1 CPU 512 MB RAM içeren başka bir sanal makinama agent olarak kurulum ve testler yaptım.
- Optimum hızı sağlamak için, mümkün olduğunda bir SSD kullanmanızı öneririz. K3s'i bir Raspberry Pi veya diğer ARM cihazlarına dağıtıyorsanız, harici bir SSD kullanmanız önerilir
- Yukarıda K3S'in dökümanda yazan gereksinimler var. Daha günceli için dökümana bakabilirsiniz. Güncel gereksinimler için döküman => [link](https://docs.k3s.io/installation/requirements)


## K3S Mimarisi

![ss](/docs/images/k3sarch.svg)

## K3S vs K3D

K3D docker containerı yada containerları üzerinde K3S'i çalıştırır. K3D kullanmayacaksanız doğrudan ana makineye veya VM üzerine K3S'i kurabilirsiniz. Basit kurulumda direk `kubectl`, `crictl`, `ctr`, `k3s-killall.sh`, and `k3s-uninstall.sh` vs araçlar ve scirplerde gelmektedir

## Diğer Kubernetes dağıtımları ile kıyaslanması

![alt text](/docs/images/k3s_kiyaslama.png)

## Kurulum
 
```bash
# Server olarak kurulum :
$ curl -sfL https://get.k3s.io | sh -

# ! /var/lib/rancher/k3s/server/node-token dizininde oluşan token Agent olarak yükleme yaparken kullanılacak

# Yüklemeyi kontrol edelim, # agent yüklendikden sonra burada bir node daha oluşacak
$ k get nodes -o wide
NAME       STATUS   ROLES                  AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
tdemirs    Ready    control-plane,master   42m   v1.31.4+k3s1   192.168.56.110   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-27-amd64   containerd://1.7.23-k3s2

# Agent olarak kurulum
$ curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -


# serverdan silmek 
$ bash /usr/local/bin/k3s-uninstall.sh

# agentdan silmek
$ bash /usr/local/bin/k3s-agent-uninstall.sh
```

Farklı konfigürasyonlar ile yüklemek için => [link](https://docs.k3s.io/installation/configuration)

## Basit Bir Örnek

K3S kullanarak basit bir web uygulaması yapalım
imperative ya da declarative yöntemlerle bunu yapabiliriz. Burada declarative yani YAML dosyaları vasıtası ile kullanacağız.

- Bu basit uygulamayı bir adet node (master) üzerinde yapalım. Öncelikle master olacak şekilde Kurulum başlığındaki gibi kurulumumuzu yapalım

- Şuan ki durumu kontrol edelim

```BASH
kubectl get all # bu komutla ne var ne yok göreceğiz (pods, services, nodes vs vs)
```

- Sırasıyla deployment, service ve ingress nesneleri oluşturacağız

```YAML
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2 # bu uygulamanın kaç adet replicasi olacağını belirliyoruz. Oluşturduğumuz iki replica pod 
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx # birden fazlada container olabilirdi
        image: nginx:latest # dockerhub'dan çekilecek image
        ports:
        - containerPort: 80
```

```YAML
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
#  type: CluserIP # zaten default service nesnemiz ClusterIP tipinde
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

```YAML
# nginx-ingress.yaml
# eğer birden fazla uygulamamız olsaydı tüm bu uygulamalara ingress ile
# yönlendirme yapabilirdik. Şimdilik sadece nginx-deployment ismindeki
# uygulamamızın service nesnesine example.com isteklerini yönlendireceğiz
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
#  annotations:
#    kubernetes.io/ingress.class: "traefik" # K3S default olarak traefik kullanır
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

- Bu YAML dosyalarını sırası ile apply komutuyla hayata geçirelim

```BASH
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f nginx-ingress.yaml
```

- apply komutları sonrasında tekrar tüm durumu görüntüleyelim.

```BASH
kubectl get all
```

![alt text](/docs/images/k3s_example_image.png)

- Şimdi ana makinemizde /etc/hosts dosyasında node ip ile example.com'u eşleştirelim. `kubectl get nodes -o wide` komutu ile node ip yi öğrenebiliriz.

```hosts
192.168.56.110 example.com # eklediğimiz tek satır bu, gerisine dokunmamıza gerek yok

# Standard host addresses
127.0.0.1  localhost
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
# This host address
127.0.1.1  mainmachine
```

- Artık uygulamamızı görüntüleyebiliriz. Tarayıcımızdan example.com'u açalım. Karşımızda nginx'in default sayfası

![alt text](/docs/images/k3s_example_response.png)

