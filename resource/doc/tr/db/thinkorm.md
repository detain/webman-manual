# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm), [top-think/think-orm](https://github.com/top-think/think-orm) tabanlı bir veritabanı bileşenidir. Bağlantı havuzunu destekler ve hem korutin hem de korutin dışı ortamlarda çalışır.

## Kurulum

`composer require -W webman/think-orm`

Kurulumdan sonra restart (yeniden başlatma) gereklidir (reload geçerli değildir).

## Yapılandırma dosyası

Gerçek ihtiyaçlarınıza göre `config/think-orm.php` yapılandırma dosyasını düzenleyin.

## Belgeler

https://www.kancloud.cn/manual/think-orm

## Kullanım

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## Model oluşturma

think-orm modelleri `support\think\Model` sınıfından türetilir, aşağıdaki gibi:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Aşağıdaki komutla da think-orm modelleri oluşturabilirsiniz:

```
php webman make:model tablo_adi
```

> **İpucu**
> Bu komut için `webman/console` gereklidir. Kurulum: `composer require webman/console ^1.2.13`

> **Dikkat**
> make:model komutu ana projenin `illuminate/database` kullandığını algılarsa, think-orm yerine Illuminate tabanlı model dosyaları oluşturur. Bu durumda `tp` parametresini ekleyin: `php webman make:model tablo_adi tp` (çalışmazsa `webman/console` güncelleyin)
