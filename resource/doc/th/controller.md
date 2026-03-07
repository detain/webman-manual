# ตัวควบคุม

สร้างไฟล์ตัวควบคุมใหม่ `app/controller/FooController.php`

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

เมื่อเข้าถึง `http://127.0.0.1:8787/foo` หน้าเว็บจะคืนค่า `hello index`

เมื่อเข้าถึง `http://127.0.0.1:8787/foo/hello` หน้าเว็บจะคืนค่า `hello webman`

คุณสามารถเปลี่ยนกฎเส้นทางผ่านการกำหนดค่าตาม [เส้นทาง](route.md)

> **เคล็ดลับ**
> หากเกิดข้อผิดพลาด 404 กรุณาเปิด `config/app.php` ตั้งค่า `controller_suffix` เป็น `Controller` และรีสตาร์ท

## คำต่อท้ายตัวควบคุม
ตั้งแต่ webman เวอร์ชัน 1.3 รองรับการตั้งค่าคำต่อท้ายตัวควบคุมใน `config/app.php` หาก `controller_suffix` ตั้งเป็นค่าว่าง `''` ตัวควบคุมจะมีรูปแบบดังนี้

`app\controller\Foo.php`

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

แนะนำอย่างยิ่งให้ตั้งค่าคำต่อท้ายตัวควบคุมเป็น `Controller` เพื่อหลีกเลี่ยงการชนกันของชื่อระหว่างตัวควบคุมกับโมเดล และเพิ่มความปลอดภัย

## คำอธิบาย
- เฟรมเวิร์กจะส่งอ็อบเจ็กต์ `support\Request` ให้ตัวควบคุมโดยอัตโนมัติ สามารถดึงข้อมูลที่ผู้ใช้ป้อน (get, post, header, cookie ฯลฯ) ได้ ดูได้ที่ [คำขอ](request.md)
- ตัวควบคุมสามารถคืนค่าเป็นตัวเลข สตริง หรืออ็อบเจ็กต์ `support\Response` ได้ แต่ไม่สามารถคืนค่าประเภทข้อมูลอื่น
- อ็อบเจ็กต์ `support\Response` สร้างได้ผ่านฟังก์ชันช่วย เช่น `response()` `json()` `xml()` `jsonp()` `redirect()` ฯลฯ

## การผูกพารามิเตอร์ตัวควบคุม

#### ตัวอย่าง
webman รองรับการผูกพารามิเตอร์คำขอเข้ากับพารามิเตอร์เมธอดตัวควบคุมโดยอัตโนมัติ เช่น

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

คุณสามารถส่งค่า `name` และ `age` ผ่าน `GET` หรือ `POST` หรือผ่านพารามิเตอร์เส้นทาง เช่น

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

ลำดับความสำคัญ: `พารามิเตอร์เส้นทาง` > `GET` > `POST`

#### ค่าเริ่มต้น

สมมติเราเข้าถึง `/user/create?name=tom` เราจะได้รับข้อผิดพลาดดังนี้

```html
Missing input parameter age
```

เนื่องจากเราไม่ได้ส่งพารามิเตอร์ `age` สามารถแก้ได้โดยตั้งค่าเริ่มต้นให้พารามิเตอร์ เช่น

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

#### ประเภทพารามิเตอร์
เมื่อเข้าถึง `/user/create?name=tom&age=not_int` เราจะได้รับข้อผิดพลาดดังนี้

> **เคล็ดลับ**
> เพื่อความสะดวกในการทดสอบ เราใส่พารามิเตอร์ในแถบที่อยู่ของเบราว์เซอร์โดยตรง ในการพัฒนาจริงควรส่งพารามิเตอร์ผ่าน `POST`

```html
Input age must be of type int, string given
```

เนื่องจากการรับข้อมูลจะถูกแปลงตามประเภท หากแปลงไม่ได้จะโยน `support\exception\InputTypeException` เนื่องจากพารามิเตอร์ `age` ที่ส่งมาแปลงเป็น `int` ไม่ได้ จึงเกิดข้อผิดพลาดดังกล่าว

#### ข้อความผิดพลาดแบบกำหนดเอง
สามารถกำหนดข้อความเช่น `Missing input parameter age` และ `Input age must be of type int, string given` ได้ผ่านระบบหลายภาษา ดูคำสั่งดังนี้

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'พารามิเตอร์ :parameter ต้องเป็นประเภท :exceptType ประเภทที่ส่งมาคือ :actualType',
    'Missing input parameter :parameter' => 'ขาดพารามิเตอร์ :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### ประเภทอื่นๆ
webman รองรับประเภทพารามิเตอร์ ได้แก่ `int` `float` `string` `bool` `array` `object` และ `อินสแตนซ์คลาส` เช่น

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

เมื่อเข้าถึง `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar` เราจะได้ผลลัพธ์ดังนี้

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

