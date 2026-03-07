# 미들웨어
일반적으로 미들웨어는 요청이나 응답을 가로채는 데 사용됩니다. 예를 들어 컨트롤러를 실행하기 전에 사용자 신원을 일관되게 확인하거나, 사용자가 로그인하지 않은 경우 로그인 페이지로 이동하도록 하는 등의 작업에 사용됩니다. 또한 응답에 헤더를 추가하거나 특정 URI 요청의 비율을 통계하는 등의 작업에도 사용됩니다.

## 미들웨어 양파 모델

```
                              
            ┌──────────────────────────────────────────────────────┐
            │                     미들웨어1                          │ 
            │     ┌──────────────────────────────────────────┐     │
            │     │               미들웨어2                    │     │
            │     │     ┌──────────────────────────────┐     │     │
            │     │     │         미들웨어3              │     │     │        
            │     │     │     ┌──────────────────┐     │     │     │
            │     │     │     │                  │     │     │     │
   ── 요청 ───────────────────────> 컨트롤러 ─ 응답 ───────────────────────────> 클라이언트
            │     │     │     │                  │     │     │     │
            │     │     │     └──────────────────┘     │     │     │
            │     │     │                              │     │     │
            │     │     └──────────────────────────────┘     │     │
            │     │                                          │     │
            │     └──────────────────────────────────────────┘     │
            │                                                      │
            └──────────────────────────────────────────────────────┘
```

미들웨어와 컨트롤러는 클래식한 양파 모델을 구성하며, 미들웨어는 양파 껍질처럼 층층으로 씌워지고, 컨트롤러는 양파 안쪽에 위치합니다. 그림에서 볼 수 있듯이 요청은 화살표처럼 미들웨어 1, 2, 3을 통과하여 컨트롤러에 도달하며, 컨트롤러가 응답을 반환하면 응답은 3, 2, 1의 순서로 다시 미들웨어를 통해 클라이언트에 반환됩니다. 즉, 각 미들웨어에서는 요청과 응답을 모두 얻을 수 있습니다.

## 요청 가로채기
가끔은 특정 요청이 컨트롤러 계층에 도달하지 않길 원할 때가 있습니다. 예를 들어 미들웨어 2에서 현재 사용자가 로그인되어 있지 않음을 발견했을 때 요청을 직접 가로채고 로그인 응답을 반환할 수 있습니다. 다음과 같이 이 과정이 진행됩니다.

```
                              
                              
            ┌────────────────────────────────────────────────────────────┐
            │                         미들웨어1                            │ 
            │     ┌────────────────────────────────────────────────┐     │
            │     │                   미들웨어2                      │     │
            │     │          ┌──────────────────────────────┐      │     │
            │     │          │        미들웨어3               │      │     │       
            │     │          │    ┌──────────────────┐      │      │     │
            │     │          │    │                  │      │      │     │
   ── 요청 ─────────────┐     │    │    컨트롤러       │      │      │     │
            │     │   응답    │    │                  │      │      │     │
   <───────────────────┘     │    └──────────────────┘      │      │     │
            │     │          │                              │      │     │
            │     │          └──────────────────────────────┘      │     │
            │     │                                                │     │
            │     └────────────────────────────────────────────────┘     │
            │                                                            │ 
            └────────────────────────────────────────────────────────────┘

```

그림에서 요청은 미들웨어 2에 도달한 후 로그인 응답이 생성됩니다. 이 응답은 미들웨어 2를 통해 다시 미들웨어 1에 전달되어 클라이언트에 반환됩니다.

## 미들웨어 인터페이스
미들웨어는 `Webman\MiddlewareInterface` 인터페이스를 구현해야 합니다.
```php
interface MiddlewareInterface
{
    /**
     * 서버 요청 처리
     *
     * 요청을 처리하여 응답을 생성합니다.
     * 응답을 직접 생성할 수 없는 경우, 제공된 요청 핸들러에 위임합니다.
     */
    public function process(Request $request, callable $handler): Response;
}
```
즉, `process` 메서드를 구현해야 하며, `process` 메서드는 `support\Response` 객체를 반환해야 합니다. 이 객체는 기본적으로 `$handler($request)`에 의해 생성되며(요청이 양파 안으로 계속 진행됨), `response()` `json()` `xml()` `redirect()` 등의 보조 함수에 의해 생성된 응답(요청 중지)일 수도 있습니다.
중간웨어에서는 요청을 받아들일 수도 있고, 컨트롤러 실행 후 응답을 받아들일 수도 있으므로 중간웨어 내부는 세 부분으로 나뉩니다.
1. 요청 전달 단계, 즉 요청 처리 전 단계
2. 컨트롤러 처리 요청 단계, 즉 요청 처리 단계
3. 응답 발신 단계, 즉 요청 처리 후 단계

