# 컨트롤러

새 컨트롤러 파일 `app/controller/FooController.php`를 생성합니다.

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

`http://127.0.0.1:8787/foo`에 접속하면 페이지는 `hello index`를 반환합니다.

`http://127.0.0.1:8787/foo/hello`에 접속하면 페이지는 `hello webman`을 반환합니다.

물론 라우트 구성을 통해 라우트 규칙을 변경할 수 있습니다. [라우트](route.md)를 참조하세요.

> **팁**
> 404 오류가 발생하면 `config/app.php`를 열어 `controller_suffix`를 `Controller`로 설정하고 다시 시작하세요.

## 컨트롤러 접미사
webman 1.3부터 `config/app.php`에서 컨트롤러 접미사를 설정할 수 있습니다. `config/app.php`에서 `controller_suffix`가 빈 문자열 `''`로 설정되어 있다면, 컨트롤러는 다음과 같이 작성됩니다.

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

컨트롤러 접미사를 `Controller`로 설정하는 것을 강력히 권장합니다. 이렇게 하면 컨트롤러와 모델 클래스 이름의 충돌을 피하고 보안을 강화할 수 있습니다.

## 설명
- 프레임워크는 자동으로 `support\Request` 객체를 컨트롤러에 전달하며, 이를 통해 사용자 입력 데이터(get, post, header, cookie 등)를 얻을 수 있습니다. [요청](request.md)을 참조하세요.
- 컨트롤러는 숫자, 문자열 또는 `support\Response` 객체를 반환할 수 있지만 다른 유형의 데이터는 반환할 수 없습니다.
- `support\Response` 객체는 `response()`, `json()`, `xml()`, `jsonp()`, `redirect()` 등의 헬퍼 함수를 통해 생성할 수 있습니다.

## 컨트롤러 매개변수 바인딩

#### 예시
webman은 컨트롤러 메서드 매개변수에 요청 매개변수를 자동으로 바인딩합니다. 예를 들어:

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

`name`과 `age`의 값을 `GET` 또는 `POST`로 전달할 수 있으며, 라우트 매개변수로도 전달할 수 있습니다. 예:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

우선순위는 `라우트 매개변수` > `GET` > `POST` 매개변수입니다.

#### 기본값

`/user/create?name=tom`에 접속하면 다음 오류가 발생합니다.

```html
Missing input parameter age
```

`age` 매개변수를 전달하지 않았기 때문입니다. 매개변수에 기본값을 설정하여 해결할 수 있습니다. 예:

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

#### 매개변수 타입
`/user/create?name=tom&age=not_int`에 접속하면 다음 오류가 발생합니다.

> **팁**
> 테스트 편의를 위해 브라우저 주소창에 직접 매개변수를 입력했습니다. 실제 개발에서는 `POST`로 매개변수를 전달해야 합니다.

```html
Input age must be of type int, string given
```

수신한 데이터는 타입에 따라 변환되며, 변환할 수 없으면 `support\exception\InputTypeException` 예외가 발생합니다. `age` 매개변수를 `int` 타입으로 변환할 수 없어 위 오류가 발생합니다.

#### 사용자 정의 오류 메시지
`Missing input parameter age` 및 `Input age must be of type int, string given`과 같은 오류 메시지는 다국어를 통해 사용자 정의할 수 있습니다. 다음 명령을 참조하세요.

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => '입력 매개변수 :parameter는 :exceptType 타입이어야 합니다. 전달된 타입은 :actualType입니다.',
    'Missing input parameter :parameter' => '입력 매개변수 :parameter가 누락되었습니다.',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### 기타 타입
webman이 지원하는 매개변수 타입은 `int`, `float`, `string`, `bool`, `array`, `object`, `클래스 인스턴스` 등입니다. 예:

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

`/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`에 접속하면 다음 결과를 얻습니다.

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

#### 클래스 인스턴스
webman은 매개변수 타입 힌트를 통해 클래스 인스턴스를 전달하는 것을 지원합니다. 예:

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

`/blog/create?blog[title]=hello&blog[content]=world`에 접속하면 다음 결과를 얻습니다.

```json
{
  "title": "hello",
  "content": "world"
}
```

#### 모델 인스턴스

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
    // 프론트엔드에서 안전하지 않은 필드가 전달되지 않도록 여기에 채울 수 있는 필드를 추가해야 합니다
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

`/user/create?user[name]=tom&user[age]=18`에 접속하면 다음과 비슷한 결과를 얻습니다.

```json
1
```

## 컨트롤러 라이프사이클

`config/app.php`에서 `controller_reuse`가 `false`로 설정되어 있을 때, 각 요청마다 해당 컨트롤러 인스턴스가 한 번 초기화되고, 요청 종료 후 컨트롤러 인스턴스가 파괴됩니다. 이것은 전통적인 프레임워크의 작동 방식과 동일합니다.

`config/app.php`에서 `controller_reuse`가 `true`로 설정되어 있을 때, 모든 요청이 컨트롤러 인스턴스를 재사용합니다. 즉, 컨트롤러 인스턴스가 생성되면 메모리에 상주하며 모든 요청에서 재사용됩니다.

> **참고**
> 컨트롤러 재사용이 활성화된 경우, 요청에서 컨트롤러의 속성을 변경하면 안 됩니다. 변경 사항이 후속 요청에 영향을 미치기 때문입니다. 예:

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
        // 이 메서드는 첫 번째 요청 update?id=1 후에 model을 유지합니다
        // delete?id=2를 요청하면 id=1의 데이터가 삭제됩니다
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **팁**
> 컨트롤러의 `__construct()` 생성자에서 데이터를 반환해도 효과가 없습니다. 예:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // 생성자에서 데이터를 반환해도 효과가 없습니다. 브라우저는 이 응답을 받지 않습니다
        return response('hello'); 
    }
}
```

## 컨트롤러 재사용과 미재사용의 차이
차이점은 다음과 같습니다.

#### 미재사용 컨트롤러
각 요청마다 새로운 컨트롤러 인스턴스가 생성되며, 요청 종료 후 해당 인스턴스가 해제되고 메모리가 회수됩니다. 미재사용은 전통적인 프레임워크와 같으며 대부분의 개발자 습관에 부합합니다. 컨트롤러의 반복적 생성과 파괴로 인해 성능이 재사용보다 약간 떨어집니다(helloworld 벤치마크에서 약 10% 정도, 실제 비즈니스에서는 거의 무시할 수 있음).

#### 재사용 컨트롤러
재사용 시 프로세스당 한 번만 컨트롤러가 생성되며, 요청 종료 후에도 이 컨트롤러 인스턴스는 해제되지 않습니다. 현재 프로세스의 후속 요청에서 이 인스턴스를 재사용합니다. 재사용은 성능이 더 우수하지만 대부분의 개발자 습관에는 부합하지 않습니다.

#### 다음의 경우 컨트롤러 재사용을 사용할 수 없습니다

요청에서 컨트롤러의 속성을 변경하는 경우, 컨트롤러 재사용을 활성화할 수 없습니다. 속성 변경이 후속 요청에 영향을 미치기 때문입니다.

일부 개발자는 컨트롤러 생성자 `__construct()`에서 각 요청에 대한 초기화를 수행하는 것을 선호합니다. 이 경우 컨트롤러 재사용을 사용할 수 없습니다. 현재 프로세스에서 생성자는 한 번만 호출되며 각 요청마다 호출되지 않기 때문입니다.
