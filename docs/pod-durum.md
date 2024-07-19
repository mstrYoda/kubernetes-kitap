# Pod Durumlari

Bir Pod'un durum alanı, bir phase alanına sahip olan bir PodStatus nesnesidir.

Bir Pod'un phase'i, Pod'un yaşam döngüsünde nerede olduğunun basit ve yüksek düzeyde bir özetidir. Phase, container veya Pod durumunun kapsamlı bir gözlemi veya kapsamlı bir durum makinesi olması amaçlanmamıştır.

Pod phase değerlerinin sayısı ve anlamları sıkı bir şekilde korunur. Burada belgelenenler dışında, belirli bir phase değerine sahip Pod'lar hakkında hiçbir varsayımda bulunulmamalıdır.

Phase için olası değerler şunlardır:

| Değer      | Açıklama |
|------------|----------|
| Pending    | Pod, Kubernetes cluster'ı tarafından kabul edilmiştir, ancak bir veya daha fazla container kurulmamış ve çalışmaya hazır hale getirilmemiştir. Bu, bir Pod'un zamanlanmayı beklerken geçirdiği zamanı ve ağ üzerinden container görüntülerini indirirken geçirdiği zamanı içerir. |
| Running    | Pod bir node'a bağlanmış ve tüm containerlar oluşturulmuştur. En az bir container hala çalışıyor veya başlatılıyor ya da yeniden başlatılıyor. |
| Succeeded  | Pod'daki tüm containerlar başarıyla sonlanmış ve yeniden başlatılmayacaktır. |
| Failed     | Pod'daki tüm containerlar sonlanmış ve en az bir container başarısızlıkla sonlanmıştır. Yani, container sıfır olmayan bir durum kodu ile çıkmış veya sistem tarafından sonlandırılmış ve otomatik olarak yeniden başlatılmak üzere ayarlanmamıştır. |
| Unknown    | Bir nedenle Pod'un durumu elde edilememiştir. Bu phase tipik olarak Pod'un çalışması gereken node ile iletişimde bir hata olduğunda ortaya çıkar. |

Not:
Bir Pod silinirken, bazı kubectl komutları tarafından Terminating olarak gösterilir. Bu Terminating durumu, Pod phase'lerinden biri değildir. Bir Pod'a, gracefully bir şekilde terminate edilmesi için bir süre verilir ve bu varsayılan olarak 30 saniyedir. Bir Pod'u zorla sonlandırmak için --force bayrağını kullanabilirsiniz. Kubernetes 1.27'den beri, kubelet, statik Pod'lar ve finalizer olmadan zorla silinen Pod'lar hariç, silinmiş Pod'ları API sunucusundan silinmeden önce bir terminal phase'e (Pod containerlarının çıkış durumlarına bağlı olarak Failed veya Succeeded) geçirir.

Bir node ölürse veya cluster'dan düşerse, Kubernetes, kaybolan node'daki tüm Pod'ların phase'ini Failed olarak ayarlamak için bir politika uygular.