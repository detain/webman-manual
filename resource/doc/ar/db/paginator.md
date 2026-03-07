# التقسيم إلى صفحات

# التقسيم إلى صفحات باستخدام ORM Laravel

توفر حزمة `illuminate/database` من Laravel وظيفة تقسيم صفحات مريحة.

## التثبيت
`composer require illuminate/pagination`

## الاستخدام
```php
public function index(Request $request)
{
    $per_page = 10;
    $users = Db::table('user')->paginate($per_page);
    return view('index/index', ['users' => $users]);
}
```

## طرق مثيل المقسم
|  الطريقة   | الوصف  |
|  ----  |-----|
|$paginator->count()|الحصول على إجمالي البيانات في الصفحة الحالية|
|$paginator->currentPage()|الحصول على رقم الصفحة الحالية|
|$paginator->firstItem()|الحصول على رقم أول عنصر في مجموعة النتائج|
|$paginator->getOptions()|الحصول على خيارات المقسم|
|$paginator->getUrlRange($start, $end)|إنشاء URLs لنطاق صفحات محدد|
|$paginator->hasPages()|هل توجد بيانات كافية لإنشاء صفحات متعددة|
|$paginator->hasMorePages()|هل توجد المزيد من الصفحات للعرض|
|$paginator->items()|الحصول على عناصر البيانات في الصفحة الحالية|
|$paginator->lastItem()|الحصول على رقم آخر عنصر في مجموعة النتائج|
|$paginator->lastPage()|الحصول على رقم آخر صفحة (غير متاح في simplePaginate)|
|$paginator->nextPageUrl()|الحصول على رابط الصفحة التالية|
|$paginator->onFirstPage()|هل الصفحة الحالية هي الأولى|
|$paginator->perPage()|الحصول على العدد الإجمالي للعناصر في كل صفحة|
|$paginator->previousPageUrl()|الحصول على رابط الصفحة السابقة|
|$paginator->total()|الحصول على إجمالي البيانات في مجموعة النتائج (غير متاح في simplePaginate)|
|$paginator->url($page)|الحصول على رابط صفحة محددة|
|$paginator->getPageName()|الحصول على اسم معامل الاستعلام المستخدم لتخزين رقم الصفحة|
|$paginator->setPageName($name)|تعيين اسم معامل الاستعلام المستخدم لتخزين رقم الصفحة|

> **ملاحظة**
> الطريقة `$paginator->links()` غير مدعومة

## مكون التقسيم إلى صفحات
في webman لا يمكن استخدام الطريقة `$paginator->links()` لعرض أزرار التقسيم، ولكن يمكننا استخدام مكونات أخرى للعرض، مثل `jasongrimes/php-paginator`.

**التثبيت**
`composer require "jasongrimes/paginator:~1.0"`

**الخادم**
```php
<?php
namespace app\controller;

use JasonGrimes\Paginator;
use support\Request;
use support\Db;

class UserController
{
    public function get(Request $request)
    {
        $per_page = 10;
        $current_page = $request->input('page', 1);
        $users = Db::table('user')->paginate($per_page, '*', 'page', $current_page);
        $paginator = new Paginator($users->total(), $per_page, $current_page, '/user/get?page=(:num)');
        return view('user/get', ['users' => $users, 'paginator'  => $paginator]);
    }
}
```

**القالب (PHP الأصلي)**
إنشاء قالب جديد app/view/user/get.html
```html
<html>
<head>
  <!-- دعم مدمج لأنماط التقسيم Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**القالب (twig)**
إنشاء قالب جديد app/view/user/get.html
```html
<html>
<head>
  <!-- دعم مدمج لأنماط التقسيم Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**القالب (blade)**
إنشاء قالب جديد app/view/user/get.blade.php
```html
<html>
<head>
  <!-- دعم مدمج لأنماط التقسيم Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**القالب (thinkphp)**
إنشاء قالب جديد app/view/user/get.html
```html
<html>
<head>
    <!-- دعم مدمج لأنماط التقسيم Bootstrap -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

النتيجة كما يلي:
![](../../assets/img/paginator.png)

# التقسيم إلى صفحات باستخدام ORM ThinkPHP

لا حاجة لتثبيت مكتبات إضافية؛ يكفي تثبيت think-orm فقط.

## الاستخدام
```php
public function index(Request $request)
{
    $per_page = 10;
    $users = Db::table('user')->paginate(['list_rows' => $per_page, 'page' => $request->get('page', 1), 'path' => $request->path()]);
    return view('index/index', ['users' => $users]);
}
```

**القالب (thinkphp)**
```html
<html>
<head>
    <!-- دعم مدمج لأنماط التقسيم Bootstrap -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{$users|raw}

</body>
</html>
```
