# buildah

Buildah, açık kaynaklı bir araçtır ve bir container image oluşturmak için kullanılır. Buildah, Docker daemon'ı olmadan da çalıştırılabilir ve container image oluşturmak için Dockerfile kullanılmasına gerek kalmaz.

Buildah, container image oluştururken aşağıdaki adımları gerçekleştirir:

 - Container image'ınızın base image'ını seçin. Base image, container image'ınızın üzerine yüklenecek ve container içinde çalışacak bir sistem görüntüsüdür. Örneğin, alpine, ubuntu, gibi.

 - Container image'ınızda yükleyeceğiniz paketleri belirtin. Örneğin, apache, nginx, gibi.

 - Container image'ınızda çalıştıracağınız uygulamayı tanımlayın. Örneğin, Python, Node.js, gibi.

 - Container image'ınızda çalıştıracağınız uygulamanın çalışma dizinini tanımlayın.

 - Container image'ınızda çalıştırılacak olan komutları tanımlayın. Örneğin, "npm start", "python app.py", gibi.

## Kullanılışı

buildah dockerfile'ına gerek kalmadan custom image yapabilmeyi sağlayan bir araçtır. Verilen komutlarla Dockerfile'da yaptığınız işi komut vererek yaparsınız.

buildah çalışabilmek için bir container runtime'a ihtiyaç duyar siz image çekmesini söylediğinizde container runtime`ın default registry'sini kullanır. Custom registry kullanabilmenizde veya local'den kullanabilmenizde mümkün.

```bash
buildah from IMAGE # bu image'ı kullanır bu image ı kullan deriz ve geçici değişiklikler bu image'a kaydedilir.
buildah run IMAGE COMMAND # Dockerfile'da yaptığımız iş burası amacı aynıdır.
buildah add IMAGE SOURCE DESTINATION # dosya ekle
buildah commit IMAGE NAME # commit et burda artık image'ımız name'e göre yeni image oluşturulur yapılan değişikler işlenmiş olur.
buildah push IMAGE REGISTRY # yeni image'ımızı registry'ye gönderelim.
```

**Not:** commit ettiğimiz kısmı yapmadan from kısmını tekrar çalıştırırsak yaptığımız geçici değişiklikler silinir.