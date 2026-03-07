# Yapılandırma Dosyaları

## Konum
webman yapılandırma dosyaları `config/` dizininde bulunur. Projenizde `config()` işlevini kullanarak ilgili yapılandırmalara erişebilirsiniz.

## Yapılandırmalara Erişim

Tüm yapılandırmaları alın:
```php
config();
```

`config/app.php` dosyasındaki tüm yapılandırmaları alın:
```php
config('app');
```

`config/app.php` dosyasındaki `debug` yapılandırmasını alın:
```php
config('app.debug');
```

Yapılandırma bir dizi ise, iç içe değerlere erişmek için `.` kullanabilirsiniz. Örnek:
```php
config('file.key1.key2');
```

## Varsayılan Değer
```php
config($key, $default);
```
Varsayılan değeri ikinci parametre olarak geçirin. Yapılandırma mevcut değilse varsayılan değer döndürülür. Yapılandırma mevcut değilse ve varsayılan ayarlanmamışsa `null` döndürülür.

## Özel Yapılandırma
Geliştiriciler `config/` dizinine kendi yapılandırma dosyalarını ekleyebilir. Örnek:

**config/payment.php**

```php
<?php
return [
    'key' => '...',
    'secret' => '...'
];
```

**Yapılandırmalara erişimde kullanım**
```php
config('payment');
config('payment.key');
config('payment.secret');
```

## Yapılandırmaları Değiştirme
Webman dinamik yapılandırma değişikliklerini desteklemez. Tüm yapılandırmalar ilgili yapılandırma dosyalarında manuel olarak değiştirilmeli, ardından uygulama yeniden yüklenmeli veya yeniden başlatılmalıdır.

> **Not**
> Sunucu yapılandırması `config/server.php` ve süreç yapılandırması `config/process.php` yeniden yüklemeyi desteklemez. Değişikliklerin etkili olması için yeniden başlatmanız gerekir.

## Önemli Hatırlatma
`config/` altında bir alt dizinde yapılandırma dosyaları oluşturursanız, örneğin `config/order/status.php`, `config/order/` dizininde aşağıdaki içeriğe sahip bir `app.php` dosyasına ihtiyacınız vardır:
```php
<?php
return [
    'enable' => true,
];
```
`enable` değerinin `true` olarak ayarlanması, çerçevenin bu dizinden yapılandırmaları yüklemesini söyler.
Yapılandırma dizin yapısı şöyle görünmelidir:
```
├── config
│   ├── order
│   │   ├── app.php
│   │   └── status.php
```
Ardından `status.php` dosyasındaki dizi veya belirli anahtarlara `config.order.status` üzerinden erişebilirsiniz.


## Yapılandırma Dosyası Referansı

#### server.php
```php
return [
    'listen' => 'http://0.0.0.0:8787', // Dinleme portu (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'transport' => 'tcp', // Aktarım protokolü (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'context' => [], // SSL vb. (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'name' => 'webman', // Süreç adı (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'count' => cpu_count() * 4, // Süreç sayısı (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'user' => '', // Kullanıcı (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'group' => '', // Grup (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'reusePort' => false, // Port yeniden kullanımını etkinleştir (1.6.0'dan itibaren kaldırıldı, config/process.php'de yapılandırılıyor)
    'event_loop' => '',  // Olay döngüsü sınıfı, varsayılan olarak otomatik seçilir
    'stop_timeout' => 2, // Durdurma/yeniden başlatma/yeniden yükleme sinyali alındığında maksimum bekleme süresi, süreç zamanında çıkmazsa zorla çıkış
    'pid_file' => runtime_path() . '/webman.pid', // PID dosyası konumu
    'status_file' => runtime_path() . '/webman.status', // Durum dosyası konumu
    'stdout_file' => runtime_path() . '/logs/stdout.log', // stdout dosyası konumu, webman başladıktan sonraki tüm çıktı buraya yazılır
    'log_file' => runtime_path() . '/logs/workerman.log', // Workerman günlük dosyası konumu
    'max_package_size' => 10 * 1024 * 1024 // Maksimum paket boyutu, 10M. Yüklenen dosya boyutu bununla sınırlıdır
];
```

