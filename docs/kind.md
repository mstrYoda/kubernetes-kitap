## Giriş

Kind, Docker kullanarak kubernetes kümesi oluşturmamızı sağlayan bir araçtır. Basit kullanımı sayesinde cluster kurulumu sırasında harcanan zamanı en aza indiriyor aslında. Daha da hızlı olması için buna ek olarak vagrant kullanacağız.

Yani bugün vagrant ile ayağa kaldırdığımız sunucuya docker ve kind kurarak bir kubernetes ortamı oluşturmaya çalışacağız. Öyleyse başlayalım…

## 1) Vagrantfile Hazırlama

Başlamadan önce Vagrant’ın ne olduğuna biraz değinelim. Vagrant, sanal makine oluşturmak ve oluşturulan sanal makineleri yönetmek için kullanılan oldukça kullanışlı bir araçtır. Bizde vagrant ile bir sanal ubuntu makinesi oluşturacağız. Ben aşağıdaki gibi bir Vagrantfile hazırladım ve sunucu özelliklerini minimum düzeyde tuttum. Sizde burdaki ayarları kendinize göre değiştirebilirsiniz.

```
IMAGE_NAME = "bento/ubuntu-16.04"

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
    config.vm.define "kind" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "kind"
    end
end
```
Şimdi bu Vagrantfile’ı çalıştırdığımızda bize ubuntu-16.04 sürümünde, belli bir cpu ve memory kullanan ve ip adresini belirlediğimiz bir ubuntu sunucusu ayağa kaldıracak. Terminalimizden bu Vagrantfile’ın bulunduğu path’e gidiyoruz ve sırasıyla aşağıdaki komutu yazıyoruz.


```
vagrant up 
vagrant ssh kind
```


vagrant up komutu ile bento/ubuntu-16.04 box’ını çektik ve ayağa kaldırdık daha sonrasında bu sanal sunucuya erişmek için diğer bir komut olan ‘vagrant ssh kind’ komutunu yazdık. Eğer sunucuya login olduysanız herşey yolunda gidiyor demektir 😊

## 2) Docker Kurulumu

