# Dizin Yapısı
```
.
├── app                           Uygulama dizini
│   ├── controller                Denetleyici dizini
│   ├── model                     Model dizini
│   ├── view                      Görünüm dizini
│   ├── middleware                Ara yazılım dizini
│   │   └── StaticFile.php        Dahili statik dosya ara yazılımı
│   ├── process                   Özel işlem dizini
│   │   ├── Http.php              Http işlemi
│   │   └── Monitor.php          İzleme işlemi
│   └── functions.php             İş özel işlevleri bu dosyada yazılır
├── config                        Yapılandırma dizini
│   ├── app.php                   Uygulama yapılandırması
│   ├── autoload.php              Burada yapılandırılan dosyalar otomatik yüklenecektir
│   ├── bootstrap.php             İşlem başlatıldığında onWorkerStart'ta çalışan geri çağırma yapılandırması
│   ├── container.php             Konteyner yapılandırması
│   ├── dependence.php            Konteyner bağımlılığı yapılandırması
│   ├── database.php             Veritabanı yapılandırması
│   ├── exception.php             İstisna yapılandırması
│   ├── log.php                   Günlük yapılandırması
│   ├── middleware.php            Ara yazılım yapılandırması
│   ├── process.php               Özel işlem yapılandırması
│   ├── redis.php                 Redis yapılandırması
│   ├── route.php                 Rota yapılandırması
│   ├── server.php                Port, işlem sayısı vb. sunucu yapılandırması
│   ├── view.php                  Görünüm yapılandırması
│   ├── static.php                Statik dosya anahtarı ve statik dosya ara yazılımı yapılandırması
│   ├── translation.php           Çoklu dil yapılandırması
│   └── session.php               Oturum yapılandırması
├── public                        Statik kaynak dizini
├── runtime                       Uygulama çalışma zamanı dizini, yazma izni gerekir
├── start.php                     Hizmet başlatma dosyası
├── vendor                        Composer ile yüklenen üçüncü taraf kitaplık dizini
└── support                       Kitaplık uyarlaması (üçüncü taraf kitaplıkları dahil)
    ├── Request.php               İstek sınıfı
    ├── Response.php              Yanıt sınıfı
    ├── Setup.php                 Kurulum sihirbazı betiği
    └── bootstrap.php             İşlem başladıktan sonra başlatma betiği
```
