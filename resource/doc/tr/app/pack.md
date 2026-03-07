# Paketleme

Örneğin, foo uygulama eklentisini paketlemek için:

* `plugin/foo/config/app.php` dosyasında sürüm numarasını ayarlayın (**önemli**)
* `plugin/foo` içinde gereksiz dosyaları silin, özellikle `plugin/foo/public` altındaki yükleme işlevi test dosyalarını silin
* Projeniz veritabanı tablosu oluşturma vb. işlemler içeriyorsa, `plugin/foo/install.sql` dosyasını doğru şekilde yapılandırın. [Veritabanı bölümü](database.md) bölümüne bakın
* Projenizin kendi bağımsız veritabanı ve Redis yapılandırması varsa, önce bu yapılandırmaları silin. Bu yapılandırmalar, uygulamaya ilk erişimde kurulum sihirbazı (kendi başınıza uygulamanız gerekir) ile tetiklenmeli ve yöneticinin manuel olarak doldurup oluşturmasına izin vermelidir.
* Projeniz webman admin arka uç menüleri içeriyorsa, `plugin/foo/config/menu.php` dosyasını yapılandırın; böylece eklenti yüklendiğinde bu menüler otomatik olarak ayarlanır. [webman-admin menü içe aktarma](https://www.workerman.net/doc/webman-admin/app-development/menu.html) bölümüne bakın
* Orijinal haline geri getirilmesi gereken diğer dosyaları geri yükleyin
* Yukarıdaki adımları tamamladıktan sonra `{ana proje}/plugin/` dizinine gidin
* Linux kullanıcıları: foo.zip oluşturmak için `zip -r foo.zip foo` komutunu çalıştırın
* Windows kullanıcıları: foo klasörüne sağ tıklayıp "ZIP dosyası olarak sıkıştır"ı seçerek foo.zip oluşturun

**foo.zip paketlenmiş dosyadır. Sonraki bölüme [Eklenti yayınlama](publish.md) bakın**
