# 데이터베이스 빠른 시작 (Laravel 데이터베이스 컴포넌트 기반)

[webman/database](https://github.com/webman-php/database)는 [illuminate/database](https://github.com/illuminate/database)를 기반으로 개발되었으며, 연결 풀 기능을 추가하여 코루틴 및 비코루틴 환경을 모두 지원합니다. 사용법은 Laravel과 동일합니다.

[다른 데이터베이스 컴포넌트 사용](others.md) 섹션을 참고하여 ThinkPHP나 기타 데이터베이스를 사용할 수도 있습니다.

## 데이터베이스 설치

`composer require -W webman/database illuminate/pagination illuminate/events symfony/var-dumper`

설치 후에는 restart로 재시작해야 합니다(reload는 적용되지 않습니다).

> **안내**
> webman/database는 Laravel의 `illuminate/database`에 의존하므로, 설치 시 `illuminate/database`의 의존성 패키지가 자동으로 설치됩니다.

> **참고**
> 페이지네이션, 데이터베이스 이벤트, SQL 기록이 필요 없다면 다음만 실행하면 됩니다.
> `composer require -W webman/database`

## 데이터베이스 설정
`config/database.php`
```php

return [
    // 기본 데이터베이스
    'default' => 'mysql',

    // 각종 데이터베이스 연결 설정
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'test',
            'username'    => 'root',
            'password'    => '',
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
            'options' => [
                PDO::ATTR_EMULATE_PREPARES => false, // swoole 또는 swow를 런타임으로 사용할 때 필수
            ],
            'pool' => [ // 연결 풀 설정
                'max_connections' => 5, // 최대 연결 수
                'min_connections' => 1, // 최소 연결 수
                'wait_timeout' => 3,    // 연결 풀에서 연결을 가져올 때 대기하는 최대 시간. 초과 시 예외 발생. 코루틴 환경에서만 유효
                'idle_timeout' => 60,   // 풀 내 연결의 최대 유휴 시간. 초과 시 연결을 닫고 회수하여 min_connections까지 유지
                'heartbeat_interval' => 50, // 연결 풀 심장박동 간격(초). 60초 미만 권장
            ],
        ],
    ],
];
```

`pool` 설정을 제외한 나머지는 Laravel과 동일합니다.

## 연결 풀에 대해
* 각 프로세스는 자체 연결 풀을 가지며, 프로세스 간에 풀을 공유하지 않습니다.
* 코루틴을 사용하지 않을 때는 요청이 프로세스 내에서 순차 실행되어 동시성이 없으므로, 풀에는 최대 1개의 연결만 존재합니다.
* 코루틴 사용 시에는 요청이 프로세스 내에서 동시 실행되며, 풀이 필요에 따라 연결 수를 동적으로 조정합니다. `max_connections`를 넘지 않고 `min_connections` 아래로 내려가지 않습니다.
* 풀 연결 수가 최대 `max_connections`이므로, 데이터베이스를 사용하는 코루틴 수가 이를 초과하면 일부가 대기열에 최대 `wait_timeout`초까지 대기하며, 초과 시 예외가 발생합니다.
* 유휴 상태일 때(코루틴·비코루틴 모두) 연결은 `idle_timeout` 후 회수되며, 연결 수가 `min_connections`에 도달할 때까지 이어집니다(`min_connections`는 0 가능).


## 데이터베이스 사용 예제
```php
<?php
namespace app\controller;

use support\Request;
use support\Db;

class UserController
{
    public function db(Request $request)
    {
        $default_uid = 29;
        $uid = $request->get('uid', $default_uid);
        $name = Db::table('users')->where('uid', $uid)->value('username');
        return response("hello $name");
    }
}
```

사용법은 Laravel과 같으며, `Db::table()` 메서드로 데이터베이스를 조작합니다.