#### อินสแตนซ์คลาส
webman รองรับการส่งอินสแตนซ์คลาสผ่าน type hint เช่น

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

เมื่อเข้าถึง `/blog/create?blog[title]=hello&blog[content]=world` เราจะได้ผลลัพธ์ดังนี้

```json
{
  "title": "hello",
  "content": "world"
}
```

#### อินสแตนซ์โมเดล

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
    // ต้องเพิ่มฟิลด์ที่เติมได้ที่นี่ เพื่อป้องกันฟิลด์ที่ไม่ปลอดภัยจากหน้าบ้าน
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

เมื่อเข้าถึง `/user/create?user[name]=tom&user[age]=18` เราจะได้ผลลัพธ์คล้ายๆ นี้

```json
1
```

## วงจรชีวิตตัวควบคุม

เมื่อ `controller_reuse` ใน `config/app.php` เป็น `false` แต่ละคำขอจะเริ่มต้นอินสแตนซ์ตัวควบคุมครั้งหนึ่ง เมื่อคำขอจบลงอินสแตนซ์ตัวควบคุมจะถูกทำลาย เหมือนกลไกการทำงานของเฟรมเวิร์กทั่วไป

เมื่อ `controller_reuse` ใน `config/app.php` เป็น `true` คำขอทั้งหมดจะใช้อินสแตนซ์ตัวควบคุมเดิม นั่นคืออินสแตนซ์ตัวควบคุมเมื่อสร้างแล้วจะอยู่ในหน่วยความจำ คำนขอทั้งหมดใช้ร่วมกัน

> **หมายเหตุ**
> เมื่อเปิดใช้การนำตัวควบคุมกลับมาใช้ คำขอไม่ควรเปลี่ยนคุณสมบัติใดๆ ของตัวควบคุม เพราะการเปลี่ยนแปลงดังกล่าวจะส่งผลต่อคำขอถัดไป เช่น

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
        // เมธอดนี้จะเก็บ model ไว้หลังคำขอแรก update?id=1
        // หากมีการขอ delete?id=2 อีกครั้ง จะลบข้อมูลของ id 1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **เคล็ดลับ**
> การ return ข้อมูลในคอนสตรัคเตอร์ `__construct()` ของตัวควบคุมจะไม่มีผล เช่น

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // การ return ข้อมูลในคอนสตรัคเตอร์ไม่มีผล เบราว์เซอร์จะไม่ได้รับการตอบสนองนี้
        return response('hello'); 
    }
}
```

## ความแตกต่างระหว่างไม่นำกลับมาใช้กับนำกลับมาใช้

#### ไม่นำตัวควบคุมกลับมาใช้
แต่ละคำขอจะ new อินสแตนซ์ตัวควบคุมใหม่ เมื่อคำขอจบลงจะปล่อยอินสแตนซ์นั้นและคืนหน่วยความจำ การไม่นำตัวควบคุมกลับมาใช้เหมือนเฟรมเวิร์กทั่วไป เหมาะกับพฤติกรรมของนักพัฒนาส่วนใหญ่ เนื่องจากตัวควบคุมถูกสร้างและทำลายซ้ำไปมา ประสิทธิภาพจะแย่กว่าแบบนำกลับมาใช้เล็กน้อย (การทดสอบ helloworld แย่ลงประมาณ 10% ถ้ามีธุรกิจจริงเกือบมองข้ามได้)

#### นำตัวควบคุมกลับมาใช้
การนำกลับมาใช้ โปรเซสหนึ่งจะ new ตัวควบคุมเพียงครั้งเดียว เมื่อคำขอจบลงจะไม่ปล่อยอินสแตนซ์ตัวควบคุมนี้ คำขอถัดไปของโปรเซสปัจจุบันจะใช้อินสแตนซ์เดิม ประสิทธิภาพดีกว่า แต่ไม่เหมาะกับพฤติกรรมของนักพัฒนาส่วนใหญ่

#### กรณีต่อไปนี้ไม่สามารถนำตัวควบคุมกลับมาใช้ได้

เมื่อคำขอจะเปลี่ยนคุณสมบัติของตัวควบคุม จะไม่สามารถเปิดใช้การนำตัวควบคุมกลับมาใช้ได้ เพราะการเปลี่ยนแปลงคุณสมบัติเหล่านั้นจะส่งผลต่อคำขอถัดไป

นักพัฒนาบางคนชอบทำการเริ่มต้นบางอย่างสำหรับแต่ละคำขอในคอนสตรัคเตอร์ `__construct()` ของตัวควบคุม ในกรณีนี้จะนำตัวควบคุมกลับมาใช้ไม่ได้ เพราะคอนสตรัคเตอร์ของโปรเซสปัจจุบันจะถูกเรียกเพียงครั้งเดียว ไม่ได้ถูกเรียกทุกคำขอ
