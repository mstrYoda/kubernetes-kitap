# K3D Kurulumu

## K3S (Lightweight Kubernetes) 

K3S, 2019 yılında açık kaynak olarak rancher tarafından piyasaya sürüldü. K3S, 100mb’ın altında bir binary dosyası olarak tasarlanmıştır. Ayrıca sertifikalı bir Kubernetes aracıdır ve cross platform çalışabilme özelliğine sahiptir.
K3S hafifliği sayesinde çok düşük sistemlerde bile kubernetes cluster’ı kurabilmemize olanak sağlıyor. K3S önerilen sistem gereksinimleri aşagıdadır.;      
- Linux 3.10   
- 512 MB RAM (sunucu)   
- 75 MB RAM (node)    
- 200 MB disk alanı    

## Linux Kurulum:

### Gereksinimler:  

- Windows, Linux veya MacOS ortamı
- Docker
- Kubectl
- K3D

### 1.1 - Docker Kurulumu   
Docker kurulumunu hızlı bir şekilde yapmak için rancher tarafından hazırlanan script’i kullanabilirsiniz veya bu adresten docker kurulumunu uygun ortamınıza göre yapabilirsiniz.   
```bash
$ curl https://releases.rancher.com/install-docker/19.03.sh | sh
```

### 1.2 - Kubectl Kurulumu    
Kubernetes API ile haberleşmek için Kubectl kurmamız gerekiyor, bunuda aşağıdaki adımları takip ederek kolaylıkla yapabilirsiniz.
```bash
$ curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
```
```
$ chmod +x ./kubectl
```
```
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

Tüm bu işlemlerden sonra aşağıdaki komut ile kubectl versiyon kontrolünü yapabilirsiniz.

```
kubectl version
```

### 1.3 - K3D Kurulumu

K3D kurulumu basit, yapmanız gereken terminale aşağıdaki komutu girmek. Dilerseniz [bu](https://k3d.io/#installation) adresten kurulum adımlarını kendi ortamınıza göre yapabilirsiniz.

```
$ wget -q -O - https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```
![1_I0hNdBXgOgL-VI9KOCHHRA.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606601738166/aYybiM3u1.png)

Sizde yukarıdaki resimdeki gibi bir çıktı aldıysanız kurulumu başarılı bir şekilde yaptınız demektir. İsterseniz versiyon kontrolü yaparak kurulumunuzu doğrulayabilirsiniz, bunun için terminalinize aşağıdaki komutu girmeniz gerekiyor.

Versiyon kontrolü için bu komutu girin:
```
$ k3d --verison
```

## 1.4 K3D Komutları
### Cluster Komutları:
### 'k3d cluster' komutu ile:
- 'k3d cluster create [cluster name]' ile yeni bir cluster oluşturulabilir.
- 'k3d cluster delete [cluster name]' ile cluster silebilirsiniz.    
  Cluster ismi yerine -a parametresi ile toplu seçim yapabilirsiniz.
- 'k3d cluster list' ile clusterlarınızı listeleyebilirsiniz.
- 'k3d cluster start [cluster name]' ile clusterlarınızı başlatabilirisniz. '-a' parametresi burada da geçerli.
- 'k3d cluster stop [cluster name]' ile clusterlarınızı durdurabilirsiniz. '-a' parametresi kullanımı geçerlidir.

### 'k3d node' komutu ile:
- 'k3d node create [node name]' ile node oluşturabilirsiniz.
- 'k3d node delete [node name]' ile node silebilirsiniz. '-a' parametresi geçerlidir.
- 'k3d node list' ile nodelarınızı listeleyebilirsiniz.
- 'k3d node start [node name]' ile nodelarınızı başlatabilirsiniz. '-a' parametresi geçerlidir.
- 'k3d node stop [node name]' ile nodelarınızı durdurabilirsiniz. '-a' parametresi geçerlidir.

## 1.5 K3D ile Cluster Kurulumu
Artık cluster kurulumuna geçebiliriz. Terminalinize aşağıdaki resimde oluşturduğum gibi sizde kendi oluşturduğunuz name ile cluster oluşturmaya başlayabilirsiniz.

```
$ k3d cluster create [NAME]
``` 

![1_H-2eVT95AM0j-ff9a34GXA.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606601983901/Hp8F2HXiU.png)

İlk cluster kurulumunda yaklaşık 150–200mb boyutunda docker image pull etmesi gerektiği için kurulum süresi internet hızınıza göre biraz zaman alabilir ama bu image’leri silmediğiniz sürece cluster kurulumunuz çok kısa sürecektir.

![1_OJ79GPbAi2DzdK12tzGeSg.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606602003681/VK_4Us4Bb.png)

Resimdeki komut ile oluşturduğunuz cluster’ları ve durumlarını görebilirsiniz.

Bu şekilde eğer uzak bir sunucuya k3d kurulumu yapıp cluster’ı bu ortamda oluşturduysanız bir uygulama deploy ettiğiniz zaman buna erişemezsiniz. Çünkü herhangi bir port’a izin vermediğimiz için sadece cluster içinden erişime açık olacaktır. Oluşturduğunuz cluster’ı aşağıdaki komut ile silebilirsiniz. Sonraki adımda uygulamalarımıza nasıl erişebiliriz buna değineceğim.

```
$ k3d cluster delete -a --> Tüm cluster'ları siler
$ k3d cluster delete [name] --> Belirtilen cluster'ı siler
``` 

# 2. Servislere Erişmek
Kind yazımda kind’ın henüz çok yeni olduğunu bir takım sorunlarının olduğunu söylemiştim, bunlardan biriside eğer kind’ı local ortamımıza değilde uzak bir sunucuya kurduysak port açmanın meşakatli bir iş olmasıydı. Elle tek tek port açmamız gerekiyordu ve çalışan bir cluster’da yeni bir açılmamış port ihtiyacımız olduğu takdirde cluster’ı silip tekrar config dosyasını ayarlayıp ayağa kaldırmamız gerekiyordu. K3D kullanırkende öncesinde port açmamız gerekiyor fakat bunun için bize bir çok yol sunmakta, ister node-port olarak istersek de Load Balancer aracılığı ile uygulamalarımıza erişebiliyoruz.

## 2.1 K3D NodePort Ayarları
Kind yazısında da belirttiğimiz gibi, port açmak için elle tek tek portları config dosyasına girmemiz gerekiyordu ve çalışan cluster’da yeni bir pod ihtiyacımız doğduğunda cluster’ı silip tekrar ayağa kaldırmamız gerekiyordu.
K3D bu noktada Kind’dan ayrılıyor çünkü k3d ile port açmak için belli bir port aralığını açmasını söyleyebiliyoruz. Hemen aşağıdaki örnek ile anlatmaya devam edeyim.

```
$ k3d cluster create [NAME] -p "30000-30100:30000-30100@server[0]"
``` 

Yukarıdaki komutta yeni bir parametre ekledik “-p” docker kullananlar bu komutu sık sık kullanmıştır -p(publish) komutu ile host-container portu açabiliyoruz. Yukarıda da bu işlemi yaptık 30000–30100 aralığını hem host tarafında hemde container tarafında açmasını söylüyorum. (Kubernetes default node port aralığı: 30000–32767)

Şimdi yukarda yaptığımız nginx uygulamasını bu sefer nodeport açarak yapalım demo-nodeport.yaml isimli bir dosya oluşturun ve aşağıdaki kodu içerisine ekleyip kaydedin.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
  labels:
    app: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: demo
        image: nginx
---
kind: Service
apiVersion: v1
metadata:
  name: demo
spec:
  selector:
    app: demo
  type: NodePort
  ports:
  - name: node-port
    port:  80
    nodePort: 30050
``` 

