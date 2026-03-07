# ENV Bileşeni vlucas/phpdotenv

## Açıklama
`vlucas/phpdotenv`, farklı ortamların (geliştirme, test vb.) yapılandırmasını ayırt etmek için kullanılan bir ortam değişkeni yükleme bileşenidir.

## Proje adresi

https://github.com/vlucas/phpdotenv
  
## Kurulum
 
```php
composer require vlucas/phpdotenv
 ```
  
## Kullanım

### Proje kök dizininde yeni `.env` dosyası oluşturun
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Yapılandırma dosyasını değiştirin
**config/database.php**
```php
return [
    // Varsayılan veritabanı
    'default' => 'mysql',

    // Çeşitli veritabanı yapılandırmaları
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **İpucu**
> `.env` dosyasını `.gitignore` listesine eklemeniz önerilir, böylece depolarına gönderilmez. Depoda bir `.env.example` örnek yapılandırma dosyası bulundurun. Projeyi dağıtırken `.env.example` dosyasını `.env` olarak kopyalayıp mevcut ortama göre yapılandırmayı düzenleyin. Böylece proje ortama göre farklı yapılandırmalar yükler.

> **Not**
> `vlucas/phpdotenv` PHP TS sürümünde (Thread Safe) hata verebilir. Lütfen NTS sürümünü (Non-Thread-Safe) kullanın. Mevcut PHP sürümünü `php -v` komutuyla kontrol edebilirsiniz.

## Daha fazla bilgi

https://github.com/vlucas/phpdotenv adresini ziyaret edin.
  
