# 多言語

多言語機能には [symfony/translation](https://github.com/symfony/translation) コンポーネントを使用します。

## インストール
```
composer require symfony/translation
```

## 言語パッケージの作成
webman はデフォルトで言語パッケージを `resource/translations` ディレクトリに配置します（存在しない場合は作成してください）。ディレクトリを変更する場合は、`config/translation.php` で設定してください。
各言語はサブフォルダに対応し、言語定義はデフォルトで `messages.php` に配置されます。例は以下のとおりです：
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

すべての言語ファイルは配列を返します。例：
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## 設定

`config/translation.php`

```php
return [
    // デフォルト言語
    'locale' => 'zh_CN',
    // フォールバック言語。現在の言語で翻訳が見つからない場合、フォールバック言語の翻訳を試行します
    'fallback_locale' => ['zh_CN', 'en'],
    // 言語ファイルを格納するフォルダ
    'path' => base_path() . '/resource/translations',
];
```

## 翻訳

翻訳には `trans()` メソッドを使用します。

言語ファイル `resource/translations/zh_CN/messages.php` を以下のように作成します：
```php
return [
    'hello' => '你好 世界!',
];
```

ファイル `app/controller/UserController.php` を作成します：
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        $hello = trans('hello'); // 你好 世界!
        return response($hello);
    }
}
```

`http://127.0.0.1:8787/user/get` にアクセスすると「你好 世界!」が返されます。

## デフォルト言語の変更

言語の切り替えには `locale()` メソッドを使用します。

言語ファイル `resource/translations/en/messages.php` を以下のように追加します：
```php
return [
    'hello' => 'hello world!',
];
```

```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 言語を切り替える
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
`http://127.0.0.1:8787/user/get` にアクセスすると「hello world!」が返されます。

`trans()` 関数の第4引数で一時的に言語を切り替えることもできます。上記の例は以下の例と等価です：
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 第4引数で言語を切り替える
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## 各リクエストで明示的に言語を設定する
translation はシングルトンであるため、すべてのリクエストがこのインスタンスを共有します。あるリクエストが `locale()` でデフォルト言語を設定すると、そのプロセスの後続するすべてのリクエストに影響します。そのため、各リクエストで明示的に言語を設定する必要があります。例えば、以下のミドルウェアを使用します。

ファイル `app/middleware/Lang.php` を作成します（ディレクトリが存在しない場合は作成してください）：
```php
<?php
namespace app\middleware;

use Webman\MiddlewareInterface;
use Webman\Http\Response;
use Webman\Http\Request;

class Lang implements MiddlewareInterface
{
    public function process(Request $request, callable $handler) : Response
    {
        locale(session('lang', 'zh_CN'));
        return $handler($request);
    }
}
```

`config/middleware.php` にグローバルミドルウェアを以下のように追加します：
```php
return [
    // グローバルミドルウェア
    '' => [
        // ... その他のミドルウェアは省略
        app\middleware\Lang::class,
    ]
];
```


## プレースホルダの使用
メッセージに翻訳が必要な変数が含まれる場合があります。例えば
```php
trans('hello ' . $name);
```
このような場合はプレースホルダで対応します。

`resource/translations/zh_CN/messages.php` を以下のように変更します：
```php
return [
    'hello' => '你好 %name%!',
];
```
翻訳時に、2番目のパラメータでプレースホルダに対応する値を渡します：
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## 複数形の処理
一部の言語では、数量によって文の構造が異なります。例えば `There is %count% apple` は、`%count%` が1のときは正しいですが、1より大きいと誤りになります。

このような場合は**パイプ**（`|`）で複数形を列挙します。

言語ファイル `resource/translations/en/messages.php` に `apple_count` を以下のように追加します：
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

数値範囲を指定して、より複雑な複数形ルールを作成することもできます：
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## 言語ファイルの指定

言語ファイルのデフォルト名は `messages.php` です。他の名前の言語ファイルを作成することもできます。

言語ファイル `resource/translations/zh_CN/admin.php` を以下のように作成します：
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

`trans()` の第3引数で言語ファイルを指定します（`.php` 拡張子は省略）：
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## 詳細情報
[symfony/translation マニュアル](https://symfony.com/doc/current/translation.html) を参照してください
