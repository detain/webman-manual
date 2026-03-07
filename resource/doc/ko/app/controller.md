# 컨트롤러

PSR4 규격에 따라 컨트롤러 클래스 네임스페이스는 `plugin\{플러그인 식별자}`로 시작합니다. 예를 들어,

새 컨트롤러 파일 `plugin/foo/app/controller/FooController.php`를 만듭니다.

```php
<?php
namespace plugin\foo\app\controller;

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

`http://127.0.0.1:8787/app/foo/foo`에 접속하면 페이지가 `hello index`를 반환합니다.

`http://127.0.0.1:8787/app/foo/foo/hello`에 접속하면 페이지가 `hello webman`을 반환합니다.


## URL 접근
애플리케이션 플러그인 URL 주소 경로는 모두 `/app`로 시작하여 플러그인 식별자를 두고, 그 다음에는 구체적인 컨트롤러 및 메소드가 따릅니다.
예를 들어, `plugin\foo\app\controller\UserController`의 URL 주소는 `http://127.0.0.1:8787/app/foo/user`입니다.
