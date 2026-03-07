# AOP

> Hyperf लेखक के योगदान के लिए धन्यवाद।

## स्थापना

- aop-integration इंस्टॉल करें

```shell
composer require "hyperf/aop-integration: ^1.1"
```

## AOP संबंधित कॉन्फ़िगरेशन जोड़ें

config फ़ोल्डर में config.php कॉन्फ़िगरेशन फ़ाइल जोड़नी होगी।

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
        // यहाँ संबंधित Aspect जोड़ें
        app\aspect\DebugAspect::class,
    ]
];

```

## एंट्री फ़ाइल start.php कॉन्फ़िगर करें

> timezone सेटिंग के नीचे इनिशियलाइज़ेशन कोड रखें। नीचे अन्य कोड छोड़ा गया है।

```
use Hyperf\AopIntegration\ClassLoader;

if ($timezone = config('app.default_timezone')) {
    date_default_timezone_set($timezone);
}

// इनिशियलाइज़ेशन
ClassLoader::init();
```

## टेस्टिंग

सबसे पहले इंटरसेप्ट की जाने वाली क्लास बनाएँ:

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

फिर संबंधित DebugAspect जोड़ें:

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

इसके बाद कंट्रोलर app/controller/IndexController.php संपादित करें:

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

फिर रूट कॉन्फ़िगर करें:

```php
<?php
use Webman\Route;

Route::any('/json', [app\controller\IndexController::class, 'json']);
```

अंत में सर्विस शुरू करें और टेस्ट चलाएँ:

```shell
php start.php start
curl  http://127.0.0.1:8787/json
```
