# قاعدة البيانات

نظرًا لأن معظم الملحقات تقوم بتثبيت [webman-admin](https://www.workerman.net/plugin/82)، يُوصى بإعادة استخدام تكوين قاعدة بيانات `webman-admin` مباشرةً.

النماذج التي أساسها `plugin\admin\app\model\Base` ستستخدم تلقائيًا تكوين قاعدة بيانات webman-admin.
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

يمكنك أيضًا الوصول إلى قاعدة بيانات webman-admin عبر `plugin.admin.mysql`، على سبيل المثال:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## استخدام قاعدة البيانات الخاصة بك

يمكن للملحقات أيضًا اختيار استخدام قاعدة بيانات خاصة بها. على سبيل المثال، محتوى `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql هو اسم الاتصال
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'قاعدة_البيانات',
            'username'    => 'اسم_المستخدم',
            'password'    => 'كلمة_المرور',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin هو اسم الاتصال
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'قاعدة_البيانات',
            'username'    => 'اسم_المستخدم',
            'password'    => 'كلمة_المرور',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

صيغة المرجع هي `Db::connection('plugin.{الملحق}.{اسم_الاتصال}');`، على سبيل المثال:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

لاستخدام قاعدة بيانات المشروع الرئيسي، استدعها مباشرةً:

```php
use support\Db;
Db::table('user')->first();
// بافتراض أن المشروع الرئيسي لديه أيضًا اتصال admin مُعد
Db::connection('admin')->table('admin')->first();
```

#### تكوين قاعدة البيانات للنموذج (Model)

يمكنك إنشاء فئة Base للنموذج وتحديد الخاصية `$connection` لاستخدام اتصال قاعدة بيانات الملحق نفسه:

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

بهذا الشكل، ستستخدم جميع النماذج في الملحق التي ترث من Base تلقائيًا قاعدة بيانات الملحق نفسه.

## استيراد قاعدة البيانات تلقائيًا

تنفيذ `php webman app-plugin:create foo` ينشئ ملحق foo، بما في ذلك `plugin/foo/api/Install.php` و `plugin/foo/install.sql`.

> **نصيحة**
> إذا لم يتم إنشاء ملف install.sql، جرّب تحديث `webman/console`: `composer require webman/console ^1.3.6`

#### استيراد قاعدة البيانات عند تثبيت الملحق

عند تثبيت ملحق، يتم تنفيذ الدالة `install` في Install.php والتي تشغّل تلقائيًا عبارات SQL في `install.sql`، مستوردةً بذلك جداول قاعدة البيانات.

يجب أن يتضمن محتوى `install.sql` إنشاء الجداول والتغييرات التاريخية للمخطط. كل عبارة يجب أن تنتهي بـ `;`، على سبيل المثال:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'المفتاح الأساسي',
  `order_id` varchar(50) NOT NULL COMMENT 'معرف الطلب',
  `user_id` int NOT NULL COMMENT 'معرف المستخدم',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'المبلغ المطلوب دفعه',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='الطلبات';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'المفتاح الأساسي',
  `name` varchar(50) NOT NULL COMMENT 'الاسم',
  `price` int NOT NULL COMMENT 'السعر',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='المنتجات';
```

**تغيير اتصال قاعدة البيانات**

بشكل افتراضي، يتم استيراد `install.sql` إلى قاعدة بيانات webman-admin. للاستيراد إلى قاعدة بيانات أخرى، عدّل الخاصية `$connection` في `Install.php`:

```php
<?php

class Install
{
    // تحديد قاعدة بيانات الملحق نفسها
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**الاختبار**

نفّذ `php webman app-plugin:install foo` لتثبيت الملحق. ثم تحقق من قاعدة البيانات: يجب أن تكون جداول `foo_orders` و `foo_goods` قد أُنشئت.

#### تغيير هيكل الجداول عند ترقية الملحق

عندما تتطلب ترقية الملحق جداول جديدة أو تغييرات في المخطط، أضف العبارات SQL المناسبة في نهاية `install.sql`. كل عبارة يجب أن تنتهي بـ `;`. على سبيل المثال، إضافة جدول `foo_user` وعمود `status` إلى `foo_orders`:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'المفتاح الأساسي',
  `order_id` varchar(50) NOT NULL COMMENT 'معرف الطلب',
  `user_id` int NOT NULL COMMENT 'معرف المستخدم',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'المبلغ المطلوب دفعه',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='الطلبات';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'المفتاح الأساسي',
 `name` varchar(50) NOT NULL COMMENT 'الاسم',
 `price` int NOT NULL COMMENT 'السعر',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='المنتجات';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'المفتاح الأساسي',
 `name` varchar(50) NOT NULL COMMENT 'الاسم'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='المستخدم';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'الحالة';
```

عند الترقية، الدالة `update` في Install.php تنفذ العبارات من `install.sql`. الجديدة تُنفّذ؛ القديمة المُطبّقة تُتخطى، مُطبقةً بذلك تغييرات قاعدة البيانات بشكل صحيح عند الترقيات.

#### حذف قاعدة البيانات عند إلغاء تثبيت الملحق

عند إلغاء تثبيت ملحق، يتم استدعاء الدالة `uninstall` في Install.php. تقوم بتحليل تلقائي لعبارات CREATE TABLE في `install.sql` وحذف تلك الجداول.

إذا كنت تريد تنفيذ `uninstall.sql` الخاص بك فقط بدلاً من حذف الجداول التلقائي، أنشئ `plugin/{اسم_الملحق}/uninstall.sql`. في هذه الحالة، الدالة `uninstall` تنفذ فقط العبارات من هذا الملف.
