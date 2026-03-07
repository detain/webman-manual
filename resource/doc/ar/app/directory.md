# هيكل الدليل

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

تمتلك إضافة التطبيق نفس هيكل الدليل وملفات التكوين الخاصة بـ webman. عملياً، تجربة التطوير مماثلة تقريباً لتطوير تطبيق webman عادي.

دلائل الإضافات وتسميتها تتبع مواصفة PSR-4. نظراً لوجود الإضافات داخل دليل `plugin`، فإن جميع مساحات الأسماء تبدأ بـ `plugin`، مثلاً `plugin\foo\app\controller\UserController`.

## حول دليل api

كل إضافة تحتوي على دليل `api`. إذا كان تطبيقك يقدّم واجهات داخلية لاستدعائها من تطبيقات أخرى، ضع تلك الواجهات في دليل `api`.

ملاحظة: الواجهات المشار إليها هنا هي واجهات استدعاء الدوال، وليست واجهات شبكة/HTTP.

مثال: إضافة البريد الإلكتروني تقدّم واجهة `Email::send()` في `plugin/email/api/Email.php` ليستدعيها التطبيقات الأخرى عند إرسال البريد. كما يُولَّد `plugin/email/api/Install.php` تلقائياً لسوق إضافات webman-admin لتنفيذ عمليات التثبيت أو إلغاء التثبيت.
