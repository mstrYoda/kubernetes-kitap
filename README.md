![k8s](https://raw.githubusercontent.com/mstrYoda/kubernetes-kitap/master/kubernetes.png)

# Hoş geldiniz

> Open Source severler olarak bir araya gelip, Kubernetes için Türkçe kaynak olması açısından bu kitabı hazırladık ve siz değerli okuyuculara sunduk.

Kitabın adresi: https://mstryoda.github.io/kubernetes-kitap/#/


# Konu Başlıkları

<!-- docs/_sidebar.md -->

* Giriş
    * [Kubernetes Nedir](./docs/kubernetes-nedir.md)
    * [Kubernetes Cluster Mimarisi](./docs/cluster.md)
    * [Pod ve Container Kavramı](./docs/pod-container.md)

* Kubernetes Node Elemanları
    * [Control Plane Elemanları](control-plane.md)
        * api-server
        * etcd
        * controller-manager
        * scheduler
    * [Worker Node Elemanları](data-plane.md)
        * kubelet
        * kube-proxy

* Development Ortamı Kurulumu
    * [**kind**](./docs/kind.md)
    * [**minikube**](./docs/minikube.md)
    * [**k3s**](./docs/k3s.md)
    * [**k3d**](./docs/k3d.md)
    * [**microk8s**](./docs/microk8s.md)

* Kubectl Kullanımı
    * [kubeconfig](./docs/kubeconfig.md)
    * **Kubectl İşlemleri**
        * [Kubectl Apply](./docs/kubectl-resource-islemleri?id=kubectl-apply.md)
        * [Resource Listelemek](./docs/kubectl-resource-islemleri?id=resource-listelemek.md)
        * [Resource Editlemek](./docs/kubectl-resource-islemleri?id=resource-editlemek.md)
        * [JsonPath ile Field filtreleme](./docs/kubectl-resource-islemleri?id=jsonpath-ile-field-filtreleme.md)
        * [Deployment Oluşturmak](./docs/kubectl-resource-islemleri?id=deployment-oluşturmak.md)
        * [Deployment Scale](./docs/kubectl-resource-islemleri?id=deployment-scale.md)

* Controllerlar
    * [Controller Kavramı Nedir](./docs/controller.md)
    * [Deployments](./docs/deployments.md)
    * [StatefulSets](./docs/statefulsets.md)
    * [DaemonSets](./docs/daemonsets.md)
    * [ReplicaSets](./docs/replicasets.md)

* Pod Genel Bakış
    * **Pod Yaşam Döngüsü**
        * [Container Statüleri](./docs/container-faz.md)
        * [Pod Durumları](./docs/pod-durum.md)
        * [Init Container](./docs/init-container.md)
    * [Pod Preset](./docs/pod-preset.md)
    * [Static Pod](./docs/static-pod.md)
    * [Ephemeral Container](./docs/ephemeral-container.md)
* **Podların Çalışacağı Nodeları Belirleme - Scheduling**
    * [NodeSelector Alanı](./docs/nodeselector.md)
    * [Taint ve Tolerant Kavramı](./docs/taint-toleration.md)
    * [Node Affinity ve Pod Affinity Kavramı](./docs/affinity-anti-affinity.md)

* Uygulama Kaynaklarının Konfigürasyonu
    * [ResourceQuata Objesi](./docs/resourcequata.md)
    * [LimitRange Objesi](./docs/limitrange.md)

* Health Check İşlemleri
    * [Health Check Yöntemleri](./docs/health-check-yontemleri.md)
    * [LivenessProbe](./docs/liveness.md)
    * [ReadinessProbe](./docs/readiness.md)
    * [StartupProbe](./docs/startup.md)

* HorizontalPodAutoscaler ile Scaling
    * [Ram & Cpu Bazlı Uygulama Scale Etme](./docs/hpa.md)
    * [Custom Metrikler ile Scaling](./docs/hpa.md)

* Config ve Secret Yönetimi
    * [ConfigMap Objesi](./docs/configmap.md)
    * [Secret Objesi](./docs/secret.md)

* Networking
    * [Service Objesi ve Service Tipleri](./docs/service.md)
    * [Endpoint Objesi](./docs/endpoint.md)
    * [Ingress Objesi ve Trafik Yönlendirme](./docs/ingress.md)
    * [Podlar Arası İletişim](./docs/podlar-arasi-iletisim.md)
    * [Nodelar Arası İletişim](./docs/nodelar-arasi-iletisim.md)

* Depolama (Volume) İşlemleri
    * [Volume Objesi](./docs/volume.md)
    * [PersistentVolume](./docs/persistentvolume.md)
    * [PersistentVolumeClaim](./docs/persistentvolumeclaim.md)
    * [Storage Extensions](./docs/storage-extensions.md)

* Uygulama ve Kullanıcı Rollerinin Yönetimi
    * [RBAC Kavramı](./docs/rbac.md)
    * [ServiceAccount](./docs/serviceaccount.md)
    * [Role ve RoleBinding](./docs/role.md)
    * [ClusterRole ve ClusterRoleBinding](./docs/clusterrole.md)

* Güvenlik
    * [PodSecurityPolicy Kavramı](./docs/podsecuritypolicy.md)
    * [Güvenli Containerlar Oluşturma](./docs/guvenli-container-olusturma.md)
    * [Node Güvenliğini Sağlama](./docs/node-guvenligi.md)
    * [Open Policy Agent ile Cluster Security](./docs/opa_cluster_security.md)

* Sertifika Yönetimi
    * [Dinamik Sertifika Yönetimi - **CertManager**](./docs/certmanager.md)

* Administration İşlemleri
    * [Cluster Kurulumu](./docs/kurulum.md)
    * [Grafana ve Prometheus ile Monitoring](./docs/monitoring.md)

* Kubernetes Development - Extensibility
    * [Custom Resource Definition ve CR Yapısı](./docs/crd-cr.md)
    * [Admission Webhook ile Validasyon/Mutasyon](./docs/admissionwebhook.md)
    * [Operator Framework](./docs/operator.md)

* Service Mesh - Istio
    * [Istio Kurulumu](./docs/istio-kurulum.md)
    * [Istio ile Mesh Security](./docs/istio-mesh-security.md)
    * [VirtualService RequestRouting](./docs/vs-request-routing.md)
    * [EnvoyFilter](./docs/envoy-filter.md)
    * [Istio Monitoring](./docs/istio-monitoring.md)

* Function as a Service / Serverless
    * [OpenFaaS](./docs/openfaas.md)
    * [Knative](./docs/knative.md)

* Harici Araçlar
    * [Uygulama Installation - **Helm**](./docs/helm.md)
    * [Terminal UI - **k9s**](./docs/k9s.md)
    * [Container Registry Aracı - **crane**](./docs/crane.md)
    * [Yaml/kubectl İşlemleri - **kapp**](./docs/kapp.md)
    * [Kubectl ile sniffer - **ksinff**](./docs/ksniff.md)
    * [kaniko - **kaniko**](./docs/kaniko.md)
    * [buildah - **buildah**](./docs/buildah.md)
    * [podman - **podman**](./docs/podman.md)
