# নিয়ন্ত্রক

নতুন নিয়ন্ত্রক ফাইল তৈরি করুন `app/controller/FooController.php`।

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

`http://127.0.0.1:8787/foo` এ অ্যাক্সেস করলে পৃষ্ঠা `hello index` ফেরত দেবে।

`http://127.0.0.1:8787/foo/hello` এ অ্যাক্সেস করলে পৃষ্ঠা `hello webman` ফেরত দেবে।

রাউট কনফিগারের মাধ্যমে রাউট নিয়ম পরিবর্তন করতে পারেন, [রাউট](route.md) দেখুন।

> **পরামর্শ**
> 404 ত্রুটি দেখা দিলে `config/app.php` খুলুন, `controller_suffix` কে `Controller` হিসেবে সেট করুন এবং পুনরায় চালু করুন।

## নিয়ন্ত্রক প্রত্যয়
webman 1.3 থেকে `config/app.php` এ নিয়ন্ত্রক প্রত্যয় সেট করার সমর্থন করে। `controller_suffix` খালি `''` সেট থাকলে নিয়ন্ত্রক এরকম হবে:

`app\controller\Foo.php`।

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

নিয়ন্ত্রক প্রত্যয় `Controller` হিসেবে সেট করতে দৃঢ়ভাবে সুপারিশ করা হয়, যাতে নিয়ন্ত্রক এবং মডেল ক্লাসের নামের সংঘর্ষ এড়ানো যায় এবং নিরাপত্তা বাড়ে।

## বিবরণ
- ফ্রেমওয়ার্ক স্বয়ংক্রিয়ভাবে নিয়ন্ত্রকে `support\Request` অবজেক্ট প্রেরণ করে, এর মাধ্যমে ব্যবহারকারীর ইনপুট ডেটা (get, post, header, cookie ইত্যাদি) পাওয়া যায়, [অনুরোধ](request.md) দেখুন।
- নিয়ন্ত্রক সংখ্যা, স্ট্রিং বা `support\Response` অবজেক্ট ফেরত দিতে পারে, অন্য ধরনের ডেটা ফেরত দিতে পারে না।
- `support\Response` অবজেক্ট `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` ইত্যাদি হেল্পার ফাংশন দিয়ে তৈরি করা যায়।

## নিয়ন্ত্রক প্যারামিটার বাইন্ডিং

#### উদাহরণ
webman নিয়ন্ত্রক মেথড প্যারামিটারে অনুরোধ প্যারামিটারের স্বয়ংক্রিয় বাইন্ডিং সমর্থন করে। উদাহরণ:

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

`name` এবং `age` এর মান `GET` বা `POST` দিয়ে অথবা রাউট প্যারামিটার দিয়ে পাঠাতে পারেন। উদাহরণ:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

অগ্রাধিকার: `রাউট প্যারামিটার` > `GET` > `POST` প্যারামিটার।

#### ডিফল্ট মান

`/user/create?name=tom` এ অ্যাক্সেস করলে এই ত্রুটি দেখা দেবে:

```html
Missing input parameter age
```

কারণ আমরা `age` প্যারামিটার পাঠাইনি। প্যারামিটারে ডিফল্ট মান দিয়ে সমাধান করা যায়। উদাহরণ:

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

#### প্যারামিটার টাইপ
`/user/create?name=tom&age=not_int` এ অ্যাক্সেস করলে এই ত্রুটি দেখা দেবে:

> **পরামর্শ**
> পরীক্ষার সুবিধার জন্য আমরা সরাসরি ব্রাউজার ঠিকানা বারে প্যারামিটার দিচ্ছি। প্রকৃত উন্নয়নে `POST` দিয়ে প্যারামিটার পাঠাতে হবে।

```html
Input age must be of type int, string given
```

প্রাপ্ত ডেটা টাইপ অনুযায়ী রূপান্তরিত হয়। রূপান্তর ব্যর্থ হলে `support\exception\InputTypeException` নিক্ষেপ করা হয়। `age` প্যারামিটার `int` এ রূপান্তর করা যায় না তাই এই ত্রুটি হয়।

#### কাস্টম ত্রুটি বার্তা
`Missing input parameter age` এবং `Input age must be of type int, string given` এর মতো বার্তা বহুভাষার মাধ্যমে কাস্টমাইজ করা যায়। নিচের কমান্ডগুলো দেখুন:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'ইনপুট প্যারামিটার :parameter অবশ্যই :exceptType টাইপের হতে হবে, প্রেরিত টাইপ :actualType',
    'Missing input parameter :parameter' => 'ইনপুট প্যারামিটার :parameter অনুপস্থিত',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### অন্যান্য টাইপ
webman `int`, `float`, `string`, `bool`, `array`, `object` এবং `ক্লাস ইনস্ট্যান্স` এর মতো প্যারামিটার টাইপ সমর্থন করে। উদাহরণ:

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

