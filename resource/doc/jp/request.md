# 説明

## リクエストオブジェクトの取得
webmanは自動的にリクエストオブジェクトをアクションメソッドの最初のパラメータに注入します。例えば、

**例**
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $default_name = 'webman';
        // クエリ文字列からnameパラメータを取得します。もしnameパラメータが渡されていない場合は$default_nameを返します。
        $name = $request->get('name', $default_name);
        // 文字列をブラウザに返します
        return response('hello ' . $name);
    }
}
```

`$request`オブジェクトを使用することで、リクエストに関連する任意のデータを取得できます。

**時々、他のクラスで現在のリクエストの`$request`オブジェクトを取得したい場合は、`request()`ヘルパー関数を使用します。**

## GETパラメータの取得
**完全なGET配列を取得**
```php
$request->get();
```
リクエストにGETパラメータがない場合は空の配列が返されます。

**GET配列の値を取得**
```php
$request->get('name');
```
GET配列にこの値が含まれていない場合はnullが返されます。

また、第2引数にデフォルト値を指定して、対応する値が見つからない場合にはデフォルト値を返すこともできます。例：
```php
$request->get('name', 'tom');
```

## POSTパラメータの取得
**完全なPOST配列を取得**
```php
$request->post();
```
リクエストにPOSTパラメータがない場合は空の配列が返されます。

**POST配列の値を取得**
```php
$request->post('name');
```
POST配列にこの値が含まれていない場合はnullが返されます。

GET方法と同様に、第2引数にデフォルト値を指定して、対応する値が見つからない場合にはデフォルト値を返すこともできます。例：
```php
$request->post('name', 'tom');
```

## ヘルパー関数input()
`$request->input()`と同様に、すべてのパラメータを取得できます。input()ヘルパー関数には2つのパラメータがあります：
1. name: 取得するパラメータ名（空の場合はすべてのパラメータの配列を取得）
2. default: デフォルト値（第1引数で取得に失敗した場合に使用）

**例**
```php
// パラメータnameを取得
$name = input('name');
// パラメータnameを取得し、存在しない場合はデフォルト値を使用
$name = input('name', '山田');
// すべてのパラメータを取得
$all_params = input();
```

## 原始リクエストのPOSTボディの取得
```php
$post = $request->rawBody();
```
これは `php-fpm` の `file_get_contents("php://input");` に似た機能です。 `application/x-www-form-urlencoded` 形式以外のPOSTリクエストデータを取得する際に便利です。

## ヘッダーの取得
**完全なヘッダー配列を取得**
```php
$request->header();
```
リクエストにヘッダーパラメータがない場合は空の配列が返されます。すべてのキーは小文字です。

**ヘッダー配列の値を取得**
```php
$request->header('host');
```
ヘッダー配列にこの値が含まれていない場合はnullが返されます。すべてのキーは小文字です。

GETメソッドと同様に、第2引数にデフォルト値を指定して、対応する値が見つからない場合にはデフォルト値を返すこともできます。例：
```php
$request->header('host', 'localhost');
```

## クッキーの取得
**完全なクッキー配列を取得**
```php
$request->cookie();
```
リクエストにクッキーパラメータがない場合は空の配列が返されます。

**クッキー配列の値を取得**
```php
$request->cookie('name');
```
クッキー配列にこの値が含まれていない場合はnullが返されます。

GETメソッドと同様に、第2引数にデフォルト値を指定して、対応する値が見つからない場合にはデフォルト値を返すこともできます。例：
```php
$request->cookie('name', 'tom');
```

## すべての入力を取得
`post` `get` の集合を含みます。
```php
$request->all();
```

## 特定の入力値を取得
`post` `get` の集合から特定の値を取得します。
```php
$request->input('name', $default_value);
```

## 一部の入力データを取得
`post` `get` の集合から一部のデータを取得します。
```php
// usernameおよびpasswordからなる配列を取得し、対応するキーがない場合は無視します
$only = $request->only(['username', 'password']);
// avatarおよびage以外のすべての入力を取得します
$except = $request->except(['avatar', 'age']);
```

## コントローラーパラメータによる入力の取得

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
上記のロジックは以下と同等です：
```php
<?php
namespace app\controller;
use support\Request;
use support\Response;

