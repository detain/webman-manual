# AOP

> Hyperf の作者による貢献に感謝します。

## インストール

- aop-integration をインストール

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP 関連の設定を追加

`config` ディレクトリに `config.php` 設定ファイルを追加する必要があります。

```php
<?php

use Hyperf\Di\Annotation\AspectCollector;

return [
    'annotations' => [
        'scan' => [
            'paths' => [
                BASE_PATH . '/app',
            ],
            'ignore_annotations' => [
                'mixin',
            ],
            'class_map' => [
            ],
            'collectors' => [
                AspectCollector::class
            ],
        ],
    ],
    'aspects' => [
        // ここに対応する Aspect を記述
        app\aspect\DebugAspect::class,
    ]
];

```

## エントリーファイル start.php の設定

> 初期化処理を timezone 設定の下に配置します。以下、他のコードは省略します。

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// 初期化
ClassLoader::init();
```

## テスト

まず、差し込む対象のクラスを作成します。

```php
<?php
namespace app\service;

class UserService
{
    public function first(): array
    {
        return ['id' => 1];
    }
}
```

次に、対応する `DebugAspect` を追加します。

```php
<?php
namespace app\aspect;

use app\service\UserService;
use Hyperf\Di\Aop\AbstractAspect;
use Hyperf\Di\Aop\ProceedingJoinPoint;

class DebugAspect extends AbstractAspect
{
    public $classes = [
        UserService::class . '::first',
    ];

    public function process(ProceedingJoinPoint $proceedingJoinPoint)
    {
        var_dump(11);
        return $proceedingJoinPoint->process();
    }
}
```

続いて、コントローラー `app/controller/IndexController.php` を編集します。

```php
<?php
namespace app\controller;

use app\service\UserService;
use support\Request;

class IndexController
{
    public function json(Request $request)
    {
        return json(['code' => 0, 'msg' => 'ok', 'data' => (new UserService())->first()]);
    }
}
```

ルートを設定します。

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

最後に、サービスを起動してテストを実行します。

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
