# İşlemlerin Doğru Kullanımı

webman'da veritabanı işlemleri diğer framework'lerdekiyle aynı şekilde kullanılır. İşte dikkat edilmesi gereken noktalar.

## Kod Yapısı

Kod yapısı diğer framework'lerle (örneğin Laravel, think-orm benzeri) aynıdır:

```php
Db::beginTransaction();
try {
    // ..iş mantığı atlandı...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Önemli:** `\Exception` yerine **mutlaka** `\Throwable` kullanılmalıdır, çünkü iş mantığı sırasında `Error` tetiklenebilir ve `Error`, `Exception` sınıfından türetilmez.

## Veritabanı Bağlantısı

İşlem içinde modelle çalışırken, modelin bağlantı ayarlayıp ayarlamadığına dikkat edin. Model bağlantı belirliyorsa, işlemi başlatırken bu bağlantıyı belirtmeniz gerekir; aksi halde işlem geçerli olmaz (think-orm benzeri). Örnek:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Model için bağlantı belirlendi
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Model bağlantı belirttiğinde, begin, commit ve rollback için bu bağlantıyı belirtmeniz gerekir:

```php
Db::connection('mysql')->beginTransaction();
try {
    // İş mantığı
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Onaylanmamış İşlem İçeren İstekleri Bulma

Bazen iş kodu hatası nedeniyle bir istekteki işlem onaylanmadan kalır. Onaylanmamış işlem içeren controller metodunu hızlıca bulmak için `webman/log` bileşenini kurabilirsiniz. Bu bileşen her istek tamamlandıktan sonra onaylanmamış işlem olup olmadığını otomatik kontrol eder ve günlüğe kaydeder. Günlük anahtar kelimesi `Uncommitted transactions`'dır.

**webman/log kurulumu**

`composer require webman/log`

> **Not**
> Kurulumdan sonra **restart** ile yeniden başlatma gerekir; reload yeterli olmaz.
