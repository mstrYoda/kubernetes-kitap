# RBAC NEDİR

Kubernetes içerisinde yetkilendirme için çeşitli yöntemler bulunmaktadır. Role Based Access Control bunlardan biridir.
Örneğin belirli bir service account için resource listeleme işlemi için yetki verip, resource silme işlemini kısıtlayabiliriz.

RBAC tanımlaması yaparken dikkat edilmesi gereken üç adet kavram bulunmaktadır.

*Subject*: Kullanıcılar, gruplar veya service accountları temsil eder.
*Resource*: Üzerinde yetkilendirme yapılacak kubernetes api objelerini temsil eder.
*Verbs*: Yapılacak işlemi temsil eder.

# Role & ClusterRole

Role ve ClusterRole kubernetes objeleri için belirli kuralları barındırır. Aralarındaki fark, Role namespace bazında çalışırken, ClusterRole adından da anlaşılacağı üzere cluster genelinde kullanılır.


Role oluşturmak için yaml tanımı yapabilir veya kubectl komutlarını kullanabiliriz.

`kubectl create role my-custom-role --verb=list --verb=get --resource=pods --resource=services --namespace k8boss`

Aynı tanımı yaml ile yapmak istersek:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-custom-role
  namespace: k8boss
rules:
  - apiGroups:
      - "*"
    resources:
      - services
      - pods
    verbs:
      - get
      - list
```

ClusterRole oluşturmak pek de farklı değil, tek fark, namespace belirtmiyoruz:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-custom-cluster-role
rules:
  - apiGroups:
      - "*"
    resources:
      - services
      - pods
    verbs:
      - get
      - list
```

# ServiceAccount

Service account tanımları podlarımız tarafından kullanılır ve ilgili podun yetkilerini belirler. Kubernetes ile haberleşen uygulamalar geliştiriyorsanız uygulamanıza özel roller içeren bir ServiceAccount oluşturmak isteyebilirsiniz.

kubectl ile ServiceAccount oluşturmak son derece basittir:

`kubectl create serviceaccount custom-sa -n k8boss`

İlgili komutum yaml karşılığı:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-sa
  namespace: k8boss
```

Oluşturduğumuz ServiceAccount'u biraz inceleyelim:

`kubectl get sa custom-sa -n k8boss -o yaml` komutu bize aşağıdaki gibi bir çıktı verecek:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2020-04-13T22:11:49Z"
  name: custom-sa
  namespace: k8boss
  resourceVersion: "1653"
  selfLink: /api/v1/namespaces/test/serviceaccounts/test-sa
  uid: 8fb1df02-6bab-43f1-afe8-3ede5f4a1710
secrets:
- name: custom-sa-token-jcq7h
```

ServiceAccount oluştururken bizim belirtmediğimiz `secret` alanının geldiğini görüyoruz. Kubernetes tarafından bir secret üretilip ServiceAccount ile ilişkilendirildi.

Şimdi oluşturulan secret değerine bakalım, `kubectl get secret custom-sa-token-jcq7h -n k8boss -o yaml` :

