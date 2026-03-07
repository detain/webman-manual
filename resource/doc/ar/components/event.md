# معالجة الأحداث
يوفر `webman/event` آلية أحداث أنيقة تتيح تنفيذ منطق الأعمال دون تعديل الكود، وتحقيق الفصل بين الوحدات. سيناريو نموذجي: عند تسجيل مستخدم جديد بنجاح، يكفي نشر حدث مخصص مثل `user.register`، وستتمكن كل وحدة من استقبال الحدث وتنفيذ المنطق المناسب.

## التثبيت
`composer require webman/event`

## الاشتراك في الأحداث
يتم تكوين الاشتراك في الأحداث بشكل موحد عبر ملف `config/event.php`.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...دوال معالجة أحداث أخرى...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...دوال معالجة أحداث أخرى...
    ]
];
```

**ملاحظة:**
- `user.register` و `user.logout` وما شابهها أسماء أحداث (نوع سلسلة). يُفضّل استخدام كلمات صغيرة مفصولة بنقطة (`.`).
- يمكن أن يكون للحدث عدة دوال معالجة؛ تُستدعى بالترتيب المحدد في التكوين.

## دوال معالجة الأحداث
يمكن أن تكون دوال المعالجة أي طريقة فصل، أو دالة، أو إغلاق (closure).

مثال: إنشاء فصل `app/event/User.php` (أنشئ المجلد إن لم يكن موجوداً).

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## نشر الأحداث
استخدم `Event::dispatch($event_name, $data);` أو `Event::emit($event_name, $data);` لنشر حدث. مثال:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

هناك دالتان للنشر: `Event::dispatch($event_name, $data);` و `Event::emit($event_name, $data);` ولهما نفس المعاملات. الفرق: `emit` يلتقط الاستثناءات داخلياً؛ إذا ألقى أحد المعالجات استثناءً، تستمر الباقي في التنفيذ. أما `dispatch` فلا يلتقط الاستثناءات؛ إذا ألقى أي معالج استثناءً، يتوقف التنفيذ وينتشر الاستثناء إلى الأعلى.

> **تلميح**
> المعامل `$data` يمكن أن يكون أي بيانات (مصفوفة، مثيل فصل، سلسلة، إلخ).

## الاستماع للأحداث باستخدام أحرف البدل
التسجيل بأحرف البدل يتيح معالجة عدة أحداث بنفس المستمع. مثال في `config/event.php`:

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

يمكن الحصول على اسم الحدث المحدد عبر المعامل الثاني `$event_data` لدالة المعالجة:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // اسم الحدث المحدد، مثل user.register، user.logout، إلخ
        var_export($user);
    }
}
```

## إيقاف بث الحدث
عند إرجاع `false` من دالة المعالجة، يتم إيقاف بث ذلك الحدث.

## معالجة الأحداث بالإغلاقات
دالة المعالجة يمكن أن تكون طريقة فصل أو إغلاقاً. مثال:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## عرض الأحداث والمستمعين
استخدم الأمر `php webman event:list` لعرض جميع الأحداث والمستمعين المكوّنة في المشروع.

## نطاق الدعم
بجانب المشروع الرئيسي، تدعم [الإضافات الأساسية](../plugin/base.md) و[إضافات التطبيق](../app/app.md) تكوين `event.php`.
**ملف تكوين الإضافة الأساسية:** `config/plugin/البائع/اسم-الإضافة/event.php`
**ملف تكوين إضافة التطبيق:** `plugin/اسم-الإضافة/config/event.php`

## ملاحظات
معالجة الأحداث ليست غير متزامنة، ولا مناسبة للأعمال البطيئة؛ يجب استخدام قوائم انتظار الرسائل للأعمال البطيئة، مثل [webman/redis-queue](https://www.workerman.net/plugin/12).
