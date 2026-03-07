# カスタム404

404の内容を動的に制御したい場合、例えば AJAX リクエストでは JSON データ `{"code:"404", "msg":"404 not found"}` を返し、ページリクエストでは `app/view/404.html` テンプレートを返す場合は、以下の例を参照してください。

> 以下は PHP ネイティブテンプレートの例です。`twig`、`blade`、`think-template` などの他のテンプレートも同様の原理です。

**`app/view/404.html` ファイルを作成**
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

**`config/route.php` に以下のコードを追加：**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // AJAX リクエストでは JSON を返す
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // ページリクエストでは 404.html テンプレートを返す
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# カスタム405

webman-framework 1.5.23 から、フォールバックコールバックは `status` パラメータをサポートします。404 はリクエストが存在しないこと、405 は現在のリクエストメソッドがサポートされていないことを示します（例：`Route::post()` で定義したルートを GET でアクセスする場合）。

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

# カスタム500

**`app/view/500.html` を新規作成**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
カスタムエラーテンプレート：
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**`app/exception/Handler.php` を新規作成**（ディレクトリが存在しない場合は作成してください）
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * レンダリングして返す
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // AJAX リクエストでは JSON データを返す
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // ページリクエストでは 500.html テンプレートを返す
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**`config/exception.php` を設定**
```php
return [
    '' => \app\exception\Handler::class,
];
```
