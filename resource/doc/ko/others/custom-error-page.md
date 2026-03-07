# 사용자 정의 404

404 내용을 동적으로 제어하려면, 예를 들어 AJAX 요청 시 JSON 데이터 `{"code:"404", "msg":"404 not found"}`를 반환하고 페이지 요청 시 `app/view/404.html` 템플릿을 반환하려면 아래 예제를 참조하세요.

> 아래 예시는 PHP 네이티브 템플릿을 사용합니다. `twig`, `blade`, `think-template` 등의 다른 템플릿도 동일한 원리를 따릅니다.

**`app/view/404.html` 파일 생성**
```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>404 not found</title>
</head>
<body>
<?=htmlspecialchars($error)?>
</body>
</html>
```

**`config/route.php`에 아래 코드 추가:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // AJAX 요청 시 JSON 반환
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // 페이지 요청 시 404.html 템플릿 반환
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# 사용자 정의 405

webman-framework 1.5.23부터 폴백 콜백은 `status` 매개변수를 지원합니다. 404는 요청이 존재하지 않음을, 405는 현재 요청 메서드가 지원되지 않음을 의미합니다 (예: `Route::post()`로 정의한 라우트를 GET으로 접근하는 경우).

```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request, $status) {
    $map = [
        404 => '404 not found',
        405 => '405 method not allowed',
    ];
    return response($map[$status], $status);
});
```

# 사용자 정의 500

**`app/view/500.html` 생성**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
사용자 정의 오류 템플릿:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**`app/exception/Handler.php` 생성** (디렉터리가 없으면 직접 생성)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * 렌더링하여 반환
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // AJAX 요청 시 JSON 데이터 반환
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // 페이지 요청 시 500.html 템플릿 반환
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**`config/exception.php` 설정**
```php
return [
    '' => \app\exception\Handler::class,
];
```