Yukardaki yaml dosyası ile deployment ve service yaratıyoruz. NodePort için 30050 adresini kullanacağım ben siz de açtığınız aralıktan bir port belirleyip belirlediğiniz port üzerinden uygulamaya erişebilirsiniz. Şimdi demo-nodeport.yaml dosyasını çalıştıralım.

```
$ kubectl create -f demo-nodeport.yaml
``` 
![1_S9EdQVJCSfcP1W5gTkfhVQ.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606602334964/ZgC69K0I6.png)

Sorunsuz bir şekilde pod’unuz ve servisiniz oluşturulduysa tarayıcınızdan uygulamaya erişebilirsiniz. (IP:NODEPORT)

![1_4CJGzHLx_5AlDM6FlA-AYw.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606602348006/sIUt1Y4fd.png)


# 3. Multi Node Cluster Kurulumu
K3D ile birden fazla node’a sahip cluster da kurabilirsiniz. Biz 3 node’lu bir cluster kurulumu yapalım ve daha sonra node komutu ile çalışan cluster’a ek olarak 1 node daha ekleyelim öncelikle aşağıdaki komut ile K3D’ye 3 node’luk bir cluster kurmasını söylüyoruz.


```
$ k3d cluster create [NAME] --servers 3
``` 

Node’lar oluştuktan sonra aşağıdaki komut’lar ile node’larımızın durumunu görebiliriz.

```
[K3D İLE KONTROL ETME]
$ k3d cluster list
``` 

```
[KUBECTL İLE KONTROL ETME]
$ kubectl get node
``` 

![1_pTu5-ZDHaJIUUOWanRQlQg.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606602480490/zIc16R4Gl.png)


## 3.1 Node Ekleme
Node’larımız başarılı bir şekilde oluştu ihtiyaca göre yeni bir node eklemek istersen ne yapmamız gerekiyor peki? Çok kolay bir şekilde aşağıdaki komut ile az önce oluşturduğumuz cluster’a node ekleyebiliriz.

```
$ k3d create node new-node --cluster [CLUSTER-NAME] --role server
``` 

![1_zDySbpHAPpbWWLkPWaEvEg.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1606602516698/g_Ko8gFAa.png)
