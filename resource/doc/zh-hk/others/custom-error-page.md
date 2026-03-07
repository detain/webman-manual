# 自定義404

如果你想動態控制404的內容，例如在 ajax 請求時返回 json 數據 `{"code:"404", "msg":"404 not found"}`，頁面請求時返回 `app/view/404.html` 模板，請參考如下示例。

> 以下以 php 原生模板為例，其他模板 `twig` `blade` `think-template` 原理類似。

**創建文件 `app/view/404.html`**
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

**在 `config/route.php` 中加入如下代碼：**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // ajax 請求時返回 json
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // 頁面請求返回 404.html 模板
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# 自定義405

從 webman-framework 1.5.23 開始，回調函數支持傳遞 status 參數。若 status 為 404 表示請求不存在，405 表示不支持當前請求方法（例如通過 GET 方式訪問由 `Route::post()` 設置的路由）。

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

# 自定義500

**新建 `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
自定義錯誤模板：
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**新建 `app/exception/Handler.php`（如目錄不存在請自行創建）**
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * 渲染返回
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // ajax 請求返回 json 數據
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // 頁面請求返回 500.html 模板
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**配置 `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
