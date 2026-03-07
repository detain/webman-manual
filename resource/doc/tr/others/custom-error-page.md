# Özel 404

404 içeriğini dinamik olarak kontrol etmek istiyorsanız, örneğin AJAX isteklerinde JSON verisi `{"code:"404", "msg":"404 not found"}` döndürmek ve sayfa isteklerinde `app/view/404.html` şablonunu döndürmek istiyorsanız aşağıdaki örneğe bakınız.

> Aşağıda PHP native şablon örneği kullanılmaktadır. `twig`, `blade`, `think-template` gibi diğer şablonlar da aynı prensibi izler.

**`app/view/404.html` dosyasını oluşturun**
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

**`config/route.php` dosyasına aşağıdaki kodu ekleyin:**
```php
use support\Request;
use Webman\Route;

Route::fallback(function(Request $request){
    // AJAX isteklerinde JSON döndür
    if ($request->expectsJson()) {
        return json(['code' => 404, 'msg' => '404 not found']);
    }
    // Sayfa isteklerinde 404.html şablonunu döndür
    return view('404', ['error' => 'some error'])->withStatus(404);
});
```

# Özel 405

webman-framework 1.5.23 sürümünden itibaren fallback callback `status` parametresini destekler. 404 isteğin mevcut olmadığını, 405 ise mevcut istek yönteminin desteklenmediğini belirtir (örn. `Route::post()` ile tanımlanmış bir rotaya GET ile erişmek).

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

# Özel 500

**`app/view/500.html` oluşturun**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>500 Internal Server Error</title>
</head>
<body>
Özel hata şablonu:
<?=htmlspecialchars($exception)?>
</body>
</html>
```

**`app/exception/Handler.php` oluşturun** (klasör yoksa kendiniz oluşturun)
```php
<?php

namespace app\exception;

use Throwable;
use Webman\Http\Request;
use Webman\Http\Response;

class Handler extends \support\exception\Handler
{
    /**
     * Yanıtı render edip döndür
     * @param Request $request
     * @param Throwable $exception
     * @return Response
     */
    public function render(Request $request, Throwable $exception) : Response
    {
        $code = $exception->getCode();
        // AJAX isteklerinde JSON verisi döndür
        if ($request->expectsJson()) {
            return json(['code' => $code ? $code : 500, 'msg' => $exception->getMessage()]);
        }
        // Sayfa isteklerinde 500.html şablonunu döndür
        return view('500', ['exception' => $exception], '')->withStatus(500);
    }
}
```

**`config/exception.php` dosyasını yapılandırın**
```php
return [
    '' => \app\exception\Handler::class,
];
```