Docker’ı hızlı bir şekilde kurmak için [bu](https://docs.docker.com/engine/install/ubuntu/) adresten kullandığınız ortama uygun olarak kurabilirsiniz. Bende docker’ın kendi kurulum talimatlarını referans alarak sizlere ubuntu için gereken adımları aşağıya sıralı olarak ekleyeceğim. Bunları adım adım vagrant ile ayağa kaldırdığınız sunucuda uygulayabilirsiniz.


```
$ sudo apt-get update -y  

$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo apt-key fingerprint 0EBFCD88

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
$ sudo apt-get update -y

$ sudo apt-get install docker-ce docker-ce-cli containerd.io -y

#Docker'ı kendi kullanıcımıza eklemek için;
$ usermod -aG docker ${USER}
```


Kurulum adımlarını yaptıktan sonra şimdi başlayıp silinecek bir test container’ı başlatalım.


```
$ docker run --rm hello-world && docker rmi hello-world
```
<noscript><img alt="Image for post" class="t u v hc aj" src="https://miro.medium.com/max/2112/1*NZF8jrHp1kCLCQY3TQAnpw.png" width="1056" height="777" srcSet="https://miro.medium.com/max/552/1*NZF8jrHp1kCLCQY3TQAnpw.png 276w, https://miro.medium.com/max/1104/1*NZF8jrHp1kCLCQY3TQAnpw.png 552w, https://miro.medium.com/max/1280/1*NZF8jrHp1kCLCQY3TQAnpw.png 640w, https://miro.medium.com/max/1400/1*NZF8jrHp1kCLCQY3TQAnpw.png 700w" sizes="700px"/></noscript>

Eğer yukarıdaki gibi bir çıktı aldıysanız docker’da başarılı bir şekilde kurulmuş demektir.

> NOT: Eğer yukarıdaki gibi bir çıktı almadıysanız bunun sebebi kurulum adımının sonunda girdiğiniz komuttan dolayı olmakta. Exit ile sunucudan çıkıp tekrar login olduktan sonra sorununuzun düzelmiş olması gerekmekte.

## 3) Kind Kurulumu

Kind’ı kurmak için [buradaki](https://kind.sigs.k8s.io/docs/user/quick-start/) github adresine gidebilirsiniz ve sizin sisteminize uygun olan kind sürümünü kurabilirsiniz. Linux kullananlar benim yaptığım işlemleri yapabilir.


```
$ kind version
```


Şimdi kind’ı kurduktan sonra `kind version` komutu ile versiyon kontrolünü yapalım. “kind v0.9.0 go1.15.2 linux/amd64” benzeri bir çıktı aldıysanız devam edelim.

## 4) Kubectl Kurulumu

Şimdi kubernetes api ile konuşmamız için gerekli olan kubectl aracını kurmamız gerekmekte. [Bu](https://kubernetes.io/docs/tasks/tools/install-kubectl/) adresten kurulumu kendi sisteminize göre yapabilirsiniz.


```
$ curl -LO "[https://storage.googleapis.com/kubernetes-release/release/$(curl](https://storage.googleapis.com/kubernetes-release/release/$(curl) -s [https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl](https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl)"
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
$ kubectl version
```


## 5) Kind ile Kubernetes Cluster Kurulumu

Başlamadan önce önemli bir konuya değinmek istiyorum. Eğer sizde benim gibi vagrant, virtualbox, vmware gibi bir ortamda sunucu ayağa kaldırıp bu sunucuya docker ve kind kurup işlemlerinizi yapıyorsanız bazı ayarlar yapmanız gerekicek(birazdan bahsedeceğim). Fakat bu şekilde değilde, bir linux kullanıyorsanız ve docker’ınız bu linux üzerindeyse veya windows kullanıyorsanız ve docker’ınız docker desktop ise işiniz gayet kolay. Peki ne bu ayarlar?

Kind docker container olarak çalışmakta ve bu yüzden biz araya label(vagrant vs) eklediğimiz zaman bu node’da ayağa kaldırdığımız uygulamalara dışardan erişemeyeceğiz doğal olarak(sunucu içinden veya container’a exec olarak erişebilirsiniz sadece). İşte bu noktada bizim cluster’ı ayağa kaldırırken hazırlamamız gereken bir config dosyası olması gerekiyor ve bu config dosyasında istediğimiz portları belirtmemiz gerekiyor. Ben uygulamalara nodePort vereceğim için bu port aralığı 30000–32767 olmalı yani bu aralıkta dışarı bir port açmam gerekiyor. Lafı uzatmadan işlemlerimize devam edelim bu sayede daha iyi anlayacaksınız. Şimdi önce local de çalışanlar için nasıl cluster kurulacağınız göstereceğim ve daha sonra vagrant, vmware vs. ortamında çalışanlar için ne yapılması gerektiğini göstereceğim.

### 5.1) Sanal Sunucu (vagrant, vmware vb.) Kullanmayanlar İçin Cluster Kurma

Eğer siz harici bir sunucu veya vagrant kullanmıyorsanız aşağıda belirtilen dosyayı oluşturmanıza gerek yok direk aşağıdaki komut ile node’u çalıştırabilirsiniz.


```
$ kind create cluster --name NODE-ISMI
```


Yukarıdaki komutu çalıştırarak işleminize devam edebilirsiniz `— name` parametresi zorunlu değil. Eğer eklemezseniz node ismi default olarak kind olarak ayarlanacaktır.
İlk başta image’i çekmesi, internet hızınıza bağlı olmakla beraber biraz zaman alabilir(sonraki cluster kurulumları çok hızlı bir şekilde işleniyor). Daha sonra aşağıdaki gibi bir ekran görmeniz gerekiyor.


<noscript><img alt="Image for post" class="t u v hc aj" src="https://miro.medium.com/max/1506/1*X4P3LQ0kqPGa07EeqTLuZw.png" width="753" height="367" srcSet="https://miro.medium.com/max/552/1*X4P3LQ0kqPGa07EeqTLuZw.png 276w, https://miro.medium.com/max/1104/1*X4P3LQ0kqPGa07EeqTLuZw.png 552w, https://miro.medium.com/max/1280/1*X4P3LQ0kqPGa07EeqTLuZw.png 640w, https://miro.medium.com/max/1400/1*X4P3LQ0kqPGa07EeqTLuZw.png 700w" sizes="700px"/></noscript>

Eğer böyle bir ekran görüyorsanız tebrikler node’unuz hazır. Şimdi node’umuzu görmek için aşağıdaki komutu yazıyoruz ve karşımıza node’umuz ile ilgili bir kaç bilgi çıkmakta.


```
$ kubectl get node
```


Evet artık bir uygulama ile test edebiliriz. Aşağıda örnek bir deployment.yaml dosyası bulunmakta dilerseniz bu deployment ile node’unuzu test edebilirsiniz. **(siz config adımını geçebilirsiniz.)**

### 5.2) Sanal Sunucu (vagrant, vmware vb.) Kullananlar için Cluster Kurulumu

Şimdi yukarda da bahsettiğim gibi siz vagrant, vmware vs kullanıyorsanız öncelikle kindconfig.yaml adında bir dosya oluşturmalısınız ve aşağıda belirttiğim gibi ayarlamalısınız.

Burda yaptığımız işlem aslında şu; master node’umuzda 30080 portunu açmasını söylüyoruz böylece node içerisinde 30080 portunda çalışan bir uygulama olduğu takdirde dışardan bu uygulamaya erişebileceğiz. Hemen test bir deployment üzerinde deneyelim.

Şimdi bu deployment.yaml dosyası ile bir deployment ve service oluşturuyoruz. Basitçe bir nginx çalıştırıyoruz aslında. Service kısmında gördüğünüz üzere bu service’in type’ı olarak NodePort kullanıyorum ve nodePort olarak da 30080 portunu verdim. Hatırlarsanız kindconfig.yaml dosyasında da bu portu kullanmıştık. Şimdi uygulamamızı ayağa kaldıralım bunun için;


```
$ kubectl apply -f deployment.yaml
```



<noscript><img alt="Image for post" class="t u v hc aj" src="https://miro.medium.com/max/2206/1*CrBifTqE-iwUZ0952M0D2w.png" width="1103" height="251" srcSet="https://miro.medium.com/max/552/1*CrBifTqE-iwUZ0952M0D2w.png 276w, https://miro.medium.com/max/1104/1*CrBifTqE-iwUZ0952M0D2w.png 552w, https://miro.medium.com/max/1280/1*CrBifTqE-iwUZ0952M0D2w.png 640w, https://miro.medium.com/max/1400/1*CrBifTqE-iwUZ0952M0D2w.png 700w" sizes="700px"/></noscript>

Yukarıdaki resimdeki gibi pod’unuzun STATUS değeri Running ise başarılı bir şekilde çalıştığı anlamına geliyor. Şimdi bunu test etmek için local bilgisayarınızdan herhangi bir tarayıcı açın ve adres alanına`192.168.50.10:30080` (kendi ip adresinize göre) değerini girin ve enter’a basın.


<noscript><img alt="Image for post" class="t u v hc aj" src="https://miro.medium.com/max/2304/1*wsZ8rfgdha7hZy6BUoAgNA.png" width="1152" height="417" srcSet="https://miro.medium.com/max/552/1*wsZ8rfgdha7hZy6BUoAgNA.png 276w, https://miro.medium.com/max/1104/1*wsZ8rfgdha7hZy6BUoAgNA.png 552w, https://miro.medium.com/max/1280/1*wsZ8rfgdha7hZy6BUoAgNA.png 640w, https://miro.medium.com/max/1400/1*wsZ8rfgdha7hZy6BUoAgNA.png 700w" sizes="700px"/></noscript>

Yukarıdaki gibi bir ekran sizi karşılıyorsa uygulamanız ve node’unuz başarılı bir şekilde çalışıyor demektir. İşiniz bittiğinde cluster’ı silmek için aşağıdaki komutu kullanmanız gerekiyor.


```
$ kind delete clusters [NODE-NAME]
```


Daha sonra tekrar başlatmak için yukarda oluşturduğumuz komutu girmeniz yeterli olacaktır. Image docker’da hazır olduğu için cluster’ın kurulması çok kısa sürecektir.

> **NOT:** Unutmayın silip tekrar cluster oluştururken bir sanal sunucu kullanıyorsanız mutlaka `— config`parametresini kullanmanız gerekiyor. Biz tek port açmıştık fakat siz daha fazla port açabilirsiniz ve bu şekilde uygulamalara erişebilirsiniz. NodePort kullanmak istemezseniz eğer bunun dışında bir çok yol bulunmakta, bunlardan biriside `_kubectl port-forward_` ,dilerseniz [bu](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) adreesten daha fazla bilgi alabilirsiniz. kullanabilirsiniz.

Kind bence hızlı bir şekilde cluster ortamı kurabildiği için ve hızlı bir şekilde testlerimizi yapabildiğimiz için çok kullanışlı bir araç. Fakat birçok sıkıntısı bulunmakta ve bu sıkıntılarında zaman geçtikçe giderileceğine inanıyorum. Kind ile birden fazla node ayağa kaldırabilirsiniz. Daha fazla bilgi için sizi kind’ın kendi [dökümanına](https://kind.sigs.k8s.io/) alalım.