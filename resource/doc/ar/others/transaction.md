# الاستخدام الصحيح للعمليات (Transactions)

استخدام عمليات قاعدة البيانات في webman مطابق للإطارات الأخرى. فيما يلي النقاط التي يجب الانتباه إليها.

## هيكل الكود

هيكل الكود مطابق للإطارات الأخرى (مثل Laravel، think-orm مشابه):

```php
Db::beginTransaction();
try {
    // ..معالجة الأعمال محذوفة...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**مهم:** يجب استخدام `\Throwable` ولا تستخدم `\Exception`، لأن عملية الأعمال قد تُطلق `Error` وهو لا يرث من `Exception`.

## اتصال قاعدة البيانات

عند التعامل مع النماذج داخل عملية، انتبه جيداً لما إذا كان النموذج يحدد اتصالاً. إذا حدد النموذج اتصالاً، يجب تحديده عند بدء العملية؛ وإلا لن تكون العملية فعالة (think-orm مشابه). مثال:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // تم تحديد الاتصال للنموذج هنا
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

عندما يحدد النموذج اتصالاً، يجب تحديد الاتصال عند begin وcommit وrollback:

```php
Db::connection('mysql')->beginTransaction();
try {
    // معالجة الأعمال
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## البحث عن الطلبات ذات العمليات غير المكتملة
أحياناً خطأ في كود الأعمال يؤدي إلى عدم إكمال عملية طلب ما. لتحديد طريقة المتحكم التي لم تُكمل العملية بسرعة، يمكنك تثبيت مكوّن `webman/log`. يفحص هذا المكوّن تلقائياً بعد انتهاء كل طلب ما إذا كانت هناك عمليات غير مكتملة ويسجلها في السجل. الكلمة المفتاحية في السجل هي `Uncommitted transactions`

**طريقة تثبيت webman/log**

`composer require webman/log`

> **ملاحظة**
> بعد التثبيت تحتاج إلى إعادة التشغيل (restart)، وreload لن ينفع
