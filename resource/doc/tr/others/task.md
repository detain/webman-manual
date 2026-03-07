# Yavaş İşlem İşleme

Bazen yavaş işlemleri işlememiz gerekir. Yavaş işlemlerin webman'daki diğer istek işlemlerini etkilememesi için, duruma göre farklı işleme çözümleri kullanılabilir.

## Seçenek 1: Mesaj Kuyruğu Kullanımı
[Redis kuyruğu](../queue/redis.md) [Stomp kuyruğu](../queue/stomp.md) sayfalarına bakınız

#### Avantajlar
Ani büyük ölçekli işlem isteklerine karşı kullanılabilir

#### Dezavantajlar
Sonuçları doğrudan istemciye döndürmek mümkün değildir. Sonuçları göndermek için diğer hizmetlerle koordinasyon gereklidir, örneğin [webman/push](https://www.workerman.net/plugin/2) ile işlem sonuçlarını göndermek.

## Seçenek 2: Yeni HTTP Portu Ekleme

Yavaş istekleri işlemek için yeni bir HTTP portu eklenir. Bu yavaş istekler bu porta erişerek belirli bir işlem grubu tarafından işlenir ve işlem sonrası sonuçlar doğrudan istemciye döner.

#### Avantajlar
Verileri doğrudan istemciye döndürebilir

#### Dezavantajlar
Ani büyük ölçekli isteklere karşı kullanılamaz

#### Uygulama Adımları
`config/process.php` içine aşağıdaki yapılandırmayı ekleyin.
```php
return [
    // ... diğer yapılandırmalar burada kısaltıldı ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // İşlem sayısı
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // İstek sınıfı ayarı
            'logger' => \support\Log::channel('default'), // Günlükçü örneği
            'appPath' => app_path(), // Uygulama dizini konumu
            'publicPath' => public_path() // Genel dizin konumu
        ]
    ]
];
```

Böylece yavaş arayüzler `http://127.0.0.1:8686/` işlem grubu üzerinden işlenebilir, diğer işlemlerin işlemlerini etkilemeden.

Ön ucun port farkını fark etmemesi için nginx'e 8686 portuna bir proxy eklenebilir. Yavaş arayüz istek yollarının hepsi `/task` ile başladığını varsayarsak, nginx yapılandırması şu şekilde olacaktır:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Yeni 8686 upstream ekle
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /task ile başlayan istekler 8686 portuna gider, ihtiyacınıza göre /task'ı değiştirin
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Diğer istekler orijinal 8787 portuna gider
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

Böylece istemci `domain.com/task/xxx` adresine eriştiğinde ayrı 8686 portu üzerinden işlenecek, 8787 portunun istek işlemesini etkilemeyecektir.

## Seçenek 3: HTTP Chunked ile Eşzamanlı Olmayan Bölümlenmiş Veri Gönderimi

#### Avantajlar
Verileri doğrudan istemciye döndürebilir

**workerman/http-client kurulumu**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // Boş chunk göndererek yanıtın sonunu belirtin
        });
        // Önce HTTP başlığı gönderin, sonraki veriler eşzamanlı olmayan şekilde gönderilir
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Not**
> Bu örnek `workerman/http-client` istemcisini HTTP sonuçlarını eşzamanlı olmayan şekilde alıp veri döndürmek için kullanır. [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html) gibi diğer eşzamanlı olmayan istemciler de kullanılabilir.
