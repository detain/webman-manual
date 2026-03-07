# webmanの静的ファイル処理

webmanは静的ファイルへのアクセスをサポートしており、静的ファイルはすべて`public`ディレクトリに配置されています。例えば、`http://127.0.0.8787/upload/avatar.png`にアクセスすると、実際には`{プロジェクトルートディレクトリ}/public/upload/avatar.png`にアクセスします。

> **注意**
> `/app/xx/ファイル名`で始まる静的ファイルへのアクセスは、実際にはアプリケーションプラグインの`public`ディレクトリにアクセスします。つまり、`{プロジェクトルートディレクトリ}/public/app/`以下のディレクトリへのアクセスはサポートされていません。
> 詳細は[アプリケーションプラグイン](./plugin/app.md)を参照してください。

## 静的ファイルサポートの無効化

静的ファイルのサポートが不要な場合は、`config/static.php`を開き、`enable`オプションをfalseに変更してください。無効化すると、すべての静的ファイルへのアクセスは404を返します。

## 静的ファイルディレクトリの変更

webmanでは、デフォルトで`public`ディレクトリが静的ファイルディレクトリとして使用されます。変更が必要な場合は、`support/helpers.php`内の`public_path()`ヘルパー関数を編集してください。

## 静的ファイルミドルウェア

webmanには、`app/middleware/StaticFile.php`に配置された静的ファイルミドルウェアが付属しています。
静的ファイルに対して特定の処理を行いたい場合（例：クロスオリジンのHTTPヘッダーを追加する、ドット（`.`）で始まるファイルへのアクセスを禁止するなど）は、このミドルウェアを使用できます。

`app/middleware/StaticFile.php`の内容は以下のようになります：
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
        // ドットで始まる隠しファイルへのアクセスを禁止
        if (strpos($request->path(), '/.') !== false) {
            return response('<h1>403 forbidden</h1>', 403);
        }
        /** @var Response $response */
        $response = $next($request);
        // クロスオリジンのヘッダーを追加
        /*$response->withHeaders([
            'Access-Control-Allow-Origin'      => '*',
            'Access-Control-Allow-Credentials' => 'true',
        ]);*/
        return $response;
    }
}
```
このミドルウェアが必要な場合は、`config/static.php`の`middleware`オプションで有効にしてください。