#### app.php
```php
return [
    'debug' => true,  // Hata ayıklama modu, hatalarda yığın izlemesi vb. etkinleştirir. Üretimde devre dışı bırakılmalıdır
    'error_reporting' => E_ALL, // Hata raporlama düzeyi
    'default_timezone' => 'Asia/Shanghai', // Varsayılan saat dilimi
    'public_path' => base_path() . DIRECTORY_SEPARATOR . 'public', // Genel dizin yolu
    'runtime_path' => base_path(false) . DIRECTORY_SEPARATOR . 'runtime', // Çalışma zamanı dizin yolu
    'controller_suffix' => 'Controller', // Denetleyici soneki
    'controller_reuse' => false, // Denetleyicilerin yeniden kullanılıp kullanılmayacağı
];
```

#### process.php
```php
use support\Log;
use support\Request;
use app\process\Http;
global $argv;

return [
     // webman süreç yapılandırması
    'webman' => [ 
        'handler' => Http::class, // Süreç işleyici sınıfı
        'listen' => 'http://0.0.0.0:8787', // Dinleme adresi
        'count' => cpu_count() * 4, // Süreç sayısı, varsayılan olarak CPU'nun 4 katı
        'user' => '', // Süreç kullanıcısı, düşük yetkili kullanıcı kullanılmalıdır
        'group' => '', // Süreç grubu, düşük yetkili grup kullanılmalıdır
        'reusePort' => false, // reusePort'u etkinleştir, bağlantıları worker süreçleri arasında dağıtır
        'eventLoop' => '', // Olay döngüsü sınıfı, boş olduğunda server.event_loop kullanılır
        'context' => [], // Dinleme bağlamı, örn. SSL
        'constructor' => [ // Süreç işleyici için kurucu parametreleri, burada Http sınıfı
            'requestClass' => Request::class, // Özel istek sınıfı
            'logger' => Log::channel('default'), // Günlükçü örneği
            'appPath' => app_path(), // Uygulama dizin yolu
            'publicPath' => public_path() // Genel dizin yolu
        ]
    ],
    // Dosya değişikliği otomatik yeniden yüklemesi ve bellek sızıntısı tespiti için izleme süreci
    'monitor' => [
        'handler' => app\process\Monitor::class, // İşleyici sınıfı
        'reloadable' => false, // Bu süreç yeniden yükleme yapmaz
        'constructor' => [ // Süreç işleyici için kurucu parametreleri
            // İzlenecek dizinler, çok fazla dizin tespiti yavaşlatır
            'monitorDir' => array_merge([
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // Değişiklikleri izlenecek dosya uzantıları
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            // Diğer seçenekler
            'options' => [
                // Dosya izlemeyi etkinleştir, yalnızca Linux, arka plan modunda dosya izlemesi varsayılan olarak devre dışıdır
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/',
                // Bellek izlemeyi etkinleştir, yalnızca Linux
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',
            ]
        ]
    ]
];
```

#### container.php
```php
// PSR-11 bağımlılık enjeksiyonu kabuğu örneği döndür
return new Webman\Container;
```

#### dependence.php
```php
// Bağımlılık enjeksiyonu kabuğunda servisleri ve bağımlılıkları yapılandır
return [];
```

#### route.php
```php

use support\Route;
// /test yolu için rota tanımla
Route::any('/test', function (Request $request) {
    return response('test');
});
```

#### view.php
```php
use support\view\Raw;
use support\view\Twig;
use support\view\Blade;
use support\view\ThinkPHP;

return [
    'handler' => Raw::class // Varsayılan görünüm işleyici sınıfı
];
```

### autoload.php
```php
// Çerçeve otomatik yükleme dosyalarını yapılandır
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php',
        base_path() . '/support/Response.php',
    ]
];
```

#### cache.php
```php
// Önbellek yapılandırması
return [
    'default' => 'file', // Varsayılan sürücü
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache') // Önbellek dosyası depolama yolu
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default' // Redis bağlantı adı, redis.php'deki yapılandırmayı ifade eder
        ],
        'array' => [
            'driver' => 'array' // Bellekte önbellek, yeniden başlatmada temizlenir
        ]
    ]
];
```