class UserController
{
    public function create(Request $request): Response
    {
        $name = $request->input('name');
        $age = (int)$request->input('age', 18);
        return json(['name' => $name, 'age' => $age]);
    }
}
```
詳細は[コントローラーパラメータバインディング](controller.md#コントローラーパラメータバインディング)を参照してください。

## アップロードファイルの取得

> **ヒント**
> ファイルのアップロードには `multipart/form-data` 形式のフォームが必要です。

**完全なアップロードファイル配列を取得**
```php
$request->file();
```

フォームの例：
```html
<form method="post" action="http://127.0.0.1:8787/upload/files" enctype="multipart/form-data">
<input name="file1" multiple="multiple" type="file">
<input name="file2" multiple="multiple" type="file">
<input type="submit">
</form>
```

`$request->file()`が返す形式は以下のようです:
```php
array (
    'file1' => object(webman\Http\UploadFile),
    'file2' => object(webman\Http\UploadFile)
)
```
これは`webman\Http\UploadFile`インスタンスの配列です。 `webman\Http\UploadFile`クラスはPHP組み込みの [`SplFileInfo`](https://www.php.net/manual/zh/class.splfileinfo.php) クラスを継承しており、便利なメソッドを提供しています。

```php
<?php
namespace app\controller;

use support\Request;

class UploadController
{
    public function files(Request $request)
    {
        foreach ($request->file() as $key => $spl_file) {
            var_export($spl_file->isValid()); // ファイルが有効かどうか、例：true|false
            var_export($spl_file->getUploadExtension()); // アップロードファイルの拡張子、例：'jpg'
            var_export($spl_file->getUploadMimeType()); // アップロードファイルのMIMEタイプ、例：'image/jpeg'
            var_export($spl_file->getUploadErrorCode()); // アップロードエラーコードを取得、例：UPLOAD_ERR_NO_TMP_DIR UPLOAD_ERR_NO_FILE UPLOAD_ERR_CANT_WRITE
            var_export($spl_file->getUploadName()); // アップロードファイル名、例：'my-test.jpg'
            var_export($spl_file->getSize()); // ファイルサイズを取得、例：13364、単位はバイト
            var_export($spl_file->getPath()); // アップロードディレクトリを取得、例：'/tmp'
            var_export($spl_file->getRealPath()); // 一時ファイルパスを取得、例：`/tmp/workerman.upload.SRliMu`
        }
        return response('ok');
    }
}
```

**注意：**

- ファイルはアップロードされた後、一時ファイルに命名されます（例： `/tmp/workerman.upload.SRliMu`）
- アップロードファイルのサイズは[defaultMaxPackageSize](http://doc.workerman.net/tcp-connection/default-max-package-size.html)によって制限されます。デフォルトは10Mであり、`config/server.php` ファイルで `max_package_size` を変更することでデフォルト値を変更できます。
- リクエストが終了すると、一時ファイルが自動的に削除されます。
- リクエストにアップロードファイルがない場合、`$request->file()`は空の配列を返します。
- アップロードされたファイルは `move_uploaded_file()` メソッドをサポートしていません。代わりに `$file->move()` メソッドを使用してください。以下の例を参照してください。

### 特定のアップロードファイルを取得
```php
$request->file('avatar');
```
ファイルが存在する場合は、対応するファイルの`webman\Http\UploadFile`インスタンスが返されます。存在しない場合はnullが返されます。

**例**
```php
<?php
namespace app\controller;

use support\Request;

class UploadController
{
    public function file(Request $request)
    {
        $file = $request->file('avatar');
        if ($file && $file->isValid()) {
            $file->move(public_path().'/files/myfile.'.$file->getUploadExtension());
            return json(['code' => 0, 'msg' => 'upload success']);
        }
        return json(['code' => 1, 'msg' => 'file not found']);
    }
}
```
## ホストを取得する
リクエストのホスト情報を取得します。
```php
$request->host();
```
リクエストのアドレスが標準ではない80または443ポートである場合、ホスト情報にはポートが含まれることがあります。例えば`example.com:8080`のような形です。ポートを除外する場合は、第1引数に`true`を渡します。

```php
$request->host(true);
```

## リクエストメソッドを取得する
```php
$request->method();
```
戻り値は`GET`、`POST`、`PUT`、`DELETE`、`OPTIONS`、`HEAD`のいずれかになります。

## リクエストURIを取得する
```php
$request->uri();
```
パスとクエリ文字列の両方を含むリクエストのURIを返します。

## リクエストパスを取得する

```php
$request->path();
```
リクエストのパス部分を返します。


## リクエストクエリ文字列を取得する

```php
$request->queryString();
```
リクエストのクエリ文字列部分を返します。

## リクエストURLを取得する
`url()`メソッドは`Query`パラメータを含まないURLを返します。
```php
$request->url();
```
`//www.workerman.net/workerman-chat`のような形で返されます。

`fullUrl()`メソッドは`Query`パラメータを含むURLを返します。
```php
$request->fullUrl();
```
`//www.workerman.net/workerman-chat?type=download`のような形で返されます。

> **注意**
> `url()`と`fullUrl()`はプロトコル部分（httpまたはhttps）を含みません。
> ブラウザでは、`//example.com`のように`//`で始まるアドレスを使用すると、自動的に現在のサイトのプロトコルを認識し、httpまたはhttpsでリクエストが自動的に行われます。

