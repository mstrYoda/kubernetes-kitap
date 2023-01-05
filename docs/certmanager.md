# Dinamik Sertifika Yönetimi 

Dinamik Sertifika Yönetimi (Dynamic Certificate Management, DCM), Kubernetes cluster'ında çalışan servisler için otomatik olarak sertifika oluşturma ve yönetme işlemlerini gerçekleştirir. Bu sayede, Kubernetes cluster'ında çalışan servisler için sertifikaların oluşturulması ve yönetilmesi işlemleri otomatik hale getirilir ve bu işlemlerin yönetimi kolaylaşır.

DCM, Kubernetes cluster'ında çalışan servisler için sertifikaları oluştururken, bu sertifikaların geçerlilik sürelerini de dikkate alır. Örneğin, bir servisin sertifikasının geçerlilik süresi dolmadan önce yeni bir sertifika oluşturulur ve bu sertifika ile servis güncellenir. Bu sayede, servislerin sertifikalarının her zaman güncel ve geçerli olması sağlanır.

DCM, Kubernetes cluster'ında çalışan servislerin sertifikalarını oluşturmak ve yönetmek için çeşitli araçlar kullanılabilir. Örneğin, Kubernetes cluster'ında çalışan servisler için sertifikaları oluşturmak ve yönetmek için kube-cert-manager, cert-manager veya Let's Encrypt gibi araçlar kullanılabilir.

 - **Cert-manager:** Cert-manager, Kubernetes'te sertifikaların oluşturulması ve yönetilmesi için kullanılan bir araçtır. Cert-manager, sertifikaların otomatik olarak oluşturulması, yönetilmesi ve yenilenmesi için kullanılır. Örneğin, sertifikaların geçerlilik süresi dolması durumunda Cert-manager, otomatik olarak yeni sertifikalar oluşturarak sistemi güncel tutar.

 - **Let's Encrypt:** Let's Encrypt, internet üzerinde ücretsiz SSL/TLS sertifikaları sunan bir kuruluştur. Kubernetes'te de Cert-manager kullanılarak Let's Encrypt üzerinden sertifikalar oluşturulabilir.