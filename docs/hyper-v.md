# Hyper-V Üzerinde K8S Kümesi(Cluster)
<!--<details>
<summary><b>İçindekiler</b> (Tıklayınız)</summary>-->

1. [Ön Bilgi](#onbilgi)
2. [Hyper-V Nedir?](#hyper-v)
3. [Gereksinimler](#gereksinimler)
4. [Kurulum Özet](#kurulumozet)
5. [Hyper-V Üzerinde Sunucuların Ayarlanması](#hyper-vsunucuayarlari)
   * 5.1 [Ana Düğüm Ayarları](#anadugumayarlari)
   * 5.2 [İşçi Düğüm Ayarları](#iscidugumayarlari)
6. [Küme Kurulumu](#kumekurulumu)
   * 6.1 [Sunucu Paket Güncellemeleri](#sunucupaket)
   * 6.2 [Docker Kurulumu](#dockerkurulumu)
   * 6.3 [Kubectl ve Kubeadm Kurulumu](#kubectlkubeadmkurulumu)
   * 6.4 [Ana Sunucu Ayarlamaları](#anasunucuayarlari)
     * 6.4.1 [IP Bloğu Ayarlanması](#ipbloguayarlanmasi)
     * 6.4.2 [Kubeconfig Dosya Yolu Değişikliği](#kubeconfig)
   * 6.5 [İşçi Sunucu Ayarlamaları](#iscisunucuayarlari)
     * 6.4.1 [İşçi Sunucunun Ana Sunucu'ya Katılması](#iscisunucuanasunucu)
   * 6.5 [Koza(Pod) Ağ Yapısı Oluşturulması](#kozapodagayarlari)
6. [Küme Kurulumu Sonrası Dağıtım Örneği](#dagitimornegi)
   * 7.1 [Nginx Dağıtımı](#nginxdagitimi)

      
<!--</details>-->

# Ön Bilgi <a name="onbilgi"></a>

K8s küme kurulumu anlatımında ingilizce terimlerin türkçe terimleri ile anlatım yapıldı. İnternette farklı kaynak ararken zorlanılmaması için türkçe olarak kullanılan terimlerin ingilizce karşılıkları aşağıda verilmiştir.

- Küme  = Cluster
- Düğüm = Node
- Ana   = Master
- İşçi  = Worker
- Koza  = Pod

# Hyper-V Nedir? <a name="hyper-v"></a>

Hyper-V, Microsoft Hyper-V, Viridian kod adındaki ve önceleri Windows Sunucu Sanallaştırma olarak bilinen, x64 bilgisayarlar için hypervisor tabanlı bir sanallaştırma sistemidir. Birden fazla sunucu rolünü tek bir fiziksel ana makinede çalışan ayrı sanal makineler olarak birleştirerek sunucu donanımı yatırımlarını iyileştirmek için bir araç sağlar. Hyper-V ayrıca, Windows haricinde Linux gibi işletim sistemleri de dahil olmak üzere birden fazla işletim sistemini verimli bir şekilde tek bir sunucuda çalıştırmak ve 64-bit bilgi işlemin gücünden faydalanmak için de kullanılabilir.Windows Server 2008'in belirli x64 sürümleriyle birlikte Hyper-V'nin bir betası sevk edilmiş ve kesinleşmiş sürüm 26 Haziran 2008'de piyasaya çıkmıştır. Yeni çıkacak olan Windows Server 2012® Hyper-V® ile de birden fazla işletim sisteminin paralel olarak aynı sunucu üzerinde çalıştırılmasını sağlamaktadır.  
{Kaynak: wikipedia.com}

# Kurulum için Genel Bilgilendirme:

## Gereksinimler:<a name="gereksinimler"></a>

- Hazır Hyper-V
- Docker
- Kubectl
- Kubeadm
## Kurulum Özet<a name="kurulumozet"></a>
Cluster için 1 adet Ana ve 1 adet İşçi düğüm kuracağız. Her adım için kodlar ve kod çıktıları ekran görüntüleri ile desteklenecek. Kuruluma geçmeden önce ortamınızda Hyper-V kurulu olması gerekmektedir.

## Hyper-V Üzerinde Sunucuların Ayarlanması:<a name="hyper-vsunucuayarlari"></a>

### Ana Düğüm Ayarları<a name="anadugumayarlari"></a>

İşletim Sistemi: Ubuntu 20.04

![1 5bBGGCJnSl5IVLxi5lyVMw.png](https://miro.medium.com/max/700/1*5bBGGCJnSl5IVLxi5lyVMw.png)

### İşçi Düğüm Ayarları:<a name="iscidugumayarlari"></a>
İşletim Sistemi: Ubuntu 20.04

![1 j2mTdlZN3kbjMgog01W9fQ.png](https://miro.medium.com/max/700/1*j2mTdlZN3kbjMgog01W9fQ.png)


## 1 - Küme Kurulumu <a name="iscidugumayarlari"></a>

### 1.1 - Sunucu Paket Güncelleme <a name="sunucupaket"></a>
Her iki sunucuda da aşağıdaki kod parçacığını çalıştırarak, paketlerin güncellenmesini sağlıyoruz.

```bash
$ apt-get update -y && apt-get upgrade -y
```
Her iki sunucunuzdada aşağıdaki gibi kod çıktısı ile karşılaşmalısınız;

![1 B0oA7UlIuS1Gjctp1rRbIQ.png](https://miro.medium.com/max/700/1*B0oA7UlIuS1Gjctp1rRbIQ.png)

### 1.2 - Docker Kurulumu <a name="dockerkurulumu"></a>
Docker kurulumu için aşağıdaki komutları çalıştırarak Docker resmi komut dosyasını çalıştırarak kuruyoruz  

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

Yukarıdaki işlemleri her iki sunucunuzdada çalıştırdıktan sonra aşağıdaki komut ile servisin çalışıp çalışmadığını kontrol edebilirsiniz.

```bash
$ systemctl status docker
```

Ana Sunucu  
![1 uEnKXv0iDIuQSVOshjotYQ.png](https://miro.medium.com/max/700/1*uEnKXv0iDIuQSVOshjotYQ.png)

İşçi Sunucu  
![1 67-WmTZQ4AgLFrVJQZ0L1w.png](https://miro.medium.com/max/700/1*67-WmTZQ4AgLFrVJQZ0L1w.png)

### 1.3 - Kubectl ve Kubeadm Kurulumu <a name="kubectlkubeadmkurulumu"></a> 
Kubernetes ortamınıza bağlanmak, yönetmek kubectl'i kullanıyoruz. Kubernetes API ile haberleşerek bu işlemleri yapabiliyoruz.

Kubeadm, kubernetes kümesinin kurulması, upgrade edilmesi gibi işlevleri sağlayan Kubernetes tarafından geliştirilmiş bir araçtır.

```bash
$ sudo apt-get install -y apt-transport-https ca-certificates curl
```
```bash
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
```bash
$ echo “deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main” | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```bash
$ sudo apt-get install -y kubelet kubeadm kubectl
```
Yukarıdaki adımda “No apt package “kubeadm”, but there is a snap with that name.” hatası alırsanız, aşağıdaki komut ile çözebilirsiniz.
```bash
$ sudo apt-add-repository “deb http://apt.kubernetes.io/ kubernetes-xenial main”
```
Yukarıdaki komutu çalıştırdıktan sonra ve kurulumlarda herhangi bir hata almadığınızı varsayarak devam ediyoruz.

```bash
$ sudo apt-mark hold kubelet kubeadm kubectl
```
Sunucularınızda son durum aşağıdaki gibi olmalı;

Ana Sunucu  
![1 zDvjiuFnZvqfeTb7Qw2xvQ.png](https://miro.medium.com/max/700/1*zDvjiuFnZvqfeTb7Qw2xvQ.png)

İşçi Sunucu  
![1 67-WmTZQ4AgLFrVJQZ0L1w.png](https://miro.medium.com/max/700/1*Gd-Us6c_Pml1kXAs_jNbMg.png)

Versiyon kontrolü için aşağıdaki kodu çalıştırıyoruz;

```bash
$ kubeadm version
```
Yukarıdaki komutu çalıştırdıktan sonra aşağıdaki çıktıyı almasınız. Eğer farklı bir çıktı aldıysanız kurulum adımlarında bir hata almış olmalısınız lütfen yukarıya dönüp kontrol ediniz.

###Swap'ı kapatalım;

```bash
$ sudo swapoff -a
```
### 1.4 - Ana Sunucu Ayarları <a name="anasunucuayarlari"></a>

Önce aşağıdaki adımlar ile Ana Sunucumuzda gerekli ayarları yapıp sonrasında İşçi Sunucumuz'a geçeceğiz.

#### 1.4.1 - IP Bloğu Ayarlanması <a name="ipbloguayarlanmasi"></a>

Kubernetes kümesi için IP bloğu tanımlamamız gerekiyor. Aşağıdaki komut ile bunu sağlayabiliriz. Siz burada kendi bloğunuzu tanımlayabilirsiniz;

```bash
sudo kubeadm init — pod-network-cidr=10.244.0.0/16
```

![1 Ug88fmeq9yOdj5kDUoZCvQ.png](https://miro.medium.com/max/700/1*Ug88fmeq9yOdj5kDUoZCvQ.png)

#### 1.4.2 - Kubeconfig Dosya Yolu Değişikliği <a name="kubeconfig"></a>
Kubernetes kümemize kubectl ile (veya geliştirdiğimiz bir uygulama tarafından) erişmek istediğimizde, küme bilgilerini barındıracağımız bir yere ihtiyacımız var. Kubeconfig tam bu noktada karşımıza çıkıyor. Varsayılan olarak ~/.kube/config dosya yolunda yer alır fakat istersek kubectl'e farklı dosya yollarında kubeconfig belirtebiliriz. Aşağıdaki komut ile bunu sağlıyoruz.

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```
### 1.5 - İşçi Sunucu Ayarları <a name="iscisunucuayarlari"></a>

Ana Sunucumuzda gerekli ayarlamaları yaptık. Sıra işçi rolü üstlenen sunucumuzda gerekli ayarlamaları yapmakta. 

#### 1.5.1 - İşçi Sunucu Ana Sunucu'ya Katılması <a name="iscisunucuanasunucu"></a>

Ana sunucu kurulumunda, "1.4.1 - IP Bloğu Ayarlanması" adımında ekranınızda aşağıdaki gibi bir komut bloğu çıktısı olmalı,

```bash
kubeadm join xx.xx.xx.xx:6443 — token r3e6pp.3thxv2ze1aehvm7j \
— discovery-token-ca-cert-hash sha256:5ce2e1ba5e84ce3ca9212089aaf0f282e23a741ed3ba7dd5c6576fdede43c9e1
```

yukarıdaki kod benim geliştirme ortamıma özel sizdeki token farklı olacaktır. Bu kodu kopyalayarak işçi sunucusuna giriyoruz. Çıktı aşağıdaki gibi olmalı;

![1 GDdvSNmarzCOHelpB502AA.png](https://miro.medium.com/max/700/1*GDdvSNmarzCOHelpB502AA.png)

Ana Sunucuda aşağıdaki komutu çalıştırıp küme durumunu kontrol ediyoruz;

```bash
kubectl get nodes
```
![1 lcFBAYPXscZ1IkAqiDW2Hg.png](https://miro.medium.com/max/700/1*lcFBAYPXscZ1IkAqiDW2Hg.png)

### 1.6 Koza(Pod) Ağ Yapısı Oluşturulması <a name="kozapodagayarlari"></a>

Öncelikle Pod'un türkçeye tam olarak nasıl çevirileceğini bilmiyorum. Ben bu dökümanda Koza olarak bahsettim. Ama daha anlamlı veya tam anlamına karşılık gelen kelime olursa güncelleyebiliriz.

Aşağıdaki komut ile küme durumunu kontrol ettiğimizde "NotReady" ifadesini görmüştük.

```bash
kubectl get nodes
```

Şimdi Ana Sunucumuz üstünde ağ yapılandırmasını yapalım ve sonrasında ilk dağıtımımızı gerçekleştirelim.

```bash
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
![1 K0p5NUnDSZp90qOnxFZXYw.png](https://miro.medium.com/max/700/1*K0p5NUnDSZp90qOnxFZXYw.png)

## 2 Küme Kurulumu Sonrası Dağıtım Örneği <a name="dagitimornegi"></a>

Kubernetes kümesi kurulumu tamamlandığına göre artık ilk örneğimizi gerçekleştirebiliriz. Ben bu örnekte Nginx dağıtımı yapmak istiyorum.

### 2.1 Nginx Dağıtımı <a name="nginxdagitimi"></a>

Aşağıdaki komutları takip ederek ilk dağıtımımızı yapalım;

```bash
kubectl create deployment nginx — image=nginx
```
![1 FGdWukx2mFv27SSgEMNRJQ.png](https://miro.medium.com/max/640/1*FGdWukx2mFv27SSgEMNRJQ.png)

```bash
kubectl expose deploy nginx — port 80 — target-port 80 — type NodePort
```
![1 hBp_VUdX4fXgikJr9aAQIQ.png](https://miro.medium.com/max/700/1*hBp_VUdX4fXgikJr9aAQIQ.png)

Dağıtımı tamamladık. Şimdi bu kozanın hangi düğümde ayağa kalktığını ve durumunu kontrol edelim;

```bash
kubectl get pod -o wide
```
![1 XZmcSy1JDCEuu3joD_eiHQ.png](https://miro.medium.com/max/700/1*XZmcSy1JDCEuu3joD_eiHQ.png)

Gördüğünüz gibi bir sorun yok. Tarayıcınızdada aşağıdaki gibi bir çıktı elde edebilirsiniz;

![1 mMewo5k_TFB3VgzeKdYQJg.png](https://miro.medium.com/max/700/1*mMewo5k_TFB3VgzeKdYQJg.png)

Kubernetes kümesini oluşturduk ve ilk dağıtım gerçekleştirdik.