Nginxプロキシを使用している場合は、`proxy_set_header X-Forwarded-Proto $scheme;`をNginxの設定に追加する必要があります。[Nginxプロキシの参考](others/nginx-proxy.md)、これにより`$request->header('x-forwarded-proto');`を使用してhttpまたはhttpsを判断できます。例えば：
```php
echo $request->header('x-forwarded-proto'); // httpまたはhttpsが出力されます
```

## リクエストHTTPバージョンの取得

```php
$request->protocolVersion();
```
文字列 `1.1` または `1.0` を返します。

## リクエストのセッションIDの取得

```php
$request->sessionId();
```
英数字で構成される文字列を返します。

## クライアントIPの取得
```php
$request->getRemoteIp();
```

## クライアントポートの取得
```php
$request->getRemotePort();
```

## クライアントの実IPの取得
```php
$request->getRealIp($safe_mode=true);
```

プロキシ（例：nginx）を使用している場合、`$request->getRemoteIp()`で取得できるのはプロキシサーバーのIP（`127.0.0.1`や`192.168.x.x`など）になることが多く、クライアントの実IPではありません。このような場合は、`$request->getRealIp()`を使用してクライアントの実IPを取得できます。

`$request->getRealIp()`はHTTPヘッダーの`x-forwarded-for`、`x-real-ip`、`client-ip`、`x-client-ip`、`via`フィールドから実IPを取得しようとします。

> HTTPヘッダーは偽造が容易なため、このメソッドで取得したクライアントIPは100%信頼できません。特に`$safe_mode`がfalseの場合です。プロキシ経由でクライアントの実IPを取得する比較的可靠な方法は、安全なプロキシサーバーのIPを把握し、実IPを格納しているHTTPヘッダーを明確に知っていることです。`$request->getRemoteIp()`が返すIPが既知の安全なプロキシサーバーと確認できた場合、`$request->header('実IPを格納するHTTPヘッダー')`で実IPを取得します。

## サーバーIPの取得
```php
$request->getLocalIp();
```

## サーバーポートの取得
```php
$request->getLocalPort();
```

## AJAXリクエストかどうかの判定
```php
$request->isAjax();
```

## PJAXリクエストかどうかの判定
```php
$request->isPjax();
```

## JSONレスポンスを期待しているかどうかの判定
```php
$request->expectsJson();
```

## クライアントがJSONレスポンスを受け入れるかどうかの判定
```php
$request->acceptJson();
```

## リクエストされたプラグイン名の取得
プラグイン以外のリクエストの場合は空文字列`''`を返します。
```php
$request->plugin;
```

## リクエストされたアプリケーション名の取得
単一アプリケーションの場合は常に空文字列`''`を返します。[マルチアプリケーション](multiapp.md)の場合はアプリケーション名を返します。
```php
$request->app;
```
> クロージャはアプリケーションに属さないため、クロージャルートからのリクエストでは`$request->app`は常に空文字列`''`を返します。
> クロージャルートについては[ルート](route.md)を参照してください。

## リクエストされたコントローラークラス名の取得
コントローラーに対応するクラス名を取得します。
```php
$request->controller;
```
`app\controller\IndexController`のような形式で返します。

> クロージャはコントローラーに属さないため、クロージャルートからのリクエストでは`$request->controller`は常に空文字列`''`を返します。
> クロージャルートについては[ルート](route.md)を参照してください。

## リクエストされたメソッド名の取得
リクエストに対応するコントローラーメソッド名を取得します。
```php
$request->action;
```
`index`のような形式で返します。

> クロージャはコントローラーに属さないため、クロージャルートからのリクエストでは`$request->action`は常に空文字列`''`を返します。
> クロージャルートについては[ルート](route.md)を参照してください。

## パラメータの上書き
リクエストのパラメータをフィルタリングしてからリクエストオブジェクトに再代入したい場合があります。そのような場合は、`setGet()`、`setPost()`、`setHeader()`メソッドを使用できます。

#### GETパラメータの上書き
```php
$request->get(); // 仮に ['name' => 'tom', 'age' => 18] が返されるとする
$request->setGet(['name' => 'tom']);
$request->get(); // 最終的に ['name' => 'tom'] が返される
```

> **注意**
> 例のとおり、`setGet()`はすべてのGETパラメータを上書きします。`setPost()`、`setHeader()`も同様の動作です。

#### POSTパラメータの上書き
```php
$post = $request->post();
foreach ($post as $key => $value) {
    $post[$key] = htmlspecialchars($value);
}
$request->setPost($post);
$request->post(); // フィルタリング後のpostパラメータを取得
```

#### HEADERパラメータの上書き
```php
$request->setHeader(['host' => 'example.com']);
$request->header('host'); // 出力: example.com
```
