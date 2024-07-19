# Container Statüleri

Kubernetes Pod'un içerisindeki her container'ın statüsünü takip eder. Scheduler oluşturlacak Pod'u bir Node'a atasığı zaman Kubelet Pod içerisindeki containerları, container runtime kullanarak oluşturmaya başlar. Bır container'ın olası 3 statüsü vardır ve her statünün kendine özel anlamları vardır.

- **Waiting:** Eğer container Running ve ya Terminated statüsünde değilse bu statüdedir. Bu statüde bulunan bir container, başlamayı tamamlamak için ihtiyaç duyduğu işlemleri hala yürütüyor olabilir: örneğin, registry'den container image'ini çekmek ya da secretları dosya sistemine yazmak. Bir container'ın neden bu statüde olduğunu öğrenmek için <kubectl> kullanarak Pod'a sorgu atabiliriz. 

- **Running:** Eğer container Running statüsünde ise herhangi bir sorun olmadan ayağa kalkmış ve çalışıyor demektir. <kubectl> ile Pod'a sorgu atarak container'ın ne zaman Running statüsüne girdiği öğrenilebilir.

- **Terminated:** Eğer container Termintaed statüsünde ise bir problemden kaynaklı container ayağa kalkıp çalışamadı ve bu statüye düştü demektir. <kubectl> ile Pod'a sorgu atarak Terminated olan container hakkında hata sebebi, hata kodu, başlangıç ve bitiş saatleri hakkında bilgiler öğrenilebilir.

Bir Pod'un container'larının statüleri hakkında detaylı bilgiye ulaşmak için kubectl describe <name-of-pod> komutunu çalıştırılabilir. Bu komut Pod içerisindeki her container'ın statüleri hakkında detaylı bir çıktı verir.
