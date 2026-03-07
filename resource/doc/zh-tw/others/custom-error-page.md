# 自訂404

若您想動態控制 404 的內容，例如在 ajax 請求時回傳 json 資料 `{"code:"404", "msg":"404 not found"}`，頁面請求時回傳 `app/view/404.html` 模板，請參考以下範例。

> 以下以 php 原生模板為例，其他模板 `twig`、`blade`、`think-template` 原理類似。

**建立檔案 `app/view/404.html`**
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

**在 `config/route.php` 中加入以下程式碼：**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // ajax 請求時回傳 json
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // 頁面請求回傳 404.html 模板
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# 自訂405

自 webman-framework 1.5.23 起，回呼函數支援傳遞 status 參數。status 為 404 表示請求不存在，405 表示不支援目前的請求方法（例如以 GET 方式存取由 `Route::post()` 設定的路由）。

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

# 自訂500

**新建 `app/view/500.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
自訂錯誤模板：
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**新建 `app/exception/Handler.php`（如目錄不存在請自行建立）**
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * 渲染回傳
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // ajax 請求回傳 json 資料
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // 頁面請求回傳 500.html 模板
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**設定 `config/exception.php`**
```php
return [
    '' => \app\exception\Handler::class,
];
```
