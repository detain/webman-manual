# Medoo Veritabanı

[webman/medoo](https://github.com/webman-php/medoo), [Medoo](https://medoo.in/)'ya bağlantı havuzu desteği ekler ve hem korutin hem korutin dışı ortamlarda çalışır. Kullanımı Medoo ile aynıdır.

## Kurulum
`composer require webman/medoo`

## Medoo Veritabanı Yapılandırması
Yapılandırma dosyası konumu: `config/plugin/webman/medoo/database.php`

## Medoo Veritabanı Kullanımı
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **İpucu**
> `Medoo::get('user', '*', ['uid' => 1]);`
> şuyla aynıdır:
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`

## Medoo Çoklu Veritabanı Yapılandırması

**Yapılandırma**
`config/plugin/webman/medoo/database.php` dosyasına yeni bir yapılandırma ekleyin; anahtar herhangi olabilir, burada `other` kullanılmıştır.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // Bağlantı havuzu yapılandırması
            'max_connections' => 5, // Maksimum bağlantı sayısı
            'min_connections' => 1, // Minimum bağlantı sayısı
            'wait_timeout' => 60,   // Havuzdan bağlantı alırken maksimum bekleme süresi; aşılırsa istisna fırlatılır
            'idle_timeout' => 3,    // Havuzdaki bağlantıların maksimum boşta kalma süresi; aşan bağlantılar kapatılarak min_connections'e inilir
            'heartbeat_interval' => 50, // Havuz heartbeat aralığı (saniye); 60 saniyeden az önerilir
        ]
    ],
    // Burada 'other' yapılandırması eklenir
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo Veritabanı Kullanımı
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

Bkz. [Medoo resmi belgeleri](https://medoo.in/api/select)
