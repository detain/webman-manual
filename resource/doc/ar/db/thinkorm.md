# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) مكوّن قاعدة بيانات يعتمد على [top-think/think-orm](https://github.com/top-think/think-orm)، ويدعم مجمع الاتصالات والعمل في بيئات الـ coroutine والـ non-coroutine.

## تثبيت think-orm

`composer require -W webman/think-orm`

يجب إعادة التشغيل (restart) بعد التثبيت (reload لا يُؤثر).

## ملف الإعدادات

عدّل ملف الإعدادات `config/think-orm.php` وفقًا لاحتياجاتك الفعلية.

## الوثائق

https://www.kancloud.cn/manual/think-orm

## الاستخدام

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

## إنشاء النماذج

نماذج think-orm ترث من `support\think\Model`، كما هو موضّح أدناه:

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

يمكنك أيضًا إنشاء نماذج think-orm باستخدام الأمر التالي:

```
php webman make:model اسم_الجدول
```

> **تلميح**
> يتطلب هذا الأمر تثبيت `webman/console`. ثبّته بـ `composer require webman/console ^1.2.13`

> **تنبيه**
> إذا اكتشف الأمر make:model أن المشروع الرئيسي يستخدم `illuminate/database`، فسيُنشئ ملفات نماذج مبنية على Illuminate بدلًا من think-orm. في هذه الحالة، أضِف المعامل `tp` لإجبار إنشاء نموذج think-orm: `php webman make:model اسم_الجدول tp` (حدّث `webman/console` إن لم ينجح).
