# Admission Webhook ile Validasyon/Mutasyon

Kubernetes Admission Webhook, Kubernetes API Server'a gönderilen istekleri (örneğin Deployment, Service, Pod vb.) inceleme ve bu isteklerin geçerli olup olmadığını doğrulama işlemini yapar. Bu sayede, Kubernetes API Server'a geçersiz veya güvensiz isteklerin gönderilmesi önlenir.

Admission Webhook, iki farklı işleme uygulanabilir:

 * Validasyon: Bu işlem, Kubernetes API Server'a gönderilen isteklerin geçerliliğini doğrular. Örneğin, bir Deployment objesi oluşturulurken, Deployment objesi için gerekli olan bazı alanların doldurulup doldurulmadığını kontrol eder. Eğer alanlar doldurulmamışsa, Kubernetes API Server isteği reddeder ve bir hata mesajı gönderir.

 * Mutasyon: Bu işlem, Kubernetes API Server'a gönderilen istekleri değiştirir. Örneğin, bir Deployment objesi oluşturulurken, Deployment objesi için gerekli olan bazı alanların otomatik olarak doldurulmasını sağlar. Bu sayede, Deployment objesi oluşturulurken zorunlu olan alanların doldurulması garanti edilir.

Admission Webhook, Kubernetes API Server'a gönderilen isteklerin geçerliliğini doğrulama ve bu istekleri değiştirme işlemlerini yaparak, Kubernetes API Server'ın güvenliğini sağlar. Bu sayede, Kubernetes API Server'a geçersiz veya güvensiz isteklerin gönderilmesi önlenir.


Admission Webhook default olarak çalışan bir hizmet değildir sizin ayarladığınız bir özelliktir. 
 
## Admission Webhook örneği

 - Öncelikle Admission Webhook'un kullanılacağı bir namespace oluşturun. Örneğin, "admission-webhook" isimli bir namespace oluşturabilirsiniz.

 - Webhook'un çalışacağı bir deployment oluşturun. Bu deployment, webhook'un çalıştırılacağı bir pod'ı içerecektir. Pod'ın yapılandırılması için bir Deployment yaml dosyası oluşturun ve pod'ın gerekli image'ını ve enviroment değişkenlerini belirtin.

 - Bir Service oluşturun ve bu service'i webhook pod'ına bağlayın. Service, webhook'a erişim için gerekli olan bir LoadBalancer veya NodePort olabilir.

 - Bir ValidatingWebhookConfiguration objesi oluşturun ve bu objeyi kullanarak webhook'un nasıl çalışacağını yapılandırın. Bu objede, webhook'un hangi nesneleri doğrulayacağını ve ne tür değişiklikleri yapacağını belirtmelisiniz.

 - ValidatingWebhookConfiguration objesini Kubernetes API Server'ına gönderin. Bu objenin içeriğine göre, Kubernetes API Server webhook'un nasıl çalışacağını yapılandırır ve webhook'u etkinleştirir.


```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: deployment-replicas-validator
webhooks:
  - name: deployment-replicas-validator.example.com
    rules:
      - apiGroups:
          - "apps"
        apiVersions:
          - "v1"
        operations:
          - "CREATE"
          - "UPDATE"
        resources:
          - "deployments"
    clientConfig:
      service:
        name: deployment-replicas-validator
        namespace: admission-webhook
      caBundle: "..."
    failurePolicy: Ignore
```

bu admission webhook çalıştıktan sonra, deployment objesi create ve update işleminde replika sayısı 2 den az ise api-server bu objeyi oluşturmayacak.

