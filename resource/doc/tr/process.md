# Özel süreçler

webman'de, workerman'da olduğu gibi özel dinleyiciler veya süreçler oluşturabilirsiniz.

> **Not**
> Windows kullanıcılarının özel süreçleri çalıştırmak için webman'ı `php windows.php` ile başlatmaları gerekir.

## Özel HTTP servisi
Bazen webman HTTP servisinin temel kodunu değiştirmenizi gerektiren özel bir ihtiyacınız olabilir. Bu durumda özel süreç kullanabilirsiniz.

Örneğin, `app\Server.php` dosyasını oluşturun.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Burada Webman\App içindeki yöntemleri geçersiz kılın
}
```

`config/process.php` içine aşağıdaki yapılandırmayı ekleyin.

```php
use Workerman\Worker;

return [
    // ... diğer yapılandırmalar atlandı ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Süreç sayısı
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // İstek sınıfını ayarlayın
            'logger' => \support\Log::channel('default'), // Günlük örneği
            'appPath' => app_path(), // app dizini konumu
            'publicPath' => public_path() // public dizini konumu
        ]
    ]
];
```

> **İpucu**
> webman'ın yerleşik HTTP sürecini kapatmak için config/server.php içinde `listen=>''` ayarlamanız yeterlidir.

## Özel WebSocket dinleyici örneği

`app/Pusher.php` dosyasını oluşturun.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> Not: Tüm onXXX yöntemleri public olmalıdır.

`config/process.php` içine aşağıdaki yapılandırmayı ekleyin.

```php
return [
    // ... diğer süreç yapılandırmaları atlandı ...
    
    // websocket_test sürecin adıdır
    'websocket_test' => [
        // Burada süreç sınıfını belirtin, yukarıda tanımlanan Pusher sınıfı
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Özel dinlemesiz süreç örneği

`app/TaskTest.php` dosyasını oluşturun.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Her 10 saniyede veritabanında yeni kullanıcı kayıtlarını kontrol edin
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

`config/process.php` içine aşağıdaki yapılandırmayı ekleyin.

```php
return [
    // ... diğer süreç yapılandırmaları atlandı ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Not: listen atlanırsa hiçbir port dinlenmez; count atlanırsa varsayılan süreç sayısı 1 olur.

## Yapılandırma dosyası açıklaması

Bir sürecin tam yapılandırması şu şekilde tanımlanır:

```php
return [
    // ... 
    
    // websocket_test sürecin adıdır
    'websocket_test' => [
        // Burada süreç sınıfını belirtin
        'handler' => app\Pusher::class,
        // Dinlenecek protokol, IP ve port (isteğe bağlı)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Süreç sayısı (isteğe bağlı, varsayılan 1)
        'count'   => 2,
        // Süreci çalıştıracak kullanıcı (isteğe bağlı, varsayılan mevcut kullanıcı)
        'user'    => '',
        // Süreci çalıştıracak kullanıcı grubu (isteğe bağlı, varsayılan mevcut grup)
        'group'   => '',
        // Mevcut sürecin reload destekleyip desteklemediği (isteğe bağlı, varsayılan true)
        'reloadable' => true,
        // reusePort etkinleştir
        'reusePort'  => true,
        // Transport (isteğe bağlı, SSL gerektiğinde 'ssl' olarak ayarlayın, varsayılan 'tcp')
        'transport'  => 'tcp',
        // Context (isteğe bağlı, transport 'ssl' olduğunda sertifika yolu geçirin)
        'context'    => [], 
        // Süreç sınıfı yapıcı parametreleri (isteğe bağlı)
        'constructor' => [],
        // Bu süreç etkin mi
        'enable' => true
    ],
];
```

## Sonuç

webman'ın özel süreçleri aslında workerman'ın basit bir sarmalayıcısıdır. Yapılandırmayı iş mantığından ayırır ve workerman'ın `onXXX` geri çağrılarını sınıf yöntemleriyle uygular. Diğer tüm kullanımlar workerman ile tamamen aynıdır.
