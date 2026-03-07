# AOP

> Hyperf 개발자의 기여에 감사드립니다.

## 설치

- aop-integration 설치

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP 관련 설정 추가

`config` 디렉터리에 `config.php` 설정 파일을 추가해야 합니다.

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
        // 여기에 해당하는 Aspect를 추가합니다
        app\aspect\DebugAspect::class,
    ]
];

```

## 진입 파일 start.php 설정

> 초기화 코드를 timezone 설정 아래에 배치합니다. 나머지 코드는 아래 생략합니다.

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// 초기화
ClassLoader::init();
```

## 테스트

먼저 가로채기 대상 클래스를 작성합니다:

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

그 다음 해당하는 `DebugAspect`를 추가합니다:

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

다음으로 컨트롤러 `app/controller/IndexController.php`를 편집합니다:

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

이후 라우트를 설정합니다:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

마지막으로 서비스를 시작하고 테스트합니다:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
