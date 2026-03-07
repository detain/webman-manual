# الحقن التلقائي للتبعيات

في webman، الحقن التلقائي للتبعيات ميزة اختيارية ومعطلة افتراضيًا. إذا كنت بحاجة إليه، يُوصى باستخدام [php-di](https://php-di.org/doc/getting-started.html). فيما يلي كيفية استخدام `php-di` مع webman.

## التثبيت

```
composer require php-di/php-di:^7.0
```

عدّل إعداد `config/container.php`. المحتوى النهائي يجب أن يكون كالتالي:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> يجب أن يعيد ملف `config/container.php` في النهاية مثيل حاوية متوافق مع مواصفة PSR-11. إن لم ترغب في استخدام `php-di`، يمكنك إنشاء وإرجاع مثيل حاوية آخر متوافق مع PSR-11 هنا. الإعداد الافتراضي يوفر فقط الوظائف الأساسية لحاوية webman.

## الحقن عبر المُنشئ

أنشئ ملف `app/service/Mailer.php` (أنشئ المجلد إن لم يكن موجودًا) بالمحتوى التالي:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // كود إرسال البريد محذوف
    }
}
```

محتوى `app/controller/UserController.php`:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'مرحباً بك!');
        return response('ok');
    }
}
```

عادةً، لإنشاء مثيل `app\controller\UserController` يلزم الكود التالي:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

عند استخدام `php-di`، لا يحتاج المطورون إلى إنشاء مثيل `Mailer` يدويًا في المتحكم — إذ يقوم webman بذلك تلقائيًا. وإن كانت هناك تبعيات أخرى عند إنشاء مثيل `Mailer`، سيُنشئها webman ويُحقنها تلقائيًا. لا يلزم أي عمل إعدادي من المطور.

> **ملاحظة**
> الحقن التلقائي للتبعيات متاح فقط للمثيلات التي تُنشأ بواسطة الإطار أو `php-di`. المثيلات المُنشأة يدويًا بـ `new` لا تدعمه. للحقن، استخدم واجهة `support\Container` بدلاً من `new`، مثلاً:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// المثيلات المُنشأة بـ new لا تدعم الحقن
$user_service = new UserService;
// المثيلات المُنشأة بـ new لا تدعم الحقن
$log_service = new LogService($path, $name);

// المثيلات المُنشأة بـ Container تدعم الحقن
$user_service = Container::get(UserService::class);
// المثيلات المُنشأة بـ Container تدعم الحقن
$log_service = Container::make(LogService::class, [$path, $name]);
```

## الحقن عبر السمات (Attributes)

إلى جانب الحقن عبر المُنشئ، يمكنك استخدام الحقن عبر السمات. بمتابعة المثال السابق، عدّل `app\controller\UserController` كالتالي:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'مرحباً بك!');
        return response('ok');
    }
}
```

هذا المثال يستخدم السمة `#[Inject]` للحقن ويحقن المثيل تلقائيًا في العضو وفقًا لنوع الكائن. التأثير مثل الحقن عبر المُنشئ لكن الكود أوضح.

> **ملاحظة**
> webman لا يدعم حقن معاملات المتحكم قبل الإصدار 1.4.6. مثلاً الكود التالي غير مدعوم عندما webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // حقن معاملات المتحكم غير مدعوم قبل الإصدار 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'مرحباً بك!');
        return response('ok');
    }
}
```

## حقن مخصص عبر المُنشئ

أحيانًا معاملات المُنشئ قد تكون ليست مثيلات صنف بل نصوصًا أو أرقامًا أو مصفوفات أو بيانات أخرى. مثلاً مُنشئ Mailer قد يحتاج عنوان IP ومنفذ خادم SMTP:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // كود إرسال البريد محذوف
    }
}
```

هنا لا يمكن استخدام الحقن التلقائي عبر المُنشئ مباشرة لأن `php-di` لا يستطيع تحديد قيم `$smtp_host` و `$smtp_port`. في هذه الحالة يمكن استخدام حقن مخصص.

