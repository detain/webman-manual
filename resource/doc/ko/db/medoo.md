# Medoo 데이터베이스

[webman/medoo](https://github.com/webman-php/medoo)는 [Medoo](https://medoo.in/)에 연결 풀 기능을 추가하며, 코루틴 환경과 비코루틴 환경 모두에서 동작합니다. 사용법은 Medoo와 같습니다.

## 설치
`composer require webman/medoo`

## Medoo 데이터베이스 설정
설정 파일 위치: `config/plugin/webman/medoo/database.php`

## Medoo 데이터베이스 사용법
```php
<?php
namespace app\controller;

use support\Request;
use support\Medoo;

class Index
{
    public function index(Request $request)
    {
        $user = Medoo::get('user', '*', ['uid' => 1]);
        return json($user);
    }
}
```

> **팁**
> `Medoo::get('user', '*', ['uid' => 1]);`
> 는
> `Medoo::instance('default')->get('user', '*', ['uid' => 1]);`
> 와 같습니다.

## Medoo 다중 데이터베이스 설정

**설정**
`config/plugin/webman/medoo/database.php`에 새 구성을 추가합니다. 키는 임의로 지정할 수 있으며, 여기서는 `other`를 사용합니다.

```php
<?php
return [
    'default' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [ // 연결 풀 설정
            'max_connections' => 5, // 최대 연결 수
            'min_connections' => 1, // 최소 연결 수
            'wait_timeout' => 60,   // 풀에서 연결 획득 대기 최대 시간, 초과 시 예외 발생
            'idle_timeout' => 3,    // 풀 내 연결 최대 유휴 시간, 초과 시 회수되어 min_connections까지 감소
            'heartbeat_interval' => 50, // 풀 heartbeat 간격(초), 60초 미만 권장
        ]
    ],
    // 여기에 other 설정 추가
    'other' => [
        'type' => 'mysql',
        'host' => 'localhost',
        'database' => 'database',
        'username' => 'username',
        'password' => 'password',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_general_ci',
        'port' => 3306,
        'prefix' => '',
        'logging' => false,
        'error' => PDO::ERRMODE_EXCEPTION,
        'option' => [
            PDO::ATTR_CASE => PDO::CASE_NATURAL
        ],
        'command' => [
            'SET SQL_MODE=ANSI_QUOTES'
        ],
        'pool' => [
            'max_connections' => 5,
            'min_connections' => 1,
            'wait_timeout' => 60,
            'idle_timeout' => 3,
            'heartbeat_interval' => 50,
        ],
    ],
];
```

## Medoo 데이터베이스 사용법
```php
$user = Medoo::instance('other')->get('user', '*', ['uid' => 1]);
```

[Medoo 공식 문서](https://medoo.in/api/select)를 참조하십시오.