중간웨어에서 3단계는 다음과 같이 나타납니다.

```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Test implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        echo '여기는 요청 전달 단계입니다. 즉, 요청 처리 전';
        
        $response = $handler($request); // 컨트롤러 실행하여 응답 받을 때까지 다음 단계로 전달
        
        echo '여기는 응답 발신 단계입니다. 즉, 요청 처리 후';
        
        return $response;
    }
}
```

## 예: 인증 확인 중간웨어
파일 `app/middleware/AuthCheckTest.php`를 생성합니다(디렉토리가 없다면 직접 생성하십시오).

```php
<?php
namespace app\middleware;

use ReflectionClass;
use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AuthCheckTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        if (session('user')) {
            // 로그인된 상태에서 요청을 계속해서 다음 단계로 전달
            return $handler($request);
        }

        // 리플렉션을 사용하여 컨트롤러 내에서 로그인이 필요하지 않은 메서드를 가져옴
        $controller = new ReflectionClass($request->controller);
        $noNeedLogin = $controller->getDefaultProperties()['noNeedLogin'] ?? [];

        // 요청한 메서드가 로그인이 필요한 경우
        if (!in_array($request->action, $noNeedLogin)) {
            // 요청을 차단하고 리다이렉션 응답을 반환하여 다음 단계로 전달하지 않음
            return redirect('/user/login');
        }

        // 로그인이 필요하지 않은 경우, 요청을 계속해서 다음 단계로 전달
        return $handler($request);
    }
}
```

새로운 컨트롤러 `app/controller/UserController.php`를 생성합니다.

```php
<?php
namespace app\controller;
use support\Request;

class UserController
{
    /**
     * 로그인이 필요하지 않은 메서드
     */
    protected $noNeedLogin = ['login'];

    public function login(Request $request)
    {
        $request->session()->set('user', ['id' => 10, 'name' => 'webman']);
        return json(['code' => 0, 'msg' => '로그인 완료']);
    }

    public function info()
    {
        return json(['code' => 0, 'msg' => '성공', 'data' => session('user')]);
    }
}
```

> **참고**
> `$noNeedLogin`에 현재 컨트롤러에서 로그인하지 않고도 액세스할 수 있는 메서드가 기록되어 있습니다.

`config/middleware.php`에 전역 중간웨어를 추가하십시오.

```php
return [
    // 전역 중간웨어
    '' => [
        // ... 기타 중간웨어는 여기에 생략
        app\middleware\AuthCheckTest::class,
    ]
];
```

인증 확인 중간웨어를 사용하면 컨트롤러 단계에서 사용자 로그인 여부에 대해 걱정하지 않고 비즈니스 코드를 작성할 수 있습니다.

## 예: CORS(교차 출처 리소스 공유) 요청 중간웨어
파일 `app/middleware/AccessControlTest.php`를 생성합니다(디렉토리가 없다면 직접 생성하십시오).

```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class AccessControlTest implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        // 옵션 요청인 경우 빈 응답을 반환하고, 아니면 계속 다음 단계로 전달하여 응답을 얻음
        $response = $request->method() == 'OPTIONS' ? response('') : $handler($request);
        
        // 응답에 교차 출처 관련 HTTP 헤더 추가
        $response->withHeaders([
            'Access-Control-Allow-Credentials' => 'true',
            'Access-Control-Allow-Origin' => $request->header('origin', '*'),
            'Access-Control-Allow-Methods' => $request->header('access-control-request-method', '*'),
            'Access-Control-Allow-Headers' => $request->header('access-control-request-headers', '*'),
        ]);
        
        return $response;
    }
}
```

> **팁**
> 교차 출처 리소스 공유(CORS)는 옵션 요청을 생성할 수 있습니다. 우리는 옵션 요청을 컨트롤러에 전달하고 싶지 않기 때문에 옵션 요청에 대해 빈 응답으로 처리합니다(`response('')`).
> 인터페이스를 설정해야 하는 경우 `Route::any(..)` 또는 `Route::add(['POST', 'OPTIONS'], ..)`을 사용하여 설정하십시오.

`config/middleware.php`에 중간웨어를 추가하십시오.

```php
return [
    // 전역 중간웨어
    '' => [
        // ... 기타 중간웨어는 여기에 생략
        app\middleware\AccessControlTest::class,
    ]
];
```

> **주의**
> Ajax 요청에서 사용자 지정 헤더를 설정하는 경우 중간웨어에서 `Access-Control-Allow-Headers` 필드에 이 사용자 지정 헤더를 추가해야 하며, 그렇지 않으면 `Request header field XXXX is not allowed by Access-Control-Allow-Headers in preflight response.` 오류가 발생할 수 있습니다.

