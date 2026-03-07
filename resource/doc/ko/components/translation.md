# 다국어

다국어 기능은 [symfony/translation](https://github.com/symfony/translation) 컴포넌트를 사용합니다.

## 설치
```
composer require symfony/translation
```

## 언어 패키지 생성
webman은 기본적으로 언어 패키지를 `resource/translations` 디렉터리에 둡니다(없으면 직접 생성). 디렉터리를 변경하려면 `config/translation.php`에서 설정하세요.
각 언어는 해당하는 하위 폴더에 대응하며, 언어 정의는 기본적으로 `messages.php`에 둡니다. 예시는 다음과 같습니다:
```
resource/
└── translations
    ├── en
    │   └── messages.php
    └── zh_CN
        └── messages.php
```

모든 언어 파일은 배열을 반환합니다. 예:
```php
// resource/translations/en/messages.php

return [
    'hello' => 'Hello webman',
];
```

## 설정

`config/translation.php`

```php
return [
    // 기본 언어
    'locale' => 'zh_CN',
    // 폴백 언어. 현재 언어에서 번역을 찾을 수 없으면 폴백 언어의 번역을 시도함
    'fallback_locale' => ['zh_CN', 'en'],
    // 언어 파일을 저장하는 폴더
    'path' => base_path() . '/resource/translations',
];
```

## 번역

번역에는 `trans()` 메서드를 사용합니다.

언어 파일 `resource/translations/zh_CN/messages.php`를 다음과 같이 생성하세요:
```php
return [
    'hello' => '你好 世界!',
];
```

파일 `app/controller/UserController.php`를 생성하세요:
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

`http://127.0.0.1:8787/user/get`에 접속하면 "你好 世界!"가 반환됩니다.

## 기본 언어 변경

언어 전환에는 `locale()` 메서드를 사용합니다.

언어 파일 `resource/translations/en/messages.php`를 다음과 같이 추가하세요:
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
        // 언어 전환
        locale('en');
        $hello = trans('hello'); // hello world!
        return response($hello);
    }
}
```
`http://127.0.0.1:8787/user/get`에 접속하면 "hello world!"가 반환됩니다.

`trans()` 함수의 4번째 매개변수로 임시 언어 전환도 가능합니다. 예를 들어 위 예제와 아래 예제는 동일합니다:
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function get(Request $request)
    {
        // 4번째 매개변수로 언어 전환
        $hello = trans('hello', [], null, 'en'); // hello world!
        return response($hello);
    }
}
```

## 각 요청에 명시적으로 언어 설정하기
translation은 싱글톤이므로 모든 요청이 이 인스턴스를 공유합니다. 어떤 요청이 `locale()`로 기본 언어를 설정하면 해당 프로세스의 이후 모든 요청에 영향을 줍니다. 따라서 각 요청에 명시적으로 언어를 설정해야 합니다. 예를 들어 다음 미들웨어를 사용하세요.

파일 `app/middleware/Lang.php`를 생성하세요(디렉터리가 없으면 직접 생성):
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

`config/middleware.php`에 전역 미들웨어를 다음과 같이 추가하세요:
```php
return [
    // 전역 미들웨어
    '' => [
        // ... 기타 미들웨어 생략
        app\middleware\Lang::class,
    ]
];
```


## 자리 표시자 사용
메시지에 번역해야 할 변수가 포함되는 경우가 있습니다. 예:
```php
trans('hello ' . $name);
```
이럴 때는 자리 표시자로 처리합니다.

`resource/translations/zh_CN/messages.php`를 다음과 같이 수정하세요:
```php
return [
    'hello' => '你好 %name%!',
];
```
번역 시 두 번째 매개변수로 자리 표시자에 해당하는 값을 전달합니다:
```php
trans('hello', ['%name%' => 'webman']); // 你好 webman!
```

## 복수 처리
일부 언어는 수량에 따라 문장 구조가 다릅니다. 예를 들어 `There is %count% apple`은 `%count%`가 1일 때 맞지만 1보다 크면 틀립니다.

이럴 때는 **파이프**(`|`)로 복수형을 나열합니다.

언어 파일 `resource/translations/en/messages.php`에 `apple_count`를 다음과 같이 추가하세요:
```php
return [
    // ...
    'apple_count' => 'There is one apple|There are %count% apples',
];
```

```php
trans('apple_count', ['%count%' => 10]); // There are 10 apples
```

숫자 범위를 지정해 더 복잡한 복수 규칙을 만들 수도 있습니다:
```php
return [
    // ...
    'apple_count' => '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf[ There are many apples'
];
```

```php
trans('apple_count', ['%count%' => 20]); // There are many apples
```

## 언어 파일 지정

언어 파일 기본 이름은 `messages.php`이며, 다른 이름의 언어 파일도 만들 수 있습니다.

언어 파일 `resource/translations/zh_CN/admin.php`를 다음과 같이 생성하세요:
```php
return [
    'hello_admin' => '你好 管理员!',
];
```

`trans()`의 세 번째 매개변수로 언어 파일을 지정합니다(확장자 `.php` 생략).
```php
trans('hello', [], 'admin', 'zh_CN'); // 你好 管理员!
```

## 더 알아보기
[symfony/translation 매뉴얼](https://symfony.com/doc/current/translation.html)을 참조하세요
