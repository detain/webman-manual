# Veritabanı Hızlı Başlangıç (Laravel Veritabanı Bileşeni ile)

[webman/database](https://github.com/webman-php/database), [illuminate/database](https://github.com/illuminate/database) üzerine kuruludur ve coroutine ile coroutine olmayan ortamlar için bağlantı havuzu ekler. Kullanımı Laravel ile aynıdır.

ThinkPHP veya diğer veritabanlarını kullanmak için [Diğer Veritabanı Bileşenlerini Kullanma](others.md) bölümüne bakabilirsiniz.

## Veritabanı Kurulumu

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

Kurulumdan sonra yeniden başlatma gerekir (reload geçersizdir).

> **İpucu**
> webman/database, Laravel'in `illuminate/database` paketine bağımlıdır; bağımlılıkları kurulum sırasında otomatik yüklenir.

> **Not**
> Sayfalama, veritabanı olayları veya SQL kaydı gerekmiyorsa şunu yeter:
> `composer require -W webman/database`

## Veritabanı Yapılandırması
`config/database.php`
```php

return [
    // Varsayılan veritabanı
    'default' => 'mysql',

    // Bağlantı yapılandırmaları
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // swoole veya swow kullanıldığında zorunlu
            ],
            'pool' => [ // Bağlantı havuzu ayarları
                'max_connections' => 5, // Maksimum bağlantı sayısı
                'min_connections' => 1, // Minimum bağlantı sayısı
                'wait_timeout' => 3,    // Havuzdan bağlantı alma için maksimum bekleme süresi; aşılınca exception. Sadece coroutine ortamında geçerli
                'idle_timeout' => 60,   // Havuzdaki bağlantıların maksimum boşta kalma süresi; sonrasında min_connections'a kadar kapatılır
                'heartbeat_interval' => 50, // Havuz kalp atışı aralığı (saniye); 60'tan küçük önerilir
            ],
        ],
    ],
];
```

`pool` ayarları dışında Laravel ile aynıdır.

## Bağlantı Havuzu Hakkında
* Her işlemin kendi havuzu vardır; havuzlar işlemler arasında paylaşılmaz.
* Coroutine kapalıyken istekler sırayla işlenir, eşzamanlılık olmaz; bu yüzden havuzda en fazla bir bağlantı olur.
* Coroutine açıkken istekler paralel işlenir; havuz gereğe göre bağlantı sayısını ayarlar, `max_connections`'ı aşmaz, `min_connections`'ın altına düşmez.
* Havuz en fazla `max_connections` olduğu için, veritabanı kullanan coroutine sayısı bu değeri aştığında bazıları en fazla `wait_timeout` saniye sırada bekler; aşılınca exception oluşur.
* Boştayken (coroutine olsun olmasın) bağlantılar `idle_timeout` sonrası geri alınır, `min_connections` sayısına kadar (`min_connections` 0 olabilir).


## Veritabanı Kullanım Örneği
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

Kullanım Laravel ile aynıdır; `Db::table()` metoduyla veritabanına işlem yapılır.
