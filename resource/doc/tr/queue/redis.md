# Redis Kuyruğu

Redis tabanlı mesaj kuyruğu, mesajların gecikmeli işlenmesini destekler.

## Kurulum
`composer require webman/redis-queue`

## Yapılandırma Dosyası
Redis yapılandırma dosyası `{ana-proje}/config/plugin/webman/redis-queue/redis.php` konumunda otomatik oluşturulur, içeriği şöyledir:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Şifre, isteğe bağlı
            'db' => 0,            // Veritabanı
            'max_attempts'  => 5, // Tüketim başarısız olduğunda tekrar deneme sayısı
            'retry_seconds' => 5, // Tekrar deneme aralığı (saniye)
        ]
    ],
];
```

### Başarısız Tüketimde Tekrar Deneme
Tüketim başarısız olursa (istisna oluşursa) mesaj gecikmeli kuyruğa konur ve bir sonraki denemeyi bekler. Tekrar deneme sayısı `max_attempts`, aralık `retry_seconds` ve `max_attempts` ile birlikte kontrol edilir. Örn. `max_attempts` 5, `retry_seconds` 10 ise 1. deneme aralığı `1*10` sn, 2. `2*10` sn, 3. `3*10` sn… 5 denemeye kadar. `max_attempts` ayarı aşılırsa mesaj `{redis-queue}-failed` anahtarlı başarısız kuyruğa gider.

## Mesaj Gönderme (Senkron)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Kuyruk adı
        $queue = 'send-mail';
        // Veri, doğrudan dizi olarak geçirilebilir, serileştirme gerekmez
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Mesaj gönder
        Redis::send($queue, $data);
        // Gecikmeli mesaj gönder, 60 saniye sonra işlenir
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
Başarılı gönderimde `Redis::send()` true, aksi halde false veya istisna döner.

> **İpucu**
> Gecikmeli kuyruk tüketim zamanında sapma olabilir. Örn. tüketim hızı üretim hızından düşükse kuyruk birikebilir ve tüketim gecikebilir. Azaltmak için daha fazla tüketim süreci çalıştırın.

## Mesaj Gönderme (Asenkron)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Kuyruk adı
        $queue = 'send-mail';
        // Veri, doğrudan dizi olarak geçirilebilir, serileştirme gerekmez
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Mesaj gönder
        Client::send($queue, $data);
        // Gecikmeli mesaj gönder, 60 saniye sonra işlenir
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` değer döndürmez. Asenkron push'dur ve Redis'e %100 iletim garantisi vermez.

> **İpucu**
> `Client::send()` mantığı yerel bellekte bir bellek kuyruğu oluşturup mesajları Redis'e asenkron senkronize etmektir (senkronizasyon hızlı, saniyede ~10.000 mesaj). Süreç yeniden başlarsa ve yerel bellek kuyruğundaki veriler tam senkronize olmamışsa mesaj kaybı olabilir. `Client::send()` asenkron gönderim kritik olmayan mesajlar için uygundur.

> **İpucu**
> `Client::send()` asenkrondur ve yalnızca Workerman çalışma ortamında kullanılabilir. Komut satırı betikleri için senkron arayüz `Redis::send()` kullanın.

## Diğer Projelerden Mesaj Gönderme
Bazen diğer projelerden mesaj göndermeniz gerekir ve `webman\redis-queue` kullanamazsınız. Bu durumda kuyruğa mesaj göndermek için aşağıdaki işlevi kullanabilirsiniz.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Burada `$redis` parametresi Redis örneğidir. Örn. redis eklentisi kullanımı:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Tüketim
Tüketim süreci yapılandırma dosyası `{ana-proje}/config/plugin/webman/redis-queue/process.php` konumundadır. Tüketici dizini `{ana-proje}/app/queue/redis/` altındadır.

`php webman redis-queue:consumer my-send-mail` komutu `{ana-proje}/app/queue/redis/MyMailSend.php` dosyasını oluşturur.

