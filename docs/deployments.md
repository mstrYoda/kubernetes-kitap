
# Kubernetes Deployment Yönetimi

Kubernetes'in Deployment objesi, istenilen duruma uygun olarak bir veya birden fazla pod oluşturur ve bu istenilen durumu sürekli olarak mevcut durumla karşılaştırır. Bu işlem, uygulamanın istenilen durumda kalmasını sağlar ve gerektiğinde düzeltmeler yapar.

## Deployment Nedir?

Deployment, Kubernetes'teki bir uygulamanın birden fazla kopyasını yönetir. Bu obje, oluşturulacak pod'ların özelliklerini ve sayılarını belirtir ve bu sayede uygulamanın yüksek erişilebilirlik ve güvenilirlikle çalışmasını sağlar.

## Temel Komutlar

### Deployment Oluşturma

```
kubectl create deployment <isim> --image=<görüntü_ismi> --replicas=<kopya sayısı>
```

Örnek:
```
kubectl create deployment my_deployment --image=openjdk:21-ea-19-jdk-slim-buster --replicas=2
```

### Deployment Listeleme

Deployment'ları listelemek için:
```
kubectl get deployments
```

Canlı olarak izlemek için `-w` parametresi ekleyebilirsiniz:
```
kubectl get deployments -w
```

### Pod Yönetimi

Deployment içindeki pod'lardan birini silmek isterseniz:
```
kubectl delete pods deployment-pod-1
```

Bu durumda, eksik pod otomatik olarak yeniden oluşturulur.

### Deployment Güncelleme

Deployment'daki bir imajı değiştirmek için:
```
kubectl set image deployment/<deployment_name> <container_name>=<new_image>
```

Örnek:
```
kubectl set image deployment/my_deployment openjdk=21-ea-19-jdk-slim-buster=nginx
```

### Replika Sayısını Ayarlama

Replika sayısını artırmak veya azaltmak için:
```
kubectl scale deployment <isim> --replicas=<yeni sayı>
```

### Deployment Silme

Bir deployment'ı silmek için:
```
kubectl delete deployment <isim>
```

## Yaml ile Deployment Tanımlama

Yaml dosyası kullanarak deployment tanımlayabilirsiniz. Örnek bir Yaml dosyası:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-java-application
  labels:
    app: simple-java-application
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-java-application
  template:
    metadata:
      labels:
        app: simple-java-application
    spec:
      containers:
        - name: simple-java-application
          image: nuricanozturk/java-spring-api:latest
          ports:
            - containerPort: 8080
```

### Yaml Dosyasının Bileşenleri

- **apiVersion**: Kullanılacak Kubernetes API sürümü.
- **kind**: Obje türü.
- **metadata**: İsim ve etiketler gibi meta veriler.
- **spec**: Pod oluşturma ve yönetim ayarları.
- **labels**: Objeleri tanımlamak ve gruplamak için kullandığımız key-value ikilileridir. Nesnelerin alt kümelerini düzenlememize ve seçmemize olanak tanır. Etiketler, nesnelerin oluşturulması sırasında eklenebilir ve daha sonra düzenlenebilir. Karmaşıklığı önler ve anlaşılabilirliği artırır. Örneğin, aynı takımda farklı alanlarda çalışan kişiler pod'lara belirli bir convention'a uyarak key-value ikilileri atadığında, her takımın çalıştığı pod'lar kolayca ayırt edilebilir. Kısacası, etiketler nesneleri tanımlama ve gruplama imkanı sunar. Ayrıca, objeler arasında bağ kurmaya da yarar. Örneğin, servisler ve deployment objeleri hangi pod'lar ile ilişki kuracağını etiketler sayesinde belirler.
- **selector**: Etiketlerin belirli kombinasyonlarını seçmeye yarayan bir mekanizmadır. Selector'lar, belirli etiketlere sahip nesneleri seçip bu nesneler üzerinde toplu işlemler yapmamıza imkan tanır. Bu sayede, belirli kriterlere göre gruplandırılmış nesneleri kolayca bulup yönetebiliriz.

### YAML dosyası ile deployment yönetimi

**Deployment Yaratma-Güncelleme**
```sh
kubectl apply -f deployment.yaml
```

**Deployment Silme**
```sh
kubectl delete -f deployment.yaml
```