## 설명

- 중간웨어에는 전역 중간웨어, 응용 프로그램 중간웨어(다중 응용 프로그램 모드에서만 유효, [다중 앱](multiapp.md) 참조) 및 라우팅 중간웨어가 있습니다.
- 중간웨어 구성 파일의 위치는 `config/middleware.php`입니다.
- 전역 중간웨어는 `''` 키 하위에 구성됩니다.
- 응용 프로그램 중간웨어는 각 응용 프로그램 이름 아래에 구성됩니다. 예시:

```php
return [
    // 전역 중간웨어
    '' => [
        app\middleware\AuthCheckTest::class,
        app\middleware\AccessControlTest::class,
    ],
    // API 응용 프로그램 중간웨어(다중 응용 프로그램 모드에서만 유효)
    'api' => [
        app\middleware\ApiOnly::class,
    ]
];
```

## 컨트롤러 중간웨어 및 메서드 중간웨어

어노테이션을 사용하여 특정 컨트롤러 또는 컨트롤러의 특정 메서드에 중간웨어를 설정할 수 있습니다.

```php
<?php
namespace app\controller;
use app\middleware\Controller1Middleware;
use app\middleware\Controller2Middleware;
use app\middleware\Method1Middleware;
use app\middleware\Method2Middleware;
use support\annotation\Middleware;
use support\Request;

#[Middleware(Controller1Middleware::class, Controller2Middleware::class)]
class IndexController
{
    #[Middleware(Method1Middleware::class, Method2Middleware::class)]
    public function index(Request $request): string
    {
        return 'hello';
    }
}
```

## 라우팅 중간웨어

특정 라우트 또는 라우트 그룹에 중간웨어를 설정할 수 있습니다.
예를 들어 `config/route.php`에서 다음 구성을 추가하십시오:

```php
<?php
use support\Request;
use Webman\Route;

Route::any('/admin', [app\admin\controller\IndexController::class, 'index'])->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);

Route::group('/blog', function () {
   Route::any('/create', function () {return response('create');});
   Route::any('/edit', function () {return response('edit');});
   Route::any('/view/{id}', function ($r, $id) {response("view $id");});
})->middleware([
    app\middleware\MiddlewareA::class,
    app\middleware\MiddlewareB::class,
]);
```
## 미들웨어 생성자 매개변수 전달

설정 파일에서 미들웨어를 직접 인스턴스화할 수 있어 생성자를 통해 매개변수를 편리하게 전달할 수 있습니다.
예를 들어 `config/middleware.php`에서 다음과 같이 구성할 수 있습니다.
```
return [
    // 전역 중간웨어
    '' => [
        new app\middleware\AuthCheckTest($param1, $param2, ...),
        new app\middleware\AccessControlTest($param1, $param2, ...)
    ],
    // API 애플리케이션 미들웨어(다중 애플리케이션 모드에서만 유효)
    'api' => [
        app\middleware\ApiOnly::class,
    ]
];
```

마찬가지로 라우팅 미들웨어도 생성자를 통해 매개변수를 전달할 수 있습니다. 예를 들어 `config/route.php`에서 다음과 같이 구성할 수 있습니다.
```
Route::any('/admin', [app\admin\controller\IndexController::class, 'index'])->middleware([
    new app\middleware\MiddlewareA($param1, $param2, ...),
    new app\middleware\MiddlewareB($param1, $param2, ...),
]);
```

중간웨어에서 매개변수 사용 예시
```
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class MiddlewareA implements MiddlewareInterface
{
    protected $param1;

    protected $param2;

    public function __construct($param1, $param2)
    {
        $this->param1 = $param1;
        $this->param2 = $param2;
    }

    public function process(Request $request, callable $handler) : Response
    {
        var_dump($this->param1, $this->param2);
        return $handler($request);
    }
}
```

## 미들웨어 실행 순서
- 미들웨어 실행 순서는 `전역 미들웨어` -> `애플리케이션 미들웨어` -> `컨트롤러 미들웨어` -> `라우팅 미들웨어` -> `메서드 미들웨어` 순입니다.
- 동일한 계층에 여러 미들웨어가 있는 경우 해당 계층의 미들웨어 실제 구성 순서대로 실행됩니다.
- 404 요청은 기본적으로 어떤 미들웨어도 트리거하지 않습니다(단, `Route::fallback(function(){})->middleware()`로 미들웨어를 추가할 수 있습니다).

## 라우팅을 통해 미들웨어에 매개변수 전달(route->setParams)

