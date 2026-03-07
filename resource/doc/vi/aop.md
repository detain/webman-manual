# AOP

> Cảm ơn tác giả của Hyperf đã đóng góp.

## Cài đặt

- Cài đặt aop-integration

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## Thêm cấu hình liên quan đến AOP

Cần thêm file cấu hình `config.php` vào thư mục `config`.

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
        // Thêm Aspect tương ứng tại đây
        app\aspect\DebugAspect::class,
    ]
];

```

## Cấu hình file khởi đầu start.php

> Đặt mã khởi tạo bên dưới cấu hình timezone. Các mã khác được bỏ qua bên dưới.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// Khởi tạo
ClassLoader::init();
```

## Kiểm thử

Đầu tiên, tạo class cần được chặn (intercept):

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

Sau đó thêm `DebugAspect` tương ứng:

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

Tiếp theo, chỉnh sửa controller `app/controller/IndexController.php`:

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

Sau đó cấu hình route:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

Cuối cùng, khởi động dịch vụ và chạy kiểm thử:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
