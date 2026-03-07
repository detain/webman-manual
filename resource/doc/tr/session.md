# webman Oturum Yönetimi

## Örnek
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

`$request->session()` ile `Workerman\Protocols\Http\Session` örneğini alın ve örnek yöntemleriyle oturum verilerini ekleyin, değiştirin veya silin.

> **Not**
> Oturum nesnesi yok edildiğinde oturum verileri otomatik kaydedilir.
> Oturum nesnesini global bir değişkende tutmak nesnenin yok edilmesini engeller ve otomatik kaydı önler. Bu durumda veriyi kaydetmek için manuel olarak `$session->save()` çağırmanız gerekir.

## Tüm oturum verilerini almak
```php
$session = $request->session();
$all = $session->all();
```
Bir dizi döndürür. Oturum verisi yoksa boş dizi döner.


## Oturumdan bir değer almak
```php
$session = $request->session();
$name = $session->get('name');
```
Veri yoksa null döner.

get yöntemine ikinci argüman olarak varsayılan değer verebilirsiniz. Oturum dizisinde ilgili değer bulunamazsa varsayılan değer döner. Örneğin:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Oturum verisi kaydetmek
Tek bir veri kaydetmek için set yöntemini kullanın.
```php
$session = $request->session();
$session->set('name', 'tom');
```
set yöntemi bir değer döndürmez. Oturum nesnesi yok edildiğinde oturum otomatik kaydedilir.

Birden çok değer kaydetmek için put yöntemini kullanın.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Benzer şekilde put yöntemi de bir değer döndürmez.

## Oturum verisini silmek
Bir veya birden fazla oturum verisini silmek için `forget` yöntemini kullanın.
```php
$session = $request->session();
// Bir öğe sil
$session->forget('name');
// Birden fazla öğe sil
$session->forget(['name', 'age']);
```

Sistem ayrıca delete yöntemini sağlar. forget'tan farklı olarak yalnızca bir öğe silebilir.
```php
$session = $request->session();
// $session->forget('name'); ile aynı
$session->delete('name');
```


## Oturumdan değer alıp silmek
```php
$session = $request->session();
$name = $session->pull('name');
```
Aşağıdaki kodla aynı etkiye sahiptir:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
İlgili oturum verisi yoksa null döner.


## Tüm oturum verilerini silmek
```php
$request->session()->flush();
```
Değer döndürmez. Oturum nesnesi yok edildiğinde oturum otomatik olarak depolamadan kaldırılır.


## Oturum değerinin varlığını kontrol etmek
```php
$session = $request->session();
$has = $session->has('name');
```
Oturum değeri yoksa veya null ise false, aksi halde true döner.

```php
$session = $request->session();
$has = $session->exists('name');
```
Yukarıdaki kod da oturum değerinin varlığını kontrol eder. Fark: değer null olsa bile `exists` true döner.

## Yardımcı işlev session()

webman aynı işlevi yerine getirmek için `session()` yardımcı işlevini sunar.
```php
// Oturum örneğini al
$session = session();
// Aynı
$session = $request->session();

// Değer al
$value = session('key', 'default');
// Aynı
$value = session()->get('key', 'default');
// Aynı
$value = $request->session()->get('key', 'default');

// Oturuma değer ata
session(['key1'=>'value1', 'key2' => 'value2']);
// Aynı
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Aynı
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Yapılandırma dosyası
Oturum yapılandırma dosyası `config/session.php` konumundadır. İçerik buna benzer:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class veya RedisSessionHandler::class veya RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // handler FileSessionHandler::class ise değer 'file',
    // handler RedisSessionHandler::class ise değer 'redis'
    // handler RedisClusterSessionHandler::class ise değer 'redis_cluster' (Redis kümesi)
    'type'    => 'file',

    // Farklı handler'lar farklı yapılandırma kullanır
    'config' => [
        // type 'file' için yapılandırma
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // type 'redis' için yapılandırma
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // session_id saklayan çerez adı
    'auto_update_timestamp' => false,  // Oturumu otomatik yenile, varsayılan: kapalı
    'lifetime' => 7*24*60*60,          // Oturum süresi
    'cookie_lifetime' => 365*24*60*60, // session_id çerez süresi
    'cookie_path' => '/',              // session_id çerez yolu
    'domain' => '',                    // session_id çerez alan adı
    'http_only' => true,               // httpOnly aç, varsayılan: açık
    'secure' => false,                 // Oturumu yalnızca HTTPS'te aç, varsayılan: kapalı
    'same_site' => '',                 // CSRF ve kullanıcı takibini önle, değerler: strict/lax/none
    'gc_probability' => [1, 1000],     // Oturum çöp toplama olasılığı
];
```

## Güvenlik
Oturumda sınıf örneklerini doğrudan saklamak önerilmez, özellikle güvenilir olmayan kaynaklardan gelenler. Ters serileştirme güvenlik riski oluşturabilir.

