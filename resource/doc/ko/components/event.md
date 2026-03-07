# 이벤트 처리
`webman/event`는 코드를 수정하지 않고 비즈니스 로직을 실행할 수 있는 세밀한 이벤트 메커니즘을 제공하여 모듈 간 결합을 완화합니다. 전형적인 시나리오: 새 사용자가 등록에 성공하면 `user.register`와 같은 사용자 정의 이벤트를 발행하기만 하면, 각 모듈이 이벤트를 수신하고 해당 비즈니스 로직을 실행할 수 있습니다.

## 설치
`composer require webman/event`

## 이벤트 구독
이벤트 구독은 `config/event.php` 파일에서 통일하여 설정합니다.

```php
<?php
return [
    'user.register' => [
        [app\event\User::class, 'register'],
        // ...다른 이벤트 처리 함수...
    ],
    'user.logout' => [
        [app\event\User::class, 'logout'],
        // ...다른 이벤트 처리 함수...
    ]
];
```

**설명:**
- `user.register`, `user.logout` 등은 이벤트 이름(문자열)입니다. 소문자로 점(`.`)으로 구분하는 것을 권장합니다.
- 하나의 이벤트에 여러 처리 함수를 연결할 수 있으며, 설정 순서대로 호출됩니다.

## 이벤트 처리 함수
처리 함수는 클래스 메서드, 함수, 클로저 등 어떤 것이어도 됩니다.

예: `app/event/User.php` 생성(디렉터리가 없으면 직접 생성)

```php
<?php
namespace app\event;
class User
{
    function register($user)
    {
        var_export($user);
    }
 
    function logout($user)
    {
        var_export($user);
    }
}
```

## 이벤트 발행
`Event::dispatch($event_name, $data);` 또는 `Event::emit($event_name, $data);`로 이벤트를 발행합니다. 예:

```php
<?php
namespace app\controller;
use support\Request;
use Webman\Event\Event;
class User
{
    public function register(Request $request)
    {
        $user = [
            'name' => 'webman',
            'age' => 2
        ];
        Event::dispatch('user.register', $user);
    }
}
```

발행 함수는 `Event::dispatch($event_name, $data);`와 `Event::emit($event_name, $data);` 두 가지이며, 매개변수는 동일합니다. 차이는 `emit`은 내부에서 예외를 자동으로 처리하여, 한 이벤트에 여러 처리 함수가 있어도 하나에서 예외가 발생해도 나머지는 계속 실행됩니다. 반면 `dispatch`는 예외를 처리하지 않아, 하나의 처리 함수에서 예외가 발생하면 다음 함수는 실행되지 않고 예외가 상위로 전파됩니다.

> **팁**
> 매개변수 `$data`는 배열, 클래스 인스턴스, 문자열 등 임의의 데이터가 가능합니다.

## 와일드카드 이벤트 리스너
와일드카드 등록을 사용하면 같은 리스너로 여러 이벤트를 처리할 수 있습니다. 예: `config/event.php`에서

```php
<?php
return [
    'user.*' => [
        [app\event\User::class, 'deal']
    ],
];
```

처리 함수의 두 번째 매개변수 `$event_data`로 실제 이벤트 이름을 얻을 수 있습니다:

```php
<?php
namespace app\event;
class User
{
    function deal($user, $event_name)
    {
        echo $event_name; // 실제 이벤트 이름 (user.register, user.logout 등)
        var_export($user);
    }
}
```

## 이벤트 브로드캐스트 중지
처리 함수에서 `false`를 반환하면 해당 이벤트의 브로드캐스트가 중지됩니다.

## 클로저로 이벤트 처리
처리 함수는 클래스 메서드 또는 클로저 모두 가능합니다. 예:

```php
<?php
return [
    'user.login' => [
        function($user){
            var_dump($user);
        }
    ]
];
```

## 이벤트 및 리스너 보기
`php webman event:list` 명령으로 프로젝트에 설정된 모든 이벤트와 리스너를 볼 수 있습니다.

## 지원 범위
메인 프로젝트 외에 [기본 플러그인](../plugin/base.md)과 [앱 플러그인](../app/app.md)도 `event.php` 설정을 지원합니다.
**기본 플러그인 설정 파일:** `config/plugin/제공자/플러그인명/event.php`
**앱 플러그인 설정 파일:** `plugin/플러그인명/config/event.php`

## 유의사항
이벤트 처리는 비동기가 아니며, 느린 비즈니스에는 적합하지 않습니다. 느린 로직은 [webman/redis-queue](https://www.workerman.net/plugin/12) 같은 메시지 큐로 처리하세요.
