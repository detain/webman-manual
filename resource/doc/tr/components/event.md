# Olay İşleme
`webman/event`, kodla uğraşmadan iş mantığını çalıştırmanıza ve modüller arasında gevşek bağ sağlamanıza imkân veren zarif bir olay mekanizması sunar. Tipik senaryo: Yeni bir kullanıcı başarıyla kayıt olduğunda, `user.register` gibi özel bir olay yayınlamanız yeterli; her modül bu olayı alıp ilgili mantığı çalıştırabilir.

## Kurulum
`composer require webman/event`

## Olaylara Abone Olma
Olay abonelikleri `config/event.php` dosyası üzerinden merkezi olarak yapılandırılır.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...diğer olay işleme fonksiyonları...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...diğer olay işleme fonksiyonları...
    ]
];
```

**Not:**
- `user.register`, `user.logout` vb. olay adlarıdır (string türü). Küçük harfli kelimeler ve nokta (`.`) ile ayrım önerilir.
- Bir olayın birden fazla işleme fonksiyonu olabilir; yapılandırma sırasına göre çağrılır.

## Olay İşleme Fonksiyonları
İşleme fonksiyonları sınıf metodu, fonksiyon veya closure olabilir.

Örnek: `app/event/User.php` sınıfını oluşturun (dizin yoksa oluşturun).

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## Olay Yayınlama
Olay yayınlamak için `Event::dispatch($event_name, $data);` veya `Event::emit($event_name, $data);` kullanın. Örnek:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

Yayınlama için iki fonksiyon var: `Event::dispatch($event_name, $data);` ve `Event::emit($event_name, $data);` — parametreler aynı. Fark: `emit` istisnaları dahilde yakalar; bir işleyicide istisna oluşursa diğerleri çalışmaya devam eder. `dispatch` istisnaları yakalamaz; herhangi bir işleyicide istisna oluşursa yürütme durur ve istisna yukarı iletilir.

> **İpucu**
> `$data` parametresi dizi, sınıf örneği, dize vb. herhangi bir veri olabilir.

## Joker Karakter ile Olay Dinleme
Joker karakter kaydı, aynı dinleyicide birden çok olayı işlemenize izin verir. Örnek `config/event.php` içinde:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

İşleme fonksiyonunun ikinci parametresi `$event_data` ile somut olay adı alınabilir:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // Somut olay adı, örn. user.register, user.logout vb.
        var_export($user);
    }
}
```

## Olay Yayınını Durdurma
İşleme fonksiyonunda `false` döndürüldüğünde, o olayın yayını durur.

## Closure ile Olay İşleme
İşleme fonksiyonu sınıf metodu veya closure olabilir. Örnek:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## Olayları ve Dinleyicileri Görüntüleme
Projede yapılandırılan tüm olayları ve dinleyicileri görmek için `php webman event:list` komutunu kullanın.

## Destek Kapsamı
Ana projenin yanı sıra [temel eklentiler](../plugin/base.md) ve [uygulama eklentileri](../app/app.md) de `event.php` yapılandırmasını destekler.
**Temel eklenti config dosyası:** `config/plugin/sağlayıcı/eklenti-adı/event.php`
**Uygulama eklentisi config dosyası:** `plugin/eklenti-adı/config/event.php`

## Dikkat Edilmesi Gerekenler
Olay işleme asenkron değildir; yavaş işler için uygun değildir. Yavaş işler için [webman/redis-queue](https://www.workerman.net/plugin/12) gibi mesaj kuyruğu kullanılmalıdır.
