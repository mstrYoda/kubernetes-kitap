# Init Container nedir?

InitContainer'lar uygulamanın ana container'ı ayağa kalkmadan önce çalışan, pod içerisinde bulunan, containerlar ile aynı tipte fakat ana containerlar başlamadan önce çalışan özelleştirilmiş öncül containerlardır. InitContainer kullanımına örnek olarak uygulamanın ayağa kalkması için gereken config ve secretlerı Pod'un dosya sistemine yazmamıza yardımcı olacak bir uygulama ya da bazı kurulum scriptleri barındıran bir uygulama olarak düşünebiliriz. 

Bir pod içerisinde birden fazla container olabildiği gibi birden fazla initContainer da olabilir. Pod içerisinde 'initContainers' fieldında dizi olarak tanımlanırlar.

InitContainerlar sıralı olarak çalışırlar. InitContainer içerisindeki process çalışıp bitmelidir. Bir initContainer içerisindeki process başarılı bir şekilde bitmeden diğer initContainer veya containerlar çalışmaz.

Eğer bir initContainer başarılı olmazsa pod yeniden başlatılır. Pod'un restartPolicy'si 'Never' ise pod 'Failed' statüsüne geçer.

## InitContainer ve Container farkları nelerdir?
- InitContainer'larda lifecycle, livenessProbe, readinessProbe ve startupProbe desteği bulunmamaktadır çünkü podun sağlıklı bir şekilde çalışıyor olması için initContainer'ların çalışıp başarılı bir şekilde tamamlanmış olmaları gereklidir.
- Pod içerisinde birden fazla initContainer varsa bunlar sıralı olarak çalışırlar. Bütün initContainer'lar başarılı bir şekilde çalışıp tamamlandıktan sonra Containerlar aynı anda çalışmaya başlarlar.

InitContainer'lar genellikle diğer containerlar çalışmaya başlamadan önce pod içerisinde yapılması gereken işleri yapmak veya uygulama containerının başlamasını bekletmek gibi ihtiyaçları karşılamak için kullanılabilirler.
