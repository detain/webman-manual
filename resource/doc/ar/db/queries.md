# استخدام قاعدة البيانات (مكوّن Laravel)

## جلب كل الصفوف
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function all(Request $request)
    {
        $users = Db::table('users')->get();
        return view('user/all', ['users' => $users]);
    }
}
```

## جلب أعمدة محددة
```php
$users = Db::table('user')->select('name', 'email as user_email')->get();
```

## جلب صف واحد
```php
$user = Db::table('users')->where('name', 'John')->first();
```

## جلب عمود واحد
```php
$titles = Db::table('roles')->pluck('title');
```
تحديد قيمة حقل id كفهرس
```php
$roles = Db::table('roles')->pluck('title', 'id');

foreach ($roles as $id => $title) {
    echo $title;
}
```

## جلب قيمة واحدة (حقل)
```php
$email = Db::table('users')->where('name', 'John')->value('email');
```

## إزالة التكرار
```php
$email = Db::table('user')->select('nickname')->distinct()->get();
```

## تقسيم النتائج إلى دفعات
إذا كنت بحاجة لمعالجة آلاف أو ملايين سجلات قاعدة البيانات، فإن قراءة كل البيانات دفعة واحدة قد تستغرق وقتاً طويلاً وتؤدي إلى نفاد الذاكرة. في هذه الحالة يمكنك استخدام الدالة `chunkById`. تجلب هذه الدالة دفعة صغيرة من مجموعة النتائج في كل مرة وتمررها لدالة إغلاق للمعالجة. على سبيل المثال، يمكننا تقسيم بيانات جدول `users` بالكامل إلى دفعات صغيرة من 100 سجل في كل مرة:
```php
Db::table('users')->orderBy('id')->chunkById(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```
يمكنك إيقاف جلب المزيد من الدفعات بإرجاع `false` من داخل دالة الإغلاق:
```php
Db::table('users')->orderBy('id')->chunkById(100, function ($users) {
    // معالجة السجلات...

    return false;
});
```

> ملاحظة: لا تحذف البيانات داخل دالة الاستدعاء، فقد يؤدي ذلك لاستبعاد بعض السجلات من مجموعة النتائج

## التجميعات

يوفر مُنشئ الاستعلامات أيضاً طرق تجميع متنوعة مثل count و max و min و avg و sum:
```php
$users = Db::table('users')->count();
$price = Db::table('orders')->max('price');
$price = Db::table('orders')->where('finalized', 1)->avg('price');
```

## التحقق من وجود السجل
```php
return Db::table('orders')->where('finalized', 1)->exists();
return Db::table('orders')->where('finalized', 1)->doesntExist();
```

## التعبيرات الأصلية
النموذج
```php
selectRaw($expression, $bindings = [])
```
قد تحتاج أحياناً لاستخدام تعبيرات أصلية في الاستعلامات. يمكنك استخدام `selectRaw()` لإنشاء تعبير أصلي:
```php
$orders = Db::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```

وبالمثل، تتوفر طرق التعبيرات الأصلية `whereRaw()` و `orWhereRaw()` و `havingRaw()` و `orHavingRaw()` و `orderByRaw()` و `groupByRaw()`.

يُستخدم `Db::raw($value)` أيضاً لإنشاء تعبير أصلي، لكنه لا يدعم ربط المعاملات، فاحذر من ثغرات حقن SQL عند استخدامه:
```php
$orders = Db::table('orders')
                ->select('department', Db::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();
```

## جمل JOIN
```php
// join
$users = Db::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();

// leftJoin            
$users = Db::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

// rightJoin
$users = Db::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

// crossJoin    
$users = Db::table('sizes')
            ->crossJoin('colors')
            ->get();
```

## جمل UNION
```php
$first = Db::table('users')
            ->whereNull('first_name');

$users = Db::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

## جمل Where
النموذج
```php
where($column, $operator = null, $value = null)
```
المعامل الأول اسم العمود، والثاني أي معامل مقبول في نظام قاعدة البيانات، والثالث القيمة للمقارنة:
```php
$users = Db::table('users')->where('votes', '=', 100)->get();

// عندما يكون المعامل علامة المساواة يمكن حذفه، لذا هذه الجملة تعادل السابقة
$users = Db::table('users')->where('votes', 100)->get();

$users = Db::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = Db::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = Db::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```

يمكنك أيضاً تمرير مصفوفة شروط لدالة where:
```php
$users = Db::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

تستقبل الدالة orWhere نفس معاملات الدالة where:
```php
$users = Db::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

يمكنك تمرير دالة إغلاق كمعامل أول للدالة orWhere:
```php
// SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
$users = Db::table('users')
            ->where('votes', '>', 100)
            ->orWhere(function($query) {
                $query->where('name', 'Abigail')
                      ->where('votes', '>', 50);
            })
            ->get();
```

whereBetween / orWhereBetween — التحقق من أن قيمة الحقل بين قيمتين:
```php
$users = Db::table('users')
           ->whereBetween('votes', [1, 100])
           ->get();
```

whereNotBetween / orWhereNotBetween — التحقق من أن قيمة الحقل خارج قيمتين:
```php
$users = Db::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
```

whereIn / whereNotIn / orWhereIn / orWhereNotIn — التحقق من أن قيمة الحقل موجودة في مصفوفة محددة:
```php
$users = Db::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
```

whereNull / whereNotNull / orWhereNull / orWhereNotNull — التحقق من أن الحقل المحدد يساوي NULL:
```php
$users = Db::table('users')
                    ->whereNull('updated_at')
                    ->get();
```

whereNotNull — التحقق من أن الحقل المحدد لا يساوي NULL:
```php
$users = Db::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
```

whereDate / whereMonth / whereDay / whereYear / whereTime — مقارنة قيمة الحقل بتاريخ معين:
```php
$users = Db::table('users')
                ->whereDate('created_at', '2016-12-31')
                ->get();
```

whereColumn / orWhereColumn — مقارنة قيم حقلين:
```php
$users = Db::table('users')
                ->whereColumn('first_name', 'last_name')
                ->get();
                
// يمكنك أيضاً تمرير معامل مقارنة
$users = Db::table('users')
                ->whereColumn('updated_at', '>', 'created_at')
                ->get();
                
// يمكن أيضاً تمرير مصفوفة للدالة whereColumn
$users = Db::table('users')
                ->whereColumn([
                    ['first_name', '=', 'last_name'],
                    ['updated_at', '>', 'created_at'],
                ])->get();
```

تجميع المعاملات
```php
// select * from users where name = 'John' and (votes > 100 or title = 'Admin')
$users = Db::table('users')
           ->where('name', '=', 'John')
           ->where(function ($query) {
               $query->where('votes', '>', 100)
                     ->orWhere('title', '=', 'Admin');
           })
           ->get();
```

whereExists
```php
// select * from users where exists ( select 1 from orders where orders.user_id = users.id )
$users = Db::table('users')
           ->whereExists(function ($query) {
               $query->select(Db::raw(1))
                     ->from('orders')
                     ->whereRaw('orders.user_id = users.id');
           })
           ->get();
```

## orderBy
```php
$users = Db::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

## الترتيب العشوائي
```php
$randomUser = Db::table('users')
                ->inRandomOrder()
                ->first();
```
> الترتيب العشوائي يؤثر بشكل كبير على أداء الخادم، ولا يُنصح باستخدامه

## groupBy / having
```php
$users = Db::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
// يمكنك تمرير عدة معاملات للدالة groupBy
$users = Db::table('users')
                ->groupBy('first_name', 'status')
                ->having('account_id', '>', 100)
                ->get();
```

## offset / limit
```php
$users = Db::table('users')
                ->offset(10)
                ->limit(5)
                ->get();
```

## الإدراج
إدراج سجل واحد
```php
Db::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
```
إدراج عدة سجلات
```php
Db::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

## معرفات التزايد التلقائي
```php
$id = Db::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> ملاحظة: عند استخدام PostgreSQL، ستعتبر الدالة insertGetId أن `id` هو اسم حقل التزايد التلقائي افتراضياً. لاستخدام تسلسل آخر، يمكنك تمرير اسم الحقل كمعامل ثانٍ للدالة insertGetId.

## التحديث
```php
$affected = Db::table('users')
              ->where('id', 1)
              ->update(['votes' => 1]);
```

## التحديث أو الإدراج
قد ترغب أحياناً بتحديث سجل موجود في قاعدة البيانات، أو إنشائه إذا لم يُعثر على سجل مطابق:
```php
Db::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```
تحاول الدالة updateOrInsert أولاً العثور على سجل مطابق باستخدام المفاتيح والقيم من المعامل الأول. عند وجود السجل، تُحدَّث قيمه وفق المعامل الثاني. إن لم يُعثر على سجل، يُدرَج سجل جديد يجمع بيانات المصفوفتين.

## الزيادة والنقصان
كلتا الدالتين تستقبلا معاملاً واحداً على الأقل: العمود المطلوب تعديله. المعامل الثاني اختياري ويحدد مقدار الزيادة أو النقصان:
```php
Db::table('users')->increment('votes');

Db::table('users')->increment('votes', 5);

Db::table('users')->decrement('votes');

Db::table('users')->decrement('votes', 5);
```
يمكنك أيضاً تحديد الحقول المحدثة أثناء العملية:
```php
Db::table('users')->increment('votes', 1, ['name' => 'John']);
```

## الحذف
```php
Db::table('users')->delete();

Db::table('users')->where('votes', '>', 100)->delete();
```
لتفريغ الجدول، استخدم الدالة truncate التي تحذف كل الصفوف وتعيد ضبط معرف التزايد التلقائي إلى الصفر:
```php
Db::table('users')->truncate();
```

## المعاملات
راجع [معاملات قاعدة البيانات](../others/transaction.md)

## القفل التشاؤمي
يتضمن مُنشئ الاستعلامات أيضاً دوالاً تُساعد في تنفيذ "القفل التشاؤمي" في جمل select. لتنفيذ "قفل مشترك"، استخدم الدالة sharedLock. يمنع القفل المشترك تعديل الأعمدة المحددة حتى يتم تأكيد المعاملة:
```php
Db::table('users')->where('votes', '>', 100)->sharedLock()->get();
```
أو استخدم الدالة lockForUpdate. يمنع قفل "التحديث" تعديل أو تحديد الصفوف بواسطة أقفال مشتركة أخرى:
```php
Db::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

## التصحيح
يمكنك استخدام الدالتين `dd` أو `dump` لعرض نتائج الاستعلام أو جمل SQL. تعرض الدالة `dd` معلومات التصحيح ثم توقف تنفيذ الطلب. تعرض الدالة `dump` معلومات التصيف أيضاً لكنها لا توقف التنفيذ:
```php
Db::table('users')->where('votes', '>', 100)->dd();
Db::table('users')->where('votes', '>', 100)->dump();
```

> **ملاحظة**
> يتطلب التصحيح تثبيت `symfony/var-dumper` بالأمر `composer require symfony/var-dumper`
