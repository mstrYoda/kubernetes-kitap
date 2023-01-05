# Custom Resource Definition ve CR Yapısı

## Custom Resource Definition
Custom Resource Definition (CRD), Kubernetes API'sine özel olarak oluşturulmuş yeni kaynak türleridir. Örneğin, bir firma, Kubernetes API'sine "Müşteri" kaynak türü eklemek isteyebilir. Bu durumda, "Müşteri" kaynak türü için bir Custom Resource Definition (CRD) oluşturulur. Bu CRD, "Müşteri" kaynak türünün nasıl oluşturulacağı, güncelleneceği ve silineceği gibi bilgileri içerir.

CRD oluşturulurken, tasarım deseni olarak "kind" ve "apiVersion" gibi özellikler belirlenir. "kind" özelliği, kaynak türünün adını belirtir. "apiVersion" özelliği ise, kaynak türünün hangi API sürümüne ait olduğunu belirtir.

Uygulamalar, CRD oluşturulan kaynak türlerine çeşitli şekillerde erişebilirler. Örneğin, bir uygulama, Kubernetes API'sine HTTP istekleri yaparak CRD oluşturulan kaynak türlerine erişebilir. Bu sayede, uygulamalar, özel kaynak türlerine ait bilgilere erişebilir ve bu bilgilere göre işlemler gerçekleştirebilirler.

## Custom Resource (CR)

Custom Resource (CR) ise, CRD oluşturulan kaynak türlerinin gerçek örnekleridir. Örneğin, "Müşteri" kaynak türü için oluşturulan bir CRD'ye ait bir CR, bir gerçek müşteri örneğini temsil eder. Uygulamalar, bu CR'leri kullanarak, özel kaynak türlerine ait bilgilere erişebilir ve bu bilgilere göre işlemler gerçekleştirebilirler.

