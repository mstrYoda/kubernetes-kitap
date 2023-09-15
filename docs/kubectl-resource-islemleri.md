# Kubectl Apply

CI/CD pipeline'ımızdan veya local ortamımızdan deployment yapmak istediğimiz zaman kubectl apply komutunu sıkça kullanırız.

- Tek bir dosyayı apply etmek için:

`kubectl apply -f deployment.yaml`

- Bulunduğumuz path içerisindeki tüm dosyaları apply etmek için:

`kubectl apply -f .`

##### Standart Input
Bu özelliği çeşitli ihtiyaçlarda kullanabiliriz. Örneğin harici bir program tarafından generate edilen çıktıyı apply etmek istediğimizde veya bir namespace üzerinden başka bir namespace'e aynı resource'u apply etmek istediğimizde aşağıdaki komutu çalıştırabiliriz:

`kubectl apply -f -`

Örneğin bir namespacede bulunan ConfigMap resource'unu başka bir namespace'e de uygulayalım. Bu işlemi yaparken bazı metadata bilgilerini kaldırmamız gerekiyor. Önceden `--export` flagi ile bu işlemi yapabiliyorduk fakat bu flag deprecated olduğu için `jq` aracı ile ilgili fieldları kaldırabiliriz.

```
kubectl get configmap -n test-namespace -o json \
    | jq 'del(.metadata.namespace,.metadata.resourceVersion,.metadata.uid) | .metadata.creationTimestamp=null' \
    | kubectl apply -n new-namespace -f -
```

Kubectl apply ile clustera deploy ettiğimiz değişiklikleri geri almak istersek `kubectl delete -f filename.yaml` diyerek silme işlemini sağlayabiliriz.

##### Dry Run
Kubernetes ile çalışırken, uygulama ve kaynak konfigürasyonlarınızı yönetmek, önemli bir görevdir. Yanlış konfigürasyonlar veya hatalar, can sıkıcı sorunlara neden olabilir. İşte bu noktada, `kubectl dry-run` özelliği devreye girer. Bu özellik, Kubernetes kaynaklarınızı oluşturmadan veya değiştirmeden önce değişiklikleri ön izlemenize olanak tanır. 

İlk olarak, oluşturmak veya değiştirmek istediğiniz Kubernetes kaynağı için bir YAML veya JSON konfigürasyon dosyasına ihtiyacınız vardır. Bu dosya, kaynağınızın tüm özelliklerini içermelidir. Örneğin, bir pod, bir deployment veya bir servis gibi herhangi bir Kubernetes nesnesinin gereksinimlerini tanımlar.

`kubectl dry-run` komutunu kullanarak değişiklikleri ön izleyebilirsiniz. `--dry-run=client`  kullanarak komutu çalıştırın ve ardından `-f `  ile konfigürasyon dosyasını belirtin. 

İşte örnek bir komut:
```
kubectl apply --dry-run=client -f deployment.yaml
```
Özet olarak; `--dry-run=client` , konfigürasyonunuzu sunucuya iletmeksizin istemci tarafında doğrulama yapmanızı sağlar. Bu, konfigürasyonunuzun sözdizim hataları veya diğer sorunlar açısından kontrol edilmesine olanak tanır. `kubectl` , deploymentı uygulamadan önce beklenen sonucu döndürecek ve clusterda gerçekleşecek hiçbir değişiklik yapmayacaktır. Çıktıyı gözden geçirerek, kaynak konfigürasyonunuzda hatalar, uyarılar veya sorunlar olup olmadığını kontrol edebilirsiniz. Bu, herhangi bir sorunu önceden tespit ederek düzeltmenize yardımcı olur.

Not: Unutmayın ki `--dry-run` özelliği, yalnızca konfigürasyon hatalarını ve yapısal sorunları kontrol etmek için kullanışlıdır. Bu, kaynağın gerçek dünyada nasıl davranacağı veya clusterın durumu hakkında herhangi bir garanti sağlamaz.

# Diff

# Resource Listelemek

Kubectl ile herhangi bir resource'u listelemek, apply edilmiş yaml içeriğini görmek veya resource'ların sayısını öğrenmek istediğimizde kullanabileceğimiz bazı komutlar:

`kubectl get <api-resource>`

Burada `api-resource` yerine kubernetes içerisinde yer alan herhangi bir api resource'u verebiliriz.

Örneğin:

- `kubectl get pods`
- `kubectl get services`
- `kubectl get deployments`
- `kubectl get endpoints`
- `kubectl get replicasets`

Bu işlemleri yaparken namespace belirtmek istersek komutların sonuna `-n` parametresi ekleyebiliriz. 

`kubectl get pods -n test`

Eğer tüm namespaceler için listeleme yapmak istersek `--all-namespaces` veya kısaca `-A` flagini kullanabiliriz.

`kubectl get pods -A`

İlgili resource için yaml olarak output almak:

`kubectl get pods pod-name -o yaml`

# Resource Editlemek

Herhangi bir resource'u editlemek istediğimiz zaman `kubectl edit <resource> <resource-name>` komutunu kullanabiliriz.

Bu komutu çalıştırdığımızda default editor ile edit ekranı karşımıza çıkacaktır. Eğer default editoru değiştirmek istersek örneğin vscode kullanmak istediğimizde `export KUBE_EDITOR='code --wait'` komutu ile `KUBE_EDITOR` environment parametresini kullanarak değişikliği yapabiliriz.

# JsonPath ile Field Filtreleme

Kubectl ile resource fieldlarında jsonpath ile filtreleme yapabiliyoruz.

- Bir deployment ismini jsonpath ile output olarak göstermek:

`kubectl get deployments test-deployment -n test -o jsonpath='{.metadata.name}'`

- Bir deployment içerisinde yer alan tüm container imagelarını listelemek:

`kubectl get deployments test-deployment -n test -o jsonpath='{.spec.template.spec.containers[*].image}'`

- Cluster içerisindeki tüm deployment isimlerini listelemek:

`kubectl get deployments -A -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'`

- Farklı fieldları outputa eklemek:

`kubectl get deployments -o jsonpath='{range .items[*]}{.metadata.name}{" - "}{.status.availableReplicas}{"\n"}{end}'`

# Deployment Oluşturmak

Kubectl ile elimizde herhangi bir yaml dosyası olmadığında da deployment oluşturabiliriz.

`kubectl create deployment <deployment-name> --image=<image-name> -n <namespace>`

# Deployment Scale

Bir deploymentı scale up/down etmek için:

`kubectl scale deployment <deployment-name> --replicas=3`
