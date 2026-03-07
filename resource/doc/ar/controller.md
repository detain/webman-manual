# المتحكم

أنشئ ملف المتحكم الجديد `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

عند الوصول إلى `http://127.0.0.1:8787/foo`، تعيد الصفحة `hello index`.

عند الوصول إلى `http://127.0.0.1:8787/foo/hello`، تعيد الصفحة `hello webman`.

يمكنك تغيير قواعد التوجيه من خلال إعداد المسارات، راجع [المسارات](route.md).

> **تلميح**
> إذا ظهر خطأ 404، افتح `config/app.php`، عيّن `controller_suffix` إلى `Controller` وأعد التشغيل.

## لاحقة المتحكم
بدءًا من الإصدار 1.3، يدعم webman تعيين لاحقة المتحكم في `config/app.php`. إذا كان `controller_suffix` مُعيَّنًا على سلسلة فارغة `''`، فالمتحكم يبدو كالتالي:

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

يُوصى بشدة بتعيين لاحقة المتحكم على `Controller` لتجنب التضارب مع أسماء فئات النماذج وزيادة الأمان.

## الشرح
- يمرر الإطار تلقائيًا كائن `support\Request` إلى المتحكم، والذي يمكن من خلاله الحصول على بيانات إدخال المستخدم (get، post، header، cookie، إلخ)، راجع [الطلب](request.md).
- يمكن للمتحكم إرجاع أرقام أو سلاسل أو كائنات `support\Response`، لكن لا يمكنه إرجاع أنواع بيانات أخرى.
- يمكن إنشاء كائنات `support\Response` باستخدام الدوال المساعدة مثل `response()`، `json()`، `xml()`، `jsonp()`، `redirect()`، إلخ.

## ربط معلمات المتحكم

#### مثال
يدعم webman الربط التلقائي لمعلمات الطلب بمعلمات أساليب المتحكم. على سبيل المثال:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

يمكنك تمرير قيم `name` و `age` عبر `GET` أو `POST`، أو عبر معلمات المسار. على سبيل المثال:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

الأولوية هي: `معلمات المسار` > `GET` > `POST`.

#### القيم الافتراضية

عند الوصول إلى `/user/create?name=tom`، ستحصل على الخطأ التالي:

```html
Missing input parameter age
```

السبب أننا لم نمرر المعلمة `age`. يمكن الحل بتعيين قيمة افتراضية. على سبيل المثال:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### أنواع المعلمات
عند الوصول إلى `/user/create?name=tom&age=not_int`، ستحصل على الخطأ التالي:

> **تلميح**
> للراحة في الاختبار نمرر المعلمات مباشرة في شريط العنوان. في التطوير الفعلي يجب تمرير المعلمات عبر `POST`.

```html
Input age must be of type int, string given
```

ذلك لأن البيانات المستلمة تُحوَّل وفقًا للنوع. عند فشل التحويل يتم رمي الاستثناء `support\exception\InputTypeException`. بما أن `age` لا يمكن تحويله إلى `int`، يظهر هذا الخطأ.

#### رسائل الأخطاء المخصصة
يمكن تخصيص رسائل مثل `Missing input parameter age` و `Input age must be of type int, string given` عبر الترجمة. راجع الأوامر التالية:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'يجب أن يكون المعلمة :parameter من نوع :exceptType، النوع المرسل هو :actualType',
    'Missing input parameter :parameter' => 'معلمة الإدخال :parameter مفقودة',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### أنواع أخرى
يدعم webman أنواعًا مثل `int`، `float`، `string`، `bool`، `array`، `object` و`مثيلات الفئات`. على سبيل المثال:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

عند الوصول إلى `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`، ستحصل على:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### مثيل الفئة
يدعم webman تمرير مثيلات الفئات عبر تلميحات النوع. على سبيل المثال:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

عند الوصول إلى `/blog/create?blog[title]=hello&blog[content]=world`، ستحصل على:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### مثيل النموذج

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // أضف هنا الحقول القابلة للملء لمنع الحقول غير الآمنة من الواجهة الأمامية
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

عند الوصول إلى `/user/create?user[name]=tom&user[age]=18`، ستحصل على نتيجة شبيهة بـ:

```json
1
```

## دورة حياة المتحكم

عندما يكون `controller_reuse` في `config/app.php` بقيمة `false`، يتم تهيئة مثيل المتحكم مرة واحدة لكل طلب، ويُدمَّر بعد انتهاء الطلب. نفس آلية الأطر التقليدية.

عندما يكون `true`، تعيد جميع الطلبات استخدام نفس مثيل المتحكم، أي أن المثيل يبقى في الذاكرة بعد الإنشاء.

> **تنبيه**
> عند إعادة الاستخدام، لا ينبغي للطلبات تغيير أي خصائص للمتحكم لأن ذلك يؤثر على الطلبات التالية. مثال:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // سيبقى هذا النموذج بعد أول طلب update?id=1
        // إذا طُلِب delete?id=2، ستُحذف بيانات id=1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **تلميح**
> إرجاع بيانات في المُنشئ `__construct()` للمتحكم لا ينتج أي تأثير. مثال:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // return في المُنشئ لا ينتج أي تأثير، المتصفح لن يستقبل هذه الاستجابة
        return response('hello'); 
    }
}
```

## الفرق بين عدم إعادة الاستخدام وإعادة الاستخدام

#### عدم إعادة الاستخدام
يُنشأ مثيل جديد للمتحكم لكل طلب، ويُحرَّر بعد انتهاء الطلب. مطابق للأطر التقليدية وعادات معظم المطورين. أداء أقل قليلًا (حوالي 10% في اختبار helloworld، عادةً غير مهم مع الأعمال الفعلية).

#### إعادة الاستخدام
يُنشأ المتحكم مرة واحدة لكل عملية ولا يُحرَّر بعد الطلب. الطلبات التالية تعيد استخدام المثيل ذاته. أداء أفضل، لكنه لا يتوافق مع عادات الكثير من المطورين.

#### حالات لا يمكن فيها إعادة الاستخدام

عندما يغيّر الطلب خصائص المتحكم — هذه التغييرات تؤثر على الطلبات التالية.

عند تنفيذ تهيئة في `__construct()` لكل طلب — المُنشئ يُستدعى مرة واحدة لكل عملية، وليس لكل طلب.