أضف الكود التالي في `config/dependence.php` (أنشئ الملف إن لم يكن موجودًا):

```php
return [
    // ... إعدادات أخرى محذوفة

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

عند حاجة الحقن لمثيل `app\service\Mailer`، سيُستخدم المثيل المُنشأ في هذا الإعداد تلقائيًا.

لاحظ أن `config/dependence.php` يستخدم `new` لإنشاء مثيل صنف `Mailer`. في هذا المثال لا مشكلة، لكن إذا كان `Mailer` يعتمد على أصناف أخرى أو يستخدم الحقن عبر السمات داخليًا، فالتهيئة بـ `new` لن تقوم بالحقن التلقائي. الحل هو استخدام حقن مخصص عبر واجهة وتهيئة الأصناف عبر `Container::get(اسم الصنف)` أو `Container::make(اسم الصنف, [معاملات المُنشئ])`.

## حقن مخصص عبر واجهة

في المشاريع الحقيقية يُفضّل البرمجة ضد الواجهات بدلاً من الأصناف الملموسة. مثلاً `app\controller\UserController` يجب أن يعتمد على `app\service\MailerInterface` بدلاً من `app\service\Mailer`.

عرّف واجهة `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

عرّف تنفيذ `MailerInterface`:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // كود إرسال البريد محذوف
    }
}
```

استخدم واجهة `MailerInterface` بدلاً من التنفيذ الملموس:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'مرحباً بك!');
        return response('ok');
    }
}
```

عرّف تنفيذ `MailerInterface` في `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

عند حاجة التطبيق لواجهة `MailerInterface`، سيُستخدم تنفيذ `Mailer` تلقائيًا.

> ميزة البرمجة ضد الواجهات أن استبدال مكوّن لا يتطلب تغيير كود الأعمال — فقط التنفيذ الملموس في `config/dependence.php`. مفيد جدًا أيضًا لاختبار الوحدة.

## حقن مخصص آخر

بالإضافة لتبعيات الأصناف، يمكن لـ `config/dependence.php` تعريف قيم أخرى كالنصوص والأرقام والمصفوفات.

مثلاً إن عُرّف `config/dependence.php` كالتالي:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

يمكن حقن `smtp_host` و `smtp_port` في خصائص الصنف عبر `#[Inject]`:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // كود إرسال البريد محذوف
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // سيُطبع 192.168.1.11:25
    }
}
```

# التحميل الكسول (Lazy Loading)

> التحميل الكسول نمط تصميم يؤخر إنشاء أو تهيئة الكائنات حتى الحاجة الفعلية إليها.

هذه الميزة تحتاج تبعية إضافية. الحزمة التالية فرع من `ocramius/proxy-manager`؛ المستودع الأصلي لا يدعم PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

الاستخدام:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass مُنشأ\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "اسم صنف الوكيل: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

المخرجات:

```
اسم صنف الوكيل: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass مُنشأ
name: Lazy Loaded Object
```

يوضّح هذا المثال أنه عند حقن صنف مُعلَن بالسمة `#[Injectable]`، يُنشأ أولاً صنف الوكيل. الصنف الفعلي يُنشأ فقط عند استدعاء أي من طرقه.

# التبعيات الدائرية

التبعيات الدائرية تحدث عندما تعتمد أصناف متعددة على بعضها، مكوّنة حلقة مغلقة من التبعيات.

- تبعية دائرية مباشرة
  - الوحدة A تعتمد على الوحدة B والوحدة B تعتمد على الوحدة A
  - تشكل الحلقة: A → B → A

- تبعية دائرية غير مباشرة
  - تشمل عدة وحدات في حلقة التبعيات
  - مثلاً: A → B → C → A

عند استخدام الحقن عبر السمات، يكتشف `php-di` التبعيات الدائرية ويرمي استثناء. عند الحاجة، استخدم الأسلوب التالي:

```php
class userController
{

    // احذف هذا الكود
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## المزيد

راجع [توثيق php-di](https://php-di.org/doc/getting-started.html).
