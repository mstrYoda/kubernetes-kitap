# Init Container nedir?

InitContainer; pod içerisinde bulunan, containerlar ile aynı tipte fakat containerlar başlamadan önce çalışan bir yapıdır.

Bir pod içerisinde birden fazla container olabildiği gibi birden fazla initContainer da olabilir. Pod içerisinde 'initContainers' fieldında dizi olarak tanımlanırlar.

InitContainerlar sıralı olarak çalışırlar. InitContainer içerisindeki process çalışıp bitmelidir. Bir initContainer içerisindeki process başarılı bir şekilde bitmeden diğer initContainer veya containerlar çalışmaz.

Eğer bir initContainer başarılı olmazsa pod yeniden başlatılır. Pod'un restartPolicy'si 'Never' ise pod 'Failed' statüsüne geçer.

## InitContainer ve Container farkları nelerdir?
- InitContainerlarda lifecycle, livenessProbe, readinessProbe ve startupProbe desteği bulunmamaktadır çünkü podun sağlıklı bir şekilde çalışıyor olması için initContainerların çalışıp başarılı bir şekilde tamamlanmış olmaları gereklidir.
- Pod içerisinde birden fazla initContainer varsa bunlar sıralı olarak çalışırlar. Bütün initContainerlar başarılı bir şekilde çalışıp tamamlandıktan sonra Containerlar aynı anda çalışmaya başlarlar.

InitContainerlar genellikle diğer containerlar çalışmaya başlamadan önce pod içerisinde yapılması gereken işleri yapmak veya uygulama containerının başlamasını bekletmek gibi ihtiyaçları karşılamak için kullanılabilirler.
