# عملية إنشاء ونشر الإضافات الأساسية

## المبدأ
1. كمثال على إضافة cross-origin، تتكون الإضافة من ثلاثة أجزاء: ملف middleware لـ cross-origin، وملف الإعدادات `middleware.php`، وملف `Install.php` الذي يُنشأ تلقائياً بالأمر.
2. نستخدم أمراً لتغليف هذه الملفات الثلاثة ونشرها عبر Composer.
3. عند تثبيت المستخدم لإضافة cross-origin عبر Composer، ينسخ `Install.php` ملف middleware والإعدادات إلى `{المشروع الرئيسي}/config/plugin` لتحميلها webman، لتفعيل إعداد cross-origin تلقائياً.
4. عند إزالة المستخدم للإضافة عبر Composer، يزيل `Install.php` ملفات middleware والإعدادات المقابلة، لتحقيق إلغاء التثبيت التلقائي.

## المواصفات
1. اسم الإضافة من جزءين: `vendor` و`اسم الإضافة`، مثل `webman/push`، بما يوافق اسم حزمة Composer.
2. ملفات إعداد الإضافة توضع في `config/plugin/vendor/اسم الإضافة/` (أمر الـ console ينشئ مجلد الإعداد تلقائياً). إذا لم تحتج الإضافة إلى إعدادات، يُحذف المجلد المنشأ تلقائياً.
3. مجلد إعداد الإضافة يدعم فقط: `app.php` (الإعداد الرئيسي)، `bootstrap.php` (تشغيل العمليات)، `route.php` (المسارات)، `middleware.php` (الوسيط)، `process.php` (عمليات مخصصة)، `database.php` (قاعدة البيانات)، `redis.php` (Redis)، `thinkorm.php` (thinkorm). هذه تُعرَف تلقائياً من webman.
4. الوصول للإعدادات: `config('plugin.vendor.اسم الإضافة.ملف الإعداد.المفتاح');`، مثل `config('plugin.webman.push.app.app_key')`.
5. إذا كان للإضافة إعدادات قاعدة بيانات خاصة: لـ `illuminate/database` استخدم `Db::connection('plugin.vendor.اسم الإضافة.الاتصال')`، ولـ `thinkorm` استخدم `Db::connect('plugin.vendor.اسم الإضافة.الاتصال')`.
6. إذا وُضعت ملفات أعمال في `app/`، يجب تجنب التعارض مع المشروع الرئيسي أو الإضافات الأخرى.
7. ينبغي للإضافات تجنب نسخ ملفات أو مجلدات إلى المشروع الرئيسي. مثلاً في إضافة cross-origin يُنسخ الإعداد فقط؛ تبقى ملفات middleware في `vendor/webman/cros/src`.
8. يُفضَّل استخدام PascalCase لأسماء مساحات الإضافة، مثل `Webman/Console`.

## مثال

**تثبيت سطر الأوامر `webman/console`**

`composer require webman/console`

### إنشاء إضافة

افترض أن الإضافة المراد إنشاؤها تُسمى `foo/admin` (اسم مشروع Composer للنشر، بالأحرف الصغيرة). نفّذ:

`php webman plugin:create --name=foo/admin`

ينشأ `vendor/foo/admin` لملفات الإضافة و`config/plugin/foo/admin` للإعدادات.

> ملاحظة
> `config/plugin/foo/admin` يدعم: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. نفس صيغة webman، دمج تلقائي.
> الوصول باستخدام البادئة `plugin`، مثل `config('plugin.foo.admin.app')`.


### تصدير الإضافة

بعد انتهاء التطوير، نفّذ:

`php webman plugin:export --name=foo/admin`

تصدير

> توضيح
> التصدير ينسخ `config/plugin/foo/admin` إلى `vendor/foo/admin/src` وينشئ `Install.php` الذي يُنفَّذ عند التثبيت والإزالة.
> التثبيت الافتراضي: نسخ الإعداد من `vendor/foo/admin/src` إلى `config/plugin` للمشروع.
> الإزالة الافتراضية: حذف ملفات إعداد الإضافة من `config/plugin` للمشروع.
> يمكن تعديل `Install.php` لإضافة منطق مخصص للتثبيت والإزالة.

### إرسال الإضافة
* افترض أن لديك حساباً على [GitHub](https://github.com) و[Packagist](https://packagist.org).
* أنشئ مستودعاً `admin` على [GitHub](https://github.com) وارفع الكود، مثل `https://github.com/اسم-المستخدم/admin`.
* ادخل إلى `https://github.com/اسم-المستخدم/admin/releases/new` وأنشئ إصداراً، مثل `v1.0.0`.
* على [Packagist](https://packagist.org) انقر `Submit` وأرسل `https://github.com/اسم-المستخدم/admin` لنشر الإضافة.

> **نصيحة**
> في حال تعارض الاسم على Packagist، اختر vendor آخر، مثل تغيير `foo/admin` إلى `myfoo/admin`.

عند التحديثات: ارفع الكود إلى GitHub، أنشئ إصداراً جديداً على `https://github.com/اسم-المستخدم/admin/releases/new`، ثم انقر `Update` على `https://packagist.org/packages/foo/admin`.

## إضافة أوامر للإضافة
بعض الإضافات تحتاج أوامر مخصصة. مثلاً عند تثبيت `webman/redis-queue` يكتسب المشروع أمر `redis-queue:consumer`. تنفيذ `php webman redis-queue:consumer send-mail` يولد بسرعة صنف consumer باسم `SendMail.php`، مما يسرّع التطوير.

لإضافة أمر `foo-admin:add` لإضافة `foo/admin`:

### إنشاء أمر

**إنشاء الملف `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'وصف الأمر';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **ملاحظة**
> لتجنب تعارض الأوامر بين الإضافات، استخدم الصيغة `vendor-plugin:أمر`. مثلاً جميع أوامر `foo/admin` يجب أن تبدأ بـ `foo-admin:`، مثل `foo-admin:add`.

### إضافة الإعداد
**إنشاء `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // إضافة المزيد عند الحاجة...
];
```

> **نصيحة**
> `command.php` يسجل الأوامر المخصصة للإضافة. كل عنصر صنف أمر. `webman/console` يحمّلها تلقائياً. راجع [أوامر الـ console](console.md).

### تنفيذ التصدير
نفّذ `php webman plugin:export --name=foo/admin` لتصدير الإضافة ونشرها على Packagist. بعد تثبيت `foo/admin` يصبح أمر `foo-admin:add` متاحاً. تنفيذ `php webman foo-admin:add jerry` يطبع `Admin add jerry`.
