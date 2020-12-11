# NodeSelector Alanı

Çalıştırdığımız podları bazı ihtiyaçlara göre belirli nodelar üzerinde schedule etmek isteyebiliriz. Bunu yapabilmenin yöntemlerinden birisi de PodSpec içerisinde yer alan `nodeSelector` fieldını kullanmaktır. Bu alan key value değer alır ve genellikle bir adet key value pair ile kullanılır.

Örneğin elimizde çeşitli regionlarda bulunan nodelar olduğunu ve bir uygulamamızın spesifik regionda çalışmasını istediğimizi varsayalım.

Bunu sağlamak amacıyla nodelarımızı ilgili regionlar bazında labellamamız ve podlara aşağıdaki örnekte bulunan bir tanımlama yapmamız yeterli olacaktır.

- Node label:

`kubectl label node node-1 region=region-1`

- pod.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    region: region-1
```