# Veritabanı

Çoğu eklenti [webman-admin](https://www.workerman.net/plugin/82) kurduğundan, `webman-admin` veritabanı yapılandırmasının doğrudan yeniden kullanılması önerilir.

Temel sınıfı `plugin\admin\app\model\Base` olan modeller, webman-admin veritabanı yapılandırmasını otomatik olarak kullanacaktır.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

webman-admin veritabanına `plugin.admin.mysql` üzerinden de erişebilirsiniz, örneğin:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Kendi Veritabanınızı Kullanma

Eklentiler kendi veritabanlarını kullanmayı da seçebilir. Örneğin, `plugin/foo/config/database.php` içeriği:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql bağlantı adıdır
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'veritabani',
            'username'    => 'kullanici_adi',
            'password'    => 'sifre',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin bağlantı adıdır
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'veritabani',
            'username'    => 'kullanici_adi',
            'password'    => 'sifre',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

Başvuru formatı: `Db::connection('plugin.{eklenti}.{bağlantı_adı}');`, örneğin:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Ana projenin veritabanını kullanmak için doğrudan çağırın:

```php
use support\Db;
Db::table('user')->first();
// Ana projenin admin bağlantısı da yapılandırıldığını varsayarak
Db::connection('admin')->table('admin')->first();
```

#### Model İçin Veritabanı Yapılandırma

Model için bir Base sınıfı oluşturup `$connection` özelliğini eklentinin kendi veritabanı bağlantısını kullanacak şekilde ayarlayabilirsiniz:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

Böylece eklentideki Base'den türeyen tüm modeller otomatik olarak eklentinin kendi veritabanını kullanır.

## Otomatik Veritabanı İçe Aktarma

`php webman app-plugin:create foo` çalıştırmak, foo eklentisini oluşturur; buna `plugin/foo/api/Install.php` ve `plugin/foo/install.sql` dahildir.

> **İpucu**
> install.sql dosyası oluşturulmuyorsa `webman/console` güncellemeyi deneyin: `composer require webman/console ^1.3.6`

#### Eklenti Kurulurken Veritabanını İçe Aktarma

Bir eklenti kurulduğunda Install.php içindeki `install` metodu çalışır ve `install.sql` içindeki SQL ifadelerini otomatik olarak yürütür; böylece veritabanı tabloları içe aktarılır.

`install.sql` içeriği tablo oluşturma ve geçmiş şema değişikliklerini içermelidir. Her ifade `;` ile bitmelidir, örneğin:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Birincil anahtar',
  `order_id` varchar(50) NOT NULL COMMENT 'Sipariş ID',
  `user_id` int NOT NULL COMMENT 'Kullanıcı ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Ödenecek tutar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Siparişler';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Birincil anahtar',
  `name` varchar(50) NOT NULL COMMENT 'Ad',
  `price` int NOT NULL COMMENT 'Fiyat',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ürünler';
```

**Veritabanı Bağlantısını Değiştirme**

Varsayılan olarak `install.sql`, webman-admin veritabanına içe aktarılır. Başka bir veritabanına içe aktarmak için Install.php içindeki `$connection` özelliğini değiştirin:

```php
<?php

class Install
{
    // Eklentinin kendi veritabanını belirtin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Test**

Eklentiyi kurmak için `php webman app-plugin:install foo` çalıştırın. Sonra veritabanını kontrol edin; `foo_orders` ve `foo_goods` tablolarının oluşturulmuş olması gerekir.

#### Eklenti Güncellemesinde Tablo Yapısını Değiştirme

Eklenti güncellemesi yeni tablolar veya şema değişiklikleri gerektirdiğinde, ilgili SQL ifadelerini `install.sql` sonuna ekleyin. Her ifade `;` ile bitmelidir. Örneğin, `foo_user` tablosu ve `foo_orders` tablosuna `status` sütunu ekleme:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Birincil anahtar',
  `order_id` varchar(50) NOT NULL COMMENT 'Sipariş ID',
  `user_id` int NOT NULL COMMENT 'Kullanıcı ID',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Ödenecek tutar',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Siparişler';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Birincil anahtar',
 `name` varchar(50) NOT NULL COMMENT 'Ad',
 `price` int NOT NULL COMMENT 'Fiyat',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Ürünler';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Birincil anahtar',
 `name` varchar(50) NOT NULL COMMENT 'Ad'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Kullanıcı';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Durum';
```

Güncellemede Install.php içindeki `update` metodu `install.sql` içindeki ifadeleri yürütür. Yeniler uygulanır; zaten uygulanmış olanlar atlanır; böylece veritabanı değişiklikleri güncellemelerde doğru şekilde uygulanır.

#### Eklenti Kaldırıldığında Veritabanını Silme

Bir eklenti kaldırıldığında Install.php içindeki `uninstall` metodu çağrılır. `install.sql` içindeki CREATE TABLE ifadelerini otomatik olarak analiz eder ve bu tabloları siler.

Yalnızca kendi `uninstall.sql` dosyanızı çalıştırmak ve otomatik tablo silmeyi devre dışı bırakmak istiyorsanız, `plugin/{eklenti_adı}/uninstall.sql` oluşturun. Bu durumda `uninstall` metodu yalnızca bu dosyadaki ifadeleri çalıştırır.