`/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` এ অ্যাক্সেস করলে:

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

#### ক্লাস ইনস্ট্যান্স
webman টাইপ হিন্টের মাধ্যমে ক্লাস ইনস্ট্যান্স পাস করার সমর্থন করে। উদাহরণ:

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

`/blog/create?blog[title]=hello&blog[content]=world` এ অ্যাক্সেস করলে:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### মডেল ইনস্ট্যান্স

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
    // ফ্রন্টএন্ড থেকে অনিরাপদ ফিল্ড আসা রোধ করতে এখানে ফিলেবল ফিল্ড যোগ করতে হবে
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

`/user/create?user[name]=tom&user[age]=18` এ অ্যাক্সেস করলে এরকম ফলাফল পাওয়া যাবে:

```json
1
```

## নিয়ন্ত্রক জীবনচক্র

`config/app.php` এ `controller_reuse` `false` হলে প্রতিটি অনুরোধে সংশ্লিষ্ট নিয়ন্ত্রক ইনস্ট্যান্স একবার আরম্ভ হয়, অনুরোধ শেষে ইনস্ট্যান্স ধ্বংস হয়। এটি ঐতিহ্যবাহী ফ্রেমওয়ার্কের মত।

`controller_reuse` `true` হলে সব অনুরোধ নিয়ন্ত্রক ইনস্ট্যান্স পুনরায় ব্যবহার করে, অর্থাৎ একবার তৈরি হলে ইনস্ট্যান্স মেমরিতে থাকে এবং সব অনুরোধে ব্যবহৃত হয়।

> **নোট**
> পুনরায় ব্যবহার চালু থাকলে অনুরোধে নিয়ন্ত্রকের কোনো বৈশিষ্ট্য পরিবর্তন করা উচিত নয়, কারণ এই পরিবর্তন পরবর্তী অনুরোধগুলোকে প্রভাবিত করবে। উদাহরণ:

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
        // প্রথম update?id=1 অনুরোধের পর model সংরক্ষিত থাকবে
        // delete?id=2 অনুরোধ এলে id=1 এর ডেটা মুছে যাবে
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **পরামর্শ**
> নিয়ন্ত্রকের `__construct()` কনস্ট্রাক্টরে ডেটা রিটার্ন করলে কোনো প্রভাব থাকবে না। উদাহরণ:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // কনস্ট্রাক্টরে return করার কোনো প্রভাব নেই, ব্রাউজার এই রেসপন্স পাবে না
        return response('hello'); 
    }
}
```

## পুনরায় ব্যবহার না করার ও করার পার্থক্য

#### পুনরায় ব্যবহার না করা
প্রতিটি অনুরোধে নতুন নিয়ন্ত্রক ইনস্ট্যান্স তৈরি হয়, অনুরোধ শেষে ইনস্ট্যান্স মুক্ত হয় এবং মেমরি রিসাইকেল হয়। ঐতিহ্যবাহী ফ্রেমওয়ার্কের মত, বেশিরভাগ ডেভেলপারের অভ্যাসের সাথে মিলে। নিয়ন্ত্রক বারবার তৈরি ও ধ্বংস হওয়ায় পারফরম্যান্স পুনরায় ব্যবহারের চেয়ে সামান্য খারাপ (helloworld বেঞ্চমার্কে প্রায় ১০% খারাপ, আসল ব্যবসায়িক লজিক সহ প্রায় উপেক্ষণীয়)।

#### পুনরায় ব্যবহার
পুনরায় ব্যবহার করলে একটি প্রসেসে শুধু একবার নিয়ন্ত্রক তৈরি হয়, অনুরোধ শেষে ইনস্ট্যান্স মুক্ত করা হয় না। বর্তমান প্রসেসের পরবর্তী অনুরোধগুলো এই ইনস্ট্যান্স পুনরায় ব্যবহার করে। পারফরম্যান্স ভালো কিন্তু বেশিরভাগ ডেভেলপারের অভ্যাসের সাথে কম মিলে।

#### নিচের ক্ষেত্রে পুনরায় ব্যবহার করা যায় না

অনুরোধ নিয়ন্ত্রকের বৈশিষ্ট্য পরিবর্তন করলে পুনরায় ব্যবহার চালু করা যায় না, কারণ এই পরিবর্তন পরবর্তী অনুরোধগুলোকে প্রভাবিত করবে।

কিছু ডেভেলপার নিয়ন্ত্রকের `__construct()` কনস্ট্রাক্টরে প্রতিটি অনুরোধের জন্য কিছু ইনিশিয়ালাইজ করতে পছন্দ করে। সেক্ষেত্রে পুনরায় ব্যবহার করা যায় না, কারণ বর্তমান প্রসেসে কনস্ট্রাক্টর শুধু একবার কল হয়, প্রতিটি অনুরোধে নয়।
