# Önbellek

[webman/cache](https://github.com/webman-php/cache), [symfony/cache](https://github.com/symfony/cache) tabanlı bir önbellek bileşenidir; hem coroutine hem de coroutine olmayan ortamlarla uyumludur ve bağlantı havuzu destekler.

## Kurulum

```php
composer require -W webman/cache
```

## Örnek
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Cache::set($key, rand());
        return response(Cache::get($key));
    }
}
```

## Yapılandırma Dosyası Konumu
Yapılandırma dosyası `config/cache.php` konumundadır. Yoksa elle oluşturun.

## Yapılandırma Dosyası İçeriği
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` dört sürücü destekler: **file**, **redis**, **array** ve **apcu**.

#### file sürücüsü
Varsayılan sürücü. Harici bağımlılığı yoktur. Süreçler arası önbellek paylaşımını destekler. Çok sunuculu paylaşımı desteklemez.

#### array sürücüsü
Bellekte depolama; en iyi performans ancak bellek tüketir. Süreçler veya sunucular arası paylaşımı desteklemez. Süreç yeniden başlatıldığında veri kaybolur. Önbellek hacmi küçük projeler için uygundur.

#### apcu sürücüsü
Bellekte depolama. Performans array sürücüsünden sonra gelir. Süreçler arası önbellek paylaşımını destekler. Çok sunuculu paylaşımı desteklemez. Süreç yeniden başlatıldığında veri kaybolur. Önbellek hacmi küçük projeler için uygundur.

> [APCu eklentisi](https://pecl.php.net/package/APCu) kurulmuş ve etkinleştirilmiş olmalıdır. Sık önbellek yazma/silme senaryoları için önerilmez; belirgin performans düşüşüne yol açabilir.

#### redis sürücüsü
[webman/redis](./redis.md) bileşenine bağımlıdır. Süreçler ve sunucular arası önbellek paylaşımını destekler.

**stores.redis.connection**

`stores.redis.connection`, `config/redis.php` içinde tanımlanan anahtara karşılık gelir. Redis kullanıldığında bağlantı havuzu ayarları dahil `webman/redis` yapılandırması yeniden kullanılır.

**`config/redis.php` içinde önbellek için ayrı bir Redis yapılandırması eklenmesi önerilir, örneğin:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Yeni ekle
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Ardından `stores.redis.connection` değerini `cache` olarak ayarlayın. Son `config/cache.php` dosyası şuna benzer:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Depo Değiştirme
Farklı sürücüler kullanmak için depoyu manuel olarak değiştirebilirsiniz, örneğin:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **İpucu**
> Önbellek anahtar adları [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) ile sınırlıdır ve `{}()/\@:` karakterlerinden herhangi birini içeremez. `symfony/cache` 7.2.4 itibarıyla bu kontrol PHP ini ayarı `zend.assertions=-1` ile geçici olarak atlanabilir.

## Diğer Önbellek Bileşenlerini Kullanma

[ThinkCache](https://github.com/webman-php/think-cache) bileşeni için [Diğer Veritabanları](others.md#ThinkCache) bölümüne bakın.