#### redis.php
```php
return [
    'default' => [
        'host' => '127.0.0.1',
        'password' => null,
        'port' => 6379,
        'database' => 0,
    ],
];
```

#### database.php
```php
return [
 // Varsayılan veritabanı
 'default' => 'mysql',
 // Veritabanı bağlantı yapılandırmaları
 'connections' => [

     'mysql' => [
         'driver'      => 'mysql',
         'host'        => '127.0.0.1',
         'port'        => 3306,
         'database'    => 'webman',
         'username'    => 'webman',
         'password'    => '',
         'unix_socket' => '',
         'charset'     => 'utf8',
         'collation'   => 'utf8_unicode_ci',
         'prefix'      => '',
         'strict'      => true,
         'engine'      => null,
     ],

     'sqlite' => [
         'driver'   => 'sqlite',
         'database' => '',
         'prefix'   => '',
     ],

     'pgsql' => [
         'driver'   => 'pgsql',
         'host'     => '127.0.0.1',
         'port'     => 5432,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
         'schema'   => 'public',
         'sslmode'  => 'prefer',
     ],

     'sqlsrv' => [
         'driver'   => 'sqlsrv',
         'host'     => 'localhost',
         'port'     => 1433,
         'database' => 'webman',
         'username' => 'webman',
         'password' => '',
         'charset'  => 'utf8',
         'prefix'   => '',
     ],
 ],
];
```

#### exception.php
```php
return [
    // İstisna işleyici sınıfını ayarla
    '' => support\exception\Handler::class,
];
```

#### log.php
```php
return [
    'default' => [
        'handlers' => [
            [
                'class' => Monolog\Handler\RotatingFileHandler::class, // İşleyici
                'constructor' => [
                    runtime_path() . '/logs/webman.log', // Günlük dosyası adı
                    7, //$maxFiles // Günlükleri 7 gün sakla
                    Monolog\Logger::DEBUG, // Günlük düzeyi
                ],
                'formatter' => [
                    'class' => Monolog\Formatter\LineFormatter::class, // Biçimlendirici
                    'constructor' => [null, 'Y-m-d H:i:s', true], // Biçimlendirici parametreleri
                ],
            ]
        ],
    ],
];
```

#### session.php
```php
return [
     // Tür
    'type' => 'file', // veya redis veya redis_cluster
     // İşleyici
    'handler' => FileSessionHandler::class,
     // Yapılandırma
    'config' => [
        'file' => [
            'save_path' => runtime_path() . '/sessions', // Depolama dizini
        ],
        'redis' => [
            'host' => '127.0.0.1',
            'port' => 6379,
            'auth' => '',
            'timeout' => 2,
            'database' => '',
            'prefix' => 'redis_session_',
        ],
        'redis_cluster' => [
            'host' => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth' => '',
            'prefix' => 'redis_session_',
        ]
    ],
    'session_name' => 'PHPSID', // Oturum adı
    'auto_update_timestamp' => false, // Oturum süresinin dolmasını önlemek için otomatik zaman damgası güncellemesi
    'lifetime' => 7*24*60*60, // Ömür
    'cookie_lifetime' => 365*24*60*60, // Çerez ömrü
    'cookie_path' => '/', // Çerez yolu
    'domain' => '', // Çerez alan adı
    'http_only' => true, // Yalnızca HTTP
    'secure' => false, // Yalnızca HTTPS
    'same_site' => '', // SameSite özniteliği
    'gc_probability' => [1, 1000], // Oturum çöp toplama olasılığı
];
```

#### middleware.php
```php
// Ara katmanı yapılandır
return [];
```

#### static.php
```php
return [
    'enable' => true, // webman statik dosya sunumunu etkinleştir
    'middleware' => [ // Önbellek politikası, CORS vb. için statik dosya ara katmanı
        //app\middleware\StaticFile::class,
    ],
];
```

#### translation.php
```php
return [
    // Varsayılan dil
    'locale' => 'zh_CN',
    // Yedek diller
    'fallback_locale' => ['zh_CN', 'en'],
    // Çeviri dosyası konumu
    'path' => base_path() . '/resource/translations',
];
```
