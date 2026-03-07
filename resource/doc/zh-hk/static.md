# webman處理靜態檔案

webman支援靜態檔案存取，靜態檔案都放置於`public`目錄下。例如存取`http://127.0.0.8787/upload/avatar.png`實際上是存取`{主項目目錄}/public/upload/avatar.png`。

> **注意**
> 以`/app/xx/檔案名`開頭的靜態檔案存取實際上是存取應用插件的`public`目錄，也就是說不支援`{主項目目錄}/public/app/`下的目錄存取。
> 更多請參考[應用插件](./plugin/app.md)。

## 關閉靜態檔案支援

如果不需要靜態檔案支援，請開啟`config/static.php`並將`enable`選項改為false。關閉後所有靜態檔案的存取將會回傳404。

## 更改靜態檔案目錄

webman預設使用`public`目錄作為靜態檔案目錄。如需修改，請編輯`support/helpers.php`中的`public_path()`輔助函數。

## 靜態檔案中介層

webman內建一個靜態檔案中介層，位置在`app/middleware/StaticFile.php`。
有時我們需要對靜態檔案進行一些處理，例如為靜態檔案增加跨域HTTP標頭，或禁止存取以點（`.`）開頭的檔案，可以使用這個中介層。

`app/middleware/StaticFile.php` 內容類似如下：
```php
<?php
namespace support\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class StaticFile implements MiddlewareInterface
{
    public function process(Request $request, callable $next) : Response
    {
        // 禁止存取以點開頭的隱藏檔案
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // 增加跨域HTTP標頭
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
若需要此中介層，請在`config/static.php`中的`middleware`選項啟用。
