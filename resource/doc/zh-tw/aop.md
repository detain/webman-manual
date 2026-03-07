# AOP

> 感謝 Hyperf 作者的貢獻。

## 安裝

- 安裝 aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## 增加 AOP 相關配置

我們需要在 `config` 目錄下新增 `config.php` 配置檔案。

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
        // 在此寫入對應的 Aspect
        app\aspect\DebugAspect::class,
    ]
];

```

## 配置入口檔案 start.php

> 將初始化方法放在 timezone 設定下方，以下省略其他程式碼。

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// 初始化
ClassLoader::init();
```

## 測試

首先建立待切入的類別：

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

接著新增對應的 `DebugAspect`：

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

然後編輯控制器 `app/controller/IndexController.php`：

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

接著設定路由：

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

最後啟動服務並進行測試。

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
