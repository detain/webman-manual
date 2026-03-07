# Klasör Yapısı

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

Bir uygulama eklentisi, webman ile aynı klasör yapısına ve yapılandırma dosyalarına sahiptir. Uygulamada, geliştirme deneyimi normal bir webman uygulaması geliştirmekten neredeyse farksızdır.

Eklenti klasörleri ve adlandırma PSR-4 tanımına uyar. Eklentiler `plugin` klasöründe tutulduğundan, tüm ad alanları `plugin` ile başlar, örneğin `plugin\foo\app\controller\UserController`.

## api Klasörü Hakkında

Her eklentide bir `api` klasörü bulunur. Uygulamanız diğer uygulamalar tarafından çağrılacak dahili arayüzler sunuyorsa, bu arayüzleri `api` klasörüne koyun.

Not: Burada sözü edilen arayüzler fonksiyon çağrı arayüzleridir, ağ/HTTP arayüzleri değildir.

Örneğin e-posta eklentisi, diğer uygulamaların e-posta gönderirken çağırması için `plugin/email/api/Email.php` konumunda `Email::send()` arayüzünü sunar. Ayrıca `plugin/email/api/Install.php`, webman-admin eklenti pazarının yükleme veya kaldırma işlemlerini çalıştırması için otomatik oluşturulur.
