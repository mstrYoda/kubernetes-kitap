# Kubernetes Nedir

Kubernetes (k8s) Google tarafından open-source olarak geliştirilen, containerized uygulamalarımızı çalıştırabileceğimiz `container orchestration` alanında tercih edilen de-facto standart projedir.



## Kubernetesin Sağladıkları

- ***Service Discovery & Load Balancing***: Kubernetes bizler için içerisinde çalışan containerlara erişebileceğimiz dns tanımlamaları yapar. Aynı zamanda gelen yükü çalışan uygulamalarımıza dengeli bir şekilde dağıtabilmek için load balancing sorumluluğunu da üstlenir.

- ***Dinamik Storage Yönetimi***: Data persist etmemiz gereken uygulamalar için dinamik olarak storage mount edebilmemizi sağlar. Bu storage çalıştığımız node üzerinde local storage veya bir cloud provider tarafında kullanabileceğimiz storage olabilir.

- ***Otomatize Rollout & Rollback***: Uygulamanızın kaç instance ile çalışacağınız istediğinizi belirtebilirsiniz. Bu sayede kubernetes yeni bir deployment yaptığınız zaman bu süreci sizin için otomatize eder.

- ***Healthcheck***: Uygulamalarınız için healthcheck tanımlamaları yaparak belirttiğiniz kurallara uymayan uygulamaların çalışmamasını veya çalışma zamanında hata alması durumunda uygulamanın restart edilmesini kubernetes bizlere sağlar.

- ***Secret & Configuration Yönetimi***: Uygulama konfigürasyonlarınızı ve uygulamanız için gerekli sensitive bilgileri kubernetes üzerinde saklayabilirsiniz.