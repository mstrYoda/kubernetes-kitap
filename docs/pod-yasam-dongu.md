# Pod Yaşam Döngüsü

Podlar belirli bir yaşam döngüsünü takip eder, bu döngü Pending aşamasıyla başlar ve containerlardan biri çalışır durumunda ise Running aşamasına doğru ilerler. Daha sonrasında Pod'da bulunan bir containerın fail olup terminate olup olmama durumuna bağlı olarak Failed ya da Succeeded aşamasına geçer.

Bireysel uygulama containerları gibi, Pod'lar da nispeten geçici (kalıcı olmaktan ziyade) varlıklar olarak kabul edilir. Pod'lar oluşturulur, benzersiz bir kimlik (UID) atanır ve Node'larda çalışmak üzere zamanlanır; burada sonlandırılana (yeniden başlatma politikasına göre) veya silinene kadar kalırlar. Bir Node ölürse, o Node üzerinde çalışan (veya çalışmak üzere zamanlanmış) Pod'lar silinmek üzere işaretlenir. Kontrol düzlemi, bir zaman aşımı süresinden sonra Pod'ları kaldırmak üzere işaretler.

## Pod lifetime

Bir Pod çalışırken, kubelet bazı türdeki hataları ele almak için konteynerleri yeniden başlatabilir. Bir Pod içinde, Kubernetes farklı container durumlarını izler ve Pod'u tekrar sağlıklı hale getirmek için ne tür bir eylem yapması gerektiğine karar verir.

Kubernetes API'sinde, Pod'ların hem bir spesifikasyonu hem de gerçek bir durumu vardır. Bir Pod nesnesinin durumu, bir dizi Pod koşulundan oluşur. Ayrıca, uygulamanız için yararlıysa, bir Pod'un durum verilerine özel hazır olma bilgilerini ekleyebilirsiniz (readiness probes).

Pod'lar yaşamları boyunca yalnızca bir kez zamanlanır; bir Pod'u belirli bir node'a atamaya binding denir ve hangi node'un kullanılacağını seçme sürecine scheduling denir. Bir Pod zamanlandığında ve bir node'a bağlandığında, Kubernetes o Pod'u node'da çalıştırmaya çalışır. Pod, durana kadar veya Pod sonlandırılana kadar o node'da çalışır; Kubernetes seçilen node'da Pod'u başlatamazsa (örneğin, node Pod başlamadan önce çökerse), o belirli Pod asla başlamaz.

Pod Scheduling Readiness'i kullanarak bir Pod'un zamanlanmasını, tüm scheduling gates kaldırılana kadar erteleyebilirsiniz. Örneğin, bir dizi Pod tanımlamak isteyebilir ancak tüm Pod'lar oluşturulduğunda zamanlamayı tetiklemek isteyebilirsiniz.


## Pods and fault recovery

Pod içindeki container'lardan biri başarısız olursa, Kubernetes belirli bir containerı yeniden başlatmayı deneyebilir. 

Ancak, Pod'lar cluster'ın geri kazanamayacağı bir şekilde fail olabilir ve bu durumda Kubernetes Pod'u daha fazla iyileştirmeye çalışmaz; bunun yerine, Kubernetes Pod'u siler ve otomatik iyileştirme sağlamak için diğer bileşenlere güvenir.

Bir Pod bir node'a zamanlanmışsa ve o node daha sonra fail olursa, Pod sağlıksız olarak kabul edilir ve Kubernetes sonunda Pod'u siler. Bir Pod, kaynak yetersizliği veya Node bakımından dolayı tahliyeden sağ çıkamaz.

Kubernetes, nispeten geçici Pod örneklerini yönetme işini üstlenen bir controller adlı daha yüksek düzeyde bir soyutlama kullanır.

Belirli bir Pod (UID ile tanımlanmış) asla farklı bir node'a "yeniden zamanlanmaz"; bunun yerine, o Pod yeni, neredeyse aynı bir Pod ile değiştirilebilir. Bir yedek Pod oluşturursanız, eski Pod'un sahip olduğu aynı adı (.metadata.name) bile kullanabilir, ancak yedek Pod eski Pod'dan farklı bir .metadata.uid'ye sahip olur.

Kubernetes, mevcut bir Pod için bir yedeğin, değiştirilen eski Pod ile aynı node'a zamanlanacağını garanti etmez.
