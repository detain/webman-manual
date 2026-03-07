# Çok Dilli

Çok dilli destek [symfony/translation](https://github.com/symfony/translation) bileşenini kullanır.

## Kurulum
```
composer require symfony/translation
```

## Dil Paketleri Oluşturma
webman varsayılan olarak dil paketlerini `resource/translations` dizininde saklar (yoksa oluşturun). Dizini değiştirmek için `config/translation.php` içinde ayarlayın.
Her dil bir alt dizine karşılık gelir, dil tanımları varsayılan olarak `messages.php` dosyasında tutulur. Örnek:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

Tüm dil dosyaları bir dizi döndürür, örneğin:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## Yapılandırma

`config/translation.php`

```php
return [
    // Varsayılan dil
    'locale' => 'zh_CN',
    // Yedek dil: Mevcut dilde çeviri bulunamazsa yedek dildeki çeviri denenir
    'fallback_locale' => ['zh_CN', 'en'],
    // Dil dosyalarının saklandığı dizin
    'path' => base_path() . '/resource/translations',
];
```

## Çeviri

Çeviri için `trans()` metodu kullanılır.

Dil dosyası `resource/translations/zh_CN/messages.php` oluşturun:
```php
return [
    'hello' => '你好 世界!',
];
```

Dosya `app/controller/UserController.php` oluşturun:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

`http://127.0.0.1:8787/user/get` adresine erişildiğinde "你好 世界!" döner.

## Varsayılan Dili Değiştirme

Dil değiştirmek için `locale()` metodu kullanılır.

Dil dosyası `resource/translations/en/messages.php` ekleyin:
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // Dili değiştir
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
`http://127.0.0.1:8787/user/get` adresine erişildiğinde "hello world!" döner.

`trans()` fonksiyonunun 4. parametresiyle geçici olarak dil değiştirilebilir. Örneğin yukarıdaki örnek şununla eşdeğerdir:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 4. parametre dili değiştirir
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## Her İstek İçin Dili Açıkça Ayarlama
translation bir singleton olduğundan tüm istekler aynı örneği paylaşır. Bir istek `locale()` ile varsayılan dili ayarlarsa, bu süreçteki sonraki tüm istekleri etkiler. Bu nedenle her istek için dili açıkça ayarlamalıyız. Örneğin aşağıdaki middleware kullanılır:

Dosya `app/middleware/Lang.php` oluşturun (dizin yoksa oluşturun):
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

`config/middleware.php` içinde global middleware ekleyin:
```php
return [
    // Global middleware
    '' => [
        // ... diğer middleware'ler atlandı
        app\middleware\Lang::class,
    ]
];
```


## Yer Tutucu Kullanma
Bazen bir mesajda çevrilmesi gereken değişkenler bulunur, örneğin
```php
trans('hello ' . $name);
```
Bu durumda yer tutucu kullanılır.

`resource/translations/zh_CN/messages.php` dosyasını güncelleyin:
```php
return [
    'hello' => '你好 %name%!',
];
```
Çeviri sırasında değerler ikinci parametreyle aktarılır:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## Çoğul Biçimlerle Çalışma
Bazı dillerde cümle yapısı miktara göre değişir. Örneğin `There is %count% apple`, `%count%` 1 olduğunda doğrudur, 1'den büyük olduğunda yanlıştır.

Bu durumda çoğul biçimleri listelemek için **pipe** (`|`) kullanılır.

Dil dosyası `resource/translations/en/messages.php` içine `apple_count` ekleyin:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

Sayı aralıkları belirleyerek daha karmaşık çoğul kuralları da oluşturabiliriz:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## Dil Dosyası Belirtme

Varsayılan dil dosyası adı `messages.php`'dir, ancak başka isimli dil dosyaları da oluşturulabilir.

Dil dosyası `resource/translations/zh_CN/admin.php` oluşturun:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

Dil dosyası `trans()` fonksiyonunun 3. parametresiyle belirtilir (`.php` uzantısı yazılmaz).
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## Daha Fazla Bilgi
[symfony/translation dokümantasyonuna](https://symfony.com/doc/current/translation.html) bakın