**라우팅 구성 `config/route.php`**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/test', [app\controller\IndexController::class, 'index'])->setParams(['some_key' =>'some value']);
```

**미들웨어(전역 미들웨어로 가정)**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        // 기본 라우팅 $request->route 는 null이므로 $request->route가 비어 있는지 확인해야 합니다.
        if ($route = $request->route) {
            $value = $route->param('some_key');
            var_export($value);
        }
        return $handler($request);
    }
}
```

## 미들웨어에서 컨트롤러에 매개변수 전달

가끔 미들웨어에서 생성된 데이터를 컨트롤러에서 사용해야 할 때, `$request` 객체에 속성을 추가하여 컨트롤러에 전달할 수 있습니다. 예를 들어:

**미들웨어**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $request->data = 'some value';
        return $handler($request);
    }
}
```

**컨트롤러:**
```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response($request->data);
    }
}
```

## 미들웨어에서 현재 요청 라우팅 정보 가져오기

`$request->route`를 사용하여 라우팅 객체를 가져와 해당 메서드를 호출하여 관련 정보를 가져올 수 있습니다.

**라우팅 구성**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/user/{uid}', [app\controller\UserController::class, 'view']);
```

**미들웨어**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $route = $request->route;
        // 요청이 어떤 라우팅과 일치하지 않는 경우(기본 라우팅 제외), $request->route는 null입니다
        // 브라우저가 주소 /user/111을 방문하면 다음과 같은 정보를 출력합니다.
        if ($route) {
            var_export($route->getPath());       // /user/{uid}
            var_export($route->getMethods());    // ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD','OPTIONS']
            var_export($route->getName());       // user_view
            var_export($route->getMiddleware()); // []
            var_export($route->getCallback());   // ['app\\controller\\UserController', 'view']
            var_export($route->param());         // ['uid'=>111]
            var_export($route->param('uid'));    // 111 
        }
        return $handler($request);
    }
}
```

## 미들웨어에서 예외 가져오기

비즈니스 처리 중에 예외가 발생할 수 있으며, 미들웨어에서 `$response->exception()`을 사용하여 예외를 가져올 수 있습니다.

**라우팅 구성**
```php
<?php
use support\Request;
use Webman\Route;

Route::any('/user/{uid}', function (Request $request, $uid) {
    throw new \Exception('exception test');
});
```

**미들웨어:**
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Hello implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        $response = $handler($request);
        $exception = $response->exception();
        if ($exception) {
            echo $exception->getMessage();
        }
        return $response;
    }
}
```

## 초글로벌 미들웨어

메인 프로젝트의 전역 미들웨어는 메인 프로젝트에만 영향을 미치며 [애플리케이션 플러그인](app/app.md)에는 영향을 주지 않습니다. 때로는 모든 플러그인을 포함한 전체에 영향을 주는 미들웨어를 추가하고 싶을 수 있으며, 이 경우 초글로벌 미들웨어를 사용할 수 있습니다.

`config/middleware.php`에서 다음과 같이 구성합니다:

```php
return [
    '@' => [ // 메인 프로젝트 및 모든 플러그인에 전역 미들웨어 추가
        app\middleware\MiddlewareGlobl::class,
    ], 
    '' => [], // 메인 프로젝트에만 전역 미들웨어 추가
];
```

> **팁**
> `@` 초글로벌 미들웨어는 메인 프로젝트뿐만 아니라 플러그인에서도 구성할 수 있습니다. 예를 들어 `plugin/ai/config/middleware.php`에서 `@` 초글로벌 미들웨어를 구성하면 메인 프로젝트 및 모든 플러그인에 영향을 미칩니다.

## 특정 플러그인에 미들웨어 추가하기

때로는 특정 [애플리케이션 플러그인](app/app.md)에 미들웨어를 추가하고 싶지만 플러그인 코드를 수정하고 싶지 않을 수 있습니다(업그레이드 시 덮어쓰여지므로). 이 경우 메인 프로젝트에서 해당 플러그인에 미들웨어를 구성할 수 있습니다.

`config/middleware.php`에서 다음과 같이 구성합니다:

```php
return [
    'plugin.ai' => [], // ai 플러그인에 미들웨어 추가
    'plugin.ai.admin' => [], // ai 플러그인의 admin 모듈(plugin\ai\app\admin 디렉토리)에 미들웨어 추가
];
```

> **팁**
> 플러그인에서도 유사한 구성을 추가하여 다른 플러그인에 영향을 줄 수 있습니다. 예를 들어 `plugin/foo/config/middleware.php`에 위와 같은 구성을 추가하면 ai 플러그인에 영향을 미칩니다.
