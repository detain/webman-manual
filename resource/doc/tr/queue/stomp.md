# Stomp Kuyruğu

Stomp, STOMP istemcilerinin herhangi bir STOMP mesaj aracısı (Broker) ile iletişim kurmasını sağlayan, birlikte çalışabilir bir bağlantı biçimi sunan basit (akış) metin tabanlı mesaj protokolüdür. [workerman/stomp](https://github.com/walkor/stomp), RabbitMQ, Apollo, ActiveMQ vb. mesaj kuyruğu senaryoları için başlıca kullanılan Stomp istemcisini gerçekleştirir.

## Kurulum
`composer require webman/stomp`

## Yapılandırma
Yapılandırma dosyası `config/plugin/webman/stomp` altındadır.

## Mesaj Gönderme
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Kuyruk
        $queue = 'examples';
        // Veri (dizi gönderirken kendi serializasyonunuzu yapmanız gerekir, örn. json_encode, serialize vb.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Gönderimi gerçekleştir
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Diğer projelerle uyumluluk için Stomp bileşeni otomatik serializasyon ve deserializasyon sunmaz. Dizi verisi gönderiyorsanız, kendiniz serializasyon yapmalı ve tüketirken kendiniz deserializasyon yapmalısınız.

## Mesaj Tüketme
Yeni `app/queue/stomp/MyMailSend.php` dosyası oluşturun (sınıf adı PSR-4 kurallarına uyduğu sürece serbesttir).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Kuyruk adı
    public $queue = 'examples';

    // Bağlantı adı, stomp.php dosyasındaki bağlantıya karşılık gelir
    public $connection = 'default';

    // Değer client ise sunucuya başarıyla tüketildiğini bildirmek için $ack_resolver->ack() çağrılmalıdır
    // Değer auto ise $ack_resolver->ack() çağrısı gerekmez
    public $ack = 'auto';

    // Tüketme
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Veri dizi ise kendiniz deserializasyon yapmalısınız
        var_export(json_decode($data, true)); // ['to' => 'tom@gmail.com', 'content' => 'hello'] çıktısı
        // Sunucuya başarıyla tüketildiğini bildir
        $ack_resolver->ack(); // ack auto olduğunda bu çağrı atlanabilir
    }
}
```

# RabbitMQ'da Stomp Protokolünü Etkinleştirme
RabbitMQ varsayılan olarak Stomp protokolünü etkinleştirmez. Etkinleştirmek için aşağıdaki komutu çalıştırın:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Etkinleştirdikten sonra Stomp'un varsayılan portu 61613'tür.
