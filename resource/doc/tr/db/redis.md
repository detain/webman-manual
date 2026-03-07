# Redis

[webman/redis](https://github.com/webman-php/redis), [illuminate/redis](https://github.com/illuminate/redis) üzerine bağlantı havuzu ekleyerek genişletir; hem coroutine hem de coroutine olmayan ortamlarda çalışır. Kullanımı Laravel ile aynıdır.

`illuminate/redis` kullanmadan önce `php-cli` için redis eklentisini kurmanız gerekir.

## Kurulum

```php
composer require -W webman/redis illuminate/events
```

Kurulumdan sonra yeniden başlatma (restart) gerekir (reload yeterli değildir).

## Yapılandırma

Redis yapılandırma dosyası `config/redis.php` içindedir:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // Bağlantı havuzu ayarları
            'max_connections' => 10,     // Havuzdaki maksimum bağlantı sayısı
            'min_connections' => 1,      // Havuzdaki minimum bağlantı sayısı
            'wait_timeout' => 3,         // Bağlantı alma için maksimum bekleme süresi (saniye)
            'idle_timeout' => 50,        // Boşta kalma süresi; sonrasında min_connections'a kadar serbest bırakılır
            'heartbeat_interval' => 50,  // Heartbeat aralığı (60 saniyeyi aşmamalı)
        ],
    ]
];
```

## Bağlantı havuzu

* Her işlemin kendi havuzu vardır; havuzlar işlemler arasında paylaşılmaz.
* Coroutine kapalıyken çalışma sıralı olduğundan en fazla bir bağlantı kullanılır.
* Coroutine açıkken çalışma paraleldir ve havuz `min_connections` ile `max_connections` arasında ölçeklenir.
* Redis kullanan coroutine sayısı `max_connections`ı aşarsa en fazla `wait_timeout` saniye beklenir; sonrasında istisna fırlatılır.
* Boşta (coroutine olsun olmasın) bağlantılar `idle_timeout` sonrası `min_connections`a kadar serbest bırakılır (0 kabul edilir).

## Örnek

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## Redis API

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

Şununla eşdeğerdir:

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **Dikkat**
> `Redis::select($db)` arayüzünü dikkatli kullanın. webman sürekli bellekte çalışan bir çerçeve olduğundan, bir istekte veritabanı değiştirmek sonraki istekleri etkiler. Birden fazla veritabanı için her `$db` için ayrı Redis bağlantıları yapılandırmanız önerilir.

## Birden fazla Redis bağlantısı kullanma

`config/redis.php` yapılandırma dosyasında örnek:

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

Varsayılan olarak `default` altındaki bağlantı kullanılır. `Redis::connection()` ile hangi Redis bağlantısının kullanılacağını seçebilirsiniz:

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## Küme yapılandırması

Uygulamanız Redis kümesi kullanıyorsa, yapılandırmada `clusters` anahtarı altında tanımlayın:

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

Varsayılan olarak küme, düğümlerde istemci tarafı parçalama yapabilir; havuz oluşturma ve büyük bellek kullanımı sağlar. İstemci parçalama hata durumlarını ele almaz; özellikle başka bir ana veritabanından önbellek verisi almak için uygundur. Redis yerel kümesi için yapılandırmada `options` altında şunu belirtin:

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## Pipeline komutları

Tek bir işlemde sunucuya çok sayıda komut göndermeniz gerektiğinde pipeline kullanın. `pipeline` metodu bir closure alır; tüm komutlar tek işlemde çalıştırılır:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
