<!-- docs/_sidebar.md -->

* Giriş
    * [Kubernetes Nedir](kubernetes-nedir.md)
    * [Kubernetes Cluster Mimarisi](cluster.md)
    * [Pod ve Container Kavramı](pod-container.md)

* Kubernetes Node Elemanları
    * [etcd](etcd.md)
    * [api-server](api-server.md)
    * [kubelet](kubelet.md)
    * [kube-proxy](kube-proxy.md)
    * [controller-manager](controller-manager.md)
    * [scheduler](scheduler.md)

* Development Ortamı Kurulumu
    * [**kind**](kind.md)
    * [**minikube**](minikube.md)
    * [**k3s**](k3s.md)
    * [**k3d**](k3d.md)
    * [**microk8s**](microk8s.md)

* Kubectl Kullanımı
    * [kubeconfig](kubeconfig.mg)
    * [Pod ve diğer resourceları listeleme](pod-listeleme.md)
    * [Standalone pod oluşturma](standalone-pod.md)
    * [Image belirterek deployment oluşturma](image-ile-deployment-olusturma.md)
    * [Yaml dosyaları ile çalışmak](yaml-dosyalari.md)

* Controllerlar
    * [Controller Kavramı Nedir](controller.md)
    * [Deployments](deployments.md)
    * [StatefulSets](statefulsets.md)
    * [DaemonSets](daemonsets.md)
    * [ReplicaSets](replicasets.md)

* Pod Genel Bakış
    * **Pod Yaşam Döngüsü**
        * [Container Statüleri](container-faz.md)
        * [Pod Durumları](pod-durum.md)
        * [Init Container](init-container.md)
    * [Pod Preset](pod-preset.md)
    * [Ephemeral Container](ephemeral-container.md)
    * **Podların Çalışacağı Nodeları Belirleme**
        * [İhtiyaç ve Örnek Senaryolar](bolum-icerigi.md)
        * [NodeSelector Alanı](taint-toleration.md)
        * [Taint ve Tolerant Kavramı](taint-toleration.md)
        * [Node Affinity ve Pod Affinity Kavramı](affinity.md)

* Uygulama Kaynaklarının Konfigürasyonu
    * [ResourceQuata Objesi](resourcequata.md)
    * [LimitRange Objesi](limitrange.md)

* Health Check İşlemleri
    * [Health Check Yöntemleri](health-check-yontemleri.md)
    * [LivenessProbe](liveness.md)
    * [ReadinessProbe](readiness.md)

* HorizontalPodAutoscaler ile Scaling
    * [Ram & Cpu Bazlı Uygulama Scale Etme](hpa.md)
    * [Custom Metrikler ile Scaling](hpa.md)

* Config ve Secret Yönetimi
    * [ConfigMap Objesi](configmap.md)
    * [Secret Objesi](secret.md)

* Networking
    * [Service Objesi ve Service Tipleri](service.md)
    * [Endpoint Objesi](endpoint.md)
    * [Ingress Objesi ve Trafik Yönlendirme](ingress.md)
    * [Podlar Arası İletişim](podlar-arasi-iletisim.md)
    * [Nodelar Arası İletişim](nodelar-arasi-iletisim.md)

* Depolama (Volume) İşlemleri
    * [Volume Objesi](volume.md)
    * [PersistentVolume](persistentvolume.md)
    * [PersistentVolumeClaim](persistentvolumeclaim.md)
    * [Storage Extensions](storage-extensions.md)

* Uygulama ve Kullanıcı Rollerinin Yönetimi
    * [RBAC Kavramı](rbac.md)
    * [ServiceAccount](serviceaccount.md)
    * [Role ve RoleBinding](role.md)
    * [ClusterRole ve ClusterRoleBinding](clusterrole.md)

* Güvenlik
    * [PodSecurityPolicy Kavramı](podsecuritypolicy.md)
    * [Güvenli Containerlar Oluşturma](guvenli-container-olusturma.md)
    * [Node Güvenliğini Sağlama](node-guvenligi.md)
    * [Open Policy Agent ile Cluster Security](opa_cluster_security.md)

* Sertifika Yönetimi
    * [Dinamik Sertifika Yönetimi - **CertManager**](certmanager.md)

* Administration İşlemleri
    * [Cluster Kurulumu](kurulum.md)
    * [Grafana ve Prometheus ile Monitoring](monitoring.md)

* Kubernetes Development - Extensibility
    * [Custom Resource Definition ve CR Yapısı](crd-cr.md)
    * [Admission Webhook ile Validasyon/Mutasyon](admissionwebhook.md)
    * [Operator Framework](operator.md)

* Harici Araçlar
    * [Uygulama Installation - **Helm**](helm.md)
    * [Terminal UI - **k9s**](k9s.md)
    * [Container Registry Aracı - **crane**](crane.md)
    * [Yaml/kubectl İşlemleri - **kapp**](kapp.md)
    * [Kubectl ile sniffer - **ksinff**](ksniff.md)
