# webman Nedir

webman, Workerman üzerine inşa edilmiş HTTP, WebSocket, TCP, UDP ve diğer modülleri entegre eden yüksek performanslı bir hizmet çerçevesidir. Kalıcı bellek, korutin ve bağlantı havuzu gibi gelişmiş teknolojilerle webman, geleneksel PHP'nin performans sınırlarını aşmanın yanı sıra uygulama alanlarını da önemli ölçüde genişletir.

Ayrıca webman, geliştiricilerin diğer geliştiriciler tarafından oluşturulan fonksiyonel modülleri hızla entegre edip yeniden kullanmasını sağlayan güçlü bir eklenti mekanizması sunar. Web sitesi kurmak, HTTP API geliştirmek, anlık mesajlaşma, IoT sistemleri, oyunlar, TCP/UDP hizmetleri, Unix Socket hizmetleri veya benzeri işler olsun, webman hepsini üstün performans ve esneklikle kolayca üstlenir.

# webman Felsefesi
**En küçük çekirdek ile maksimum genişletilebilirlik ve en yüksek performans sağlamak.**

webman yalnızca temel işlevleri (yönlendirme, ara yazılım, oturum, özel işlem arabirimi) sunar. Diğer tüm işlevler Composer ekosisteminden yeniden kullanılır. Yani webman içinde alıştığınız bileşenleri kullanabilirsiniz; örneğin veritabanı için Laravel'ın [illuminate/database](./db/tutorial.md), ThinkPHP'nin [ThinkORM](./db/thinkorm.md) veya `Medoo` gibi başka bileşenleri seçebilirsiniz. Bunları webman'a entegre etmek çok kolaydır.

# webman'ın Özellikleri

1. Yüksek istikrar. webman workerman üzerine kuruludur; workerman sektörde çok az hatayla yüksek istikrarlı bir soket çerçevesidir.

2. Çok yüksek performans. webman'ın performansı geleneksel php-fpm çerçevelerinden 10–100 kat daha yüksektir ve Go'nun gin, echo vb. çerçevelerinden yaklaşık iki kat daha fazladır.

3. Yüksek yeniden kullanılabilirlik. Mevcut Composer ekosistemi değiştirilmeden yeniden kullanılabilir.

4. Yüksek genişletilebilirlik. Özel işlemleri destekler; workerman'ın yapabildiği her şey yapılabilir.

5. Son derece basit ve kullanımı kolay; öğrenme maliyeti düşük, kod yazımı geleneksel çerçevelerle aynıdır.

6. [İkili paketleme](./others/bin.md) destekler; PHP ortamı olmadan doğrudan çalıştırılabilir.

7. En müsamahakâr ve geliştirici dostu MIT açık kaynak lisansını kullanır.

# Proje Adresleri
GitHub: https://github.com/walkor/webman **Yıldız vermeyi unutmayın!**

Gitee: https://gitee.com/walkor/webman **Yıldız vermeyi unutmayın!**

# Üçüncü Taraf Performans Test Verileri

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Veritabanı sorgulama işlemlerinde webman tek makinede 390.000 QPS'e kadar ulaşır; geleneksel php-fpm mimarisindeki Laravel çerçevesinden neredeyse 80 kat daha yüksektir.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Veritabanı sorgulama işlemlerinde webman, benzer Go web çerçevelerinden yaklaşık iki kat daha yüksek performans gösterir.

Yukarıdaki veriler [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf) kaynaklıdır.