```
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EUXhNekl5TURNMU1sb1hEVE13TURReE1USXlNRE0xTWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTTBVCkgvMnFZSGpicVI5bXlka3ZUc0lsZnFRVlAvdzVvNXZhaUF5UkVVbkNXdWt0bFBJK3FSTnRvZmVTS3pSd003bnAKR1haek5vR3JlWEtNbEVhaktrMTRBdjRwN2F4OGlRUkJ5OGZWejU5TXd5NmVoRXR6NFpDN1dvSTBTNndxSVRENgo4cktLdmtCVmVRaG9RenFXMjRZclpDZ3VTTmkxNm90bUZ6SWh6dENmUkJWVUZLRjF5SEREdXQxUk9LYVl6U3h5Ckw5RFRXVlUxY3Q2L0t3N0NabGdkSDh0STJqc3NSWE5MdURHVmJkQ1RMelg5eTk0Y25nY0tTeVNiZ2F3dXQ2ZnoKWkZmVld2UzJBait1WjR5cUV1citNZ3dZdW9YQjlLSHhxcFp5YXVQaEJiSVR4T2pXM0RMWkZjTGoxY0xBNUVKdApZWmZ0eVZkVjNCSVFGNm5ZTi9jQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHaG5VUlVSTFBMVk9FS1I2c0h0VXZhWkVFSisKdGdPT3JTZnpZTVBqWXZTdURJYjgvR0o1ZWswL21uMzFwdlZVRktRYis5aHdwOFViYWZOeXR2dEhaZ0pjZGNnbQpSQlVob3Yzc043WEVvY1hiK3VpSHAwNmt3TlVnOTRETXNVRmRZcExwUGYydm9HcjBycXFDeVNHUFFCYkFCd0hCCktpLzZxaUFBM0VVVTAvZUJLOVVDdmVFNVhxcXl1LytJbG1hODFyQTZMQVYyYkE0Z2I1L0MvRmg0c0VJaUphd2oKQnI1TkEvVWhmWUhyaXNhVGRoLzlxWE5memMwcFUzejFqbGdxZnNGZURxWjNzSisrTWNJQk1PWWFaTVA1U1Z3UwpJbC9aQ056bHArTjdTTERqZTRDOWg0SXRXMWRGdGNQdnNNcXBtTmtQelVZN0pNcTQxNGlTUkUzeXYxbz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  namespace: dGVzdA==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklsWkdSMGR2TTBJdGFXc3pOMFZuYWpKSWIwMXpTR2hrTmtvemJsSjZOVmM1TTNacWVsTkJkazk2VWpRaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUowWlhOMElpd2lhM1ZpWlhKdVpYUmxjeTVwYnk5elpYSjJhV05sWVdOamIzVnVkQzl6WldOeVpYUXVibUZ0WlNJNkluUmxjM1F0YzJFdGRHOXJaVzR0YW1OeE4yZ2lMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2lkR1Z6ZEMxellTSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqaG1ZakZrWmpBeUxUWmlZV0l0TkRObU1TMWhabVU0TFRObFpHVTFaalJoTVRjeE1DSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHAwWlhOME9uUmxjM1F0YzJFaWZRLlJxeXFaQUx2M19qeUNhZnMtbF9pa0ctQXA4RFNPZjRoT2psaXRkODYxRzBDcXpmY0VYLW1GbkxlbFNHSDg1Z1MyeHR2SmxBci0zcG9sZjdnMEJCNlRKRHl0Ny1rN2V2a2dyTzVGbG85b2R3NVVqeWpLdFd6czdLbjBsci1DUDU4OXV1ekFPRTlIdHpnZTMwYlYxQWVmdEtZUmNsdUlxb1hVRk1ZWko5a3hXZk1hZkh4UlVOX2M1TkppRVNibktCeXhyelFqNV9xUVNoMFFKbkdxVjdCTWJJcksyZW02ME5hLWFJNWpCMDNxYnJsM2ZBUk91ZEphTW5BWEwwZDQxSkZJU0o2TW5qSlpKZlFXUWpTOEFNWlNMVXF4TlpXZlcwc2N0OGdhX3dLT0c4cUVMd2YxLWVJdFlZa1pRckRONVk5SEllc2U5QjZvRjFWZUtlcVdKdlY2QQ==
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: custom-sa
    kubernetes.io/service-account.uid: 8fb1df02-6bab-43f1-afe8-3ede5f4a1710
  creationTimestamp: "2020-04-13T22:11:49Z"
  name: custom-sa-token-jcq7h
  namespace: k8boss
  resourceVersion: "1652"
  selfLink: /api/v1/namespaces/k8boss/secrets/custom-sa-token-jcq7h
  uid: 45956fb0-2449-48c8-a0d3-d1a088a1f4ea
type: kubernetes.io/service-account-token
```

# RoleBinding & ClusterRoleBinding

RoleBinding, kullanıcılara veya service accountlara oluşturduğumuz Role objelerini bağlamamızı sağlar. Aralarındaki fark, ClusterRole tanımını sadece ClusterRoleBinding ile kullanabiliriz.

Bir ServiceAccount ile yapabildiklerimiz sorgulamak için şu komutu çalıştırabiliriz:

`kubectl auth can-i list pods --as=system:serviceaccount:k8boss:custom-sa -n k8boss`

Şimdilik "no" çıktısını görüyoruz. 

Bu ServiceAccount için RoleBinding oluşturalım:

`kubectl create rolebinding my-custom-role-binding --role=my-custom-role --serviceaccount=k8boss:custom-sa --namespace k8boss`

Komutun yaml karşılığı:

```
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: my-custom-role-binding
    namespace: k8boss
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: my-custom-role
  subjects:
  - kind: ServiceAccount
    name: custom-sa
    namespace: k8boss
```

ClusterRoleBinding oluşturmak çok da farklı değil:

`kubectl create clusterrolebinding my-custom-clusterrole-binding --clusterrole=my-custom-cluster-role --serviceaccount=k8boss:custom-sa`

Şimdi tekrar ServiceAccount ile yapabildiklerimizi sorgulayalım:

`kubectl auth can-i list pods --as=system:serviceaccount:k8boss:custom-sa -n k8boss`

Bu sefer "yes" çıktısını görüyoruz.

# Aggregated ClusterRole

Son olarak aggregated ClusterRole kavramına bakacağız. Bazı durumlarda birden fazla ClusterRole objesini birleştirmek isteyebiliriz, böyle bir durumda `aggregationRule` alanını kullanabiliriz.

Örnek olarak podları ve servisleri listeleyebildiğimizi iki farklı ClusterRole olsun.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-custom-cluster-role
  labels:
    rbac.example.com/aggregate-to-pod-list: "true"
rules:
  - apiGroups:
      - "*"
    resources:
      - pods
    verbs:
      - list

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-custom-cluster-role-2
  labels:
    rbac.example.com/aggregate-to-service-list: "true"
rules:
  - apiGroups:
      - "*"
    resources:
      - service
    verbs:
      - list
```

Şimdi yukarıdaki iki ClusterRole objesini aggregationRule alanı ile birleştiren yeni bir ClusterRole oluşturalım:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-aggregated-role
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-pod-list: "true"
      rbac.example.com/aggregate-to-service-list: "true"
rules: [] # The control plane automatically fills in the rules
```