> **İpucu**
> Bu komut [Konsol](../plugin/console.md) eklentisinin kurulmasını gerektirir. Kurmak istemezseniz aşağıdakine benzer bir dosyayı manuel oluşturabilirsiniz:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Tüketilecek kuyruk adı
    public $queue = 'send-mail';

    // Bağlantı adı, plugin/webman/redis-queue/redis.php içindeki bağlantıya karşılık gelir
    public $connection = 'default';

    // Tüketim
    public function consume($data)
    {
        // Seriden çıkarma gerekmez
        var_export($data); // Çıktı ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Tüketim başarısızlığı geri çağrısı
    /* 
    $package = [
        'id' => 1357277951, // Mesaj ID
        'time' => 1709170510, // Mesaj zamanı
        'delay' => 0, // Gecikme süresi
        'attempts' => 2, // Tüketim sayısı
        'queue' => 'send-mail', // Kuyruk adı
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Mesaj içeriği
        'max_attempts' => 5, // Maksimum tekrar deneme
        'error' => 'Hata mesajı' // Hata mesajı
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Seriden çıkarma gerekmez
        var_export($package); 
    }
}
```

> **Not**
> Tüketim sırasında istisna veya Error fırlatılmazsa tüketim başarılı sayılır; aksi halde başarısız ve mesaj tekrar deneme kuyruğuna girer. redis-queue'da ack mekanizması yoktur; otomatik ack olarak düşünebilirsiniz (istisna veya Error olmadığında). Mevcut mesajı başarısız tüketilmiş işaretlemek için manuel istisna fırlatarak tekrar deneme kuyruğuna gönderebilirsiniz. Pratikte ack mekanizmasından farkı yoktur.

> **İpucu**
> Tüketiciler çoklu sunucu ve süreç destekler, aynı mesaj **iki kez** tüketilmez. Tüketilen mesajlar kuyruktan otomatik silinir; manuel silmeye gerek yoktur.

> **İpucu**
> Tüketim süreçleri birden fazla farklı kuyruğu aynı anda tüketebilir. Yeni kuyruk eklemek `process.php` içindeki yapılandırmayı değiştirmez. Yeni kuyruk tüketicisi eklerken `app/queue/redis` altına ilgili `Consumer` sınıfını ekleyin ve `$queue` özelliğiyle tüketilecek kuyruk adını belirtin.

> **İpucu**
> Windows kullanıcıları webman başlatmak için `php windows.php` çalıştırmalıdır, aksi halde tüketim süreci başlamaz.

> **İpucu**
> onConsumeFailure geri çağrısı her tüketim başarısızlığında tetiklenir. Başarısızlık sonrası mantığı burada işleyebilirsiniz. (Bu özellik `webman/redis-queue>=1.3.2` ve `workerman/redis-queue>=1.2.1` gerektirir)

## Farklı Kuyruklar İçin Farklı Tüketim Süreçleri
Varsayılan olarak tüm tüketiciler aynı süreci paylaşır. Bazen bazı kuyrukların tüketimini ayırmak gerekir—örn. yavaş tüketim işini bir süreç grubunda, hızlı tüketimi başka grupta. Bunun için tüketicileri iki dizine ayırabilirsiniz, örn. `app_path() . '/queue/redis/fast'` ve `app_path() . '/queue/redis/slow'` (tüketici sınıfının namespace'i buna göre güncellenmelidir). Yapılandırma:
```php
return [
    ...diğer yapılandırmalar atlandı...
    
    'redis_consumer_fast'  => [ // Anahtar özeldir, format kısıtı yok, burada redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Tüketici sınıfı dizini
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // Anahtar özeldir, format kısıtı yok, burada redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Tüketici sınıfı dizini
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Böylece hızlı iş tüketicileri `queue/redis/fast` dizinine, yavaş tüketiciler `queue/redis/slow` dizinine gider; kuyruklara tüketim süreçleri atanmış olur.

## Çoklu Redis Yapılandırması
#### Yapılandırma
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Şifre, dize tipi, isteğe bağlı
            'db' => 0,            // Veritabanı
            'max_attempts'  => 5, // Tüketim başarısız olduğunda tekrar deneme
            'retry_seconds' => 5, // Tekrar deneme aralığı (saniye)
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Şifre, dize tipi, isteğe bağlı
            'db' => 0,            // Veritabanı
            'max_attempts'  => 5, // Tüketim başarısız olduğunda tekrar deneme
            'retry_seconds' => 5, // Tekrar deneme aralığı (saniye)
        ]
    ],
];
```

Yapılandırmaya `other` anahtarlı ek bir Redis yapılandırması eklenmiştir.

#### Çoklu Redis'e Mesaj Gönderme

```php
// Anahtarı `default` olan kuyruğa mesaj gönder
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Aynısı
Client::send($queue, $data);
Redis::send($queue, $data);

// Anahtarı `other` olan kuyruğa mesaj gönder
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Çoklu Redis'ten Tüketim
Yapılandırmada anahtarı `other` olan kuyruktan mesaj tüketme:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Tüketilecek kuyruk adı
    public $queue = 'send-mail';

    // === Yapılandırmada anahtarı 'other' olan kuyruğu tüketmek için burada 'other' ayarlayın ===
    public $connection = 'other';

    // Tüketim
    public function consume($data)
    {
        // Seriden çıkarma gerekmez
        var_export($data);
    }
}
```

## Sık Sorulan Sorular

**`Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` hatası neden oluşur?**

Bu hata yalnızca asenkron gönderim arayüzü `Client::send()` ile oluşur. Asenkron gönderim önce mesajları yerel bellekte saklar, süreç boştayken Redis'e gönderir. Redis mesajları üretim hızından daha yavaş alırsa veya süreç diğer işlerle meşgulse ve bellekteki mesajları Redis'e senkronize edecek zaman bulamazsa mesaj birikimi olabilir. 600 saniyeden fazla birikme olursa bu hata tetiklenir.

Çözüm: Mesaj gönderimi için senkron gönderim arayüzü `Redis::send()` kullanın.
