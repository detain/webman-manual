# Redis

[webman/redis](https://github.com/webman-php/redis)는 [illuminate/redis](https://github.com/illuminate/redis)를 기반으로 연결 풀 기능을 추가한 것으로, 코루틴 및 비코루틴 환경을 모두 지원하며 Laravel과 동일하게 사용합니다.

`illuminate/redis`를 사용하기 전에 `php-cli`에 Redis 확장을 설치해야 합니다.

## 설치

```php
composer require -W webman/redis illuminate/events
```

설치 후에는 restart가 필요합니다 (reload는 적용되지 않음).

## 구성

Redis 구성 파일은 `config/redis.php`에 있습니다.

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
        'pool' => [ // 연결 풀 설정
            'max_connections' => 10,     // 풀 최대 연결 수
            'min_connections' => 1,      // 풀 최소 연결 수
            'wait_timeout' => 3,         // 연결 획득 최대 대기 시간(초)
            'idle_timeout' => 50,        // 유휴 타임아웃, 초과 후 min_connections까지 연결 해제
            'heartbeat_interval' => 50,  // heartbeat 간격 (60초 이하여야 함)
        ],
    ]
];
```

## 연결 풀에 대해

* 각 프로세스는 자체 연결 풀을 가지며, 프로세스 간에는 공유되지 않습니다.
* 코루틴을 사용하지 않으면 작업이 순차 실행되어 최대 1개의 연결만 사용됩니다.
* 코루틴을 사용하면 작업이 동시 실행되며, 풀은 `min_connections`~`max_connections` 범위에서 동적으로 조정됩니다.
* Redis를 사용하는 코루틴 수가 `max_connections`를 초과하면 대기열에서 최대 `wait_timeout`초 대기하며, 초과 시 예외가 발생합니다.
* 유휴 시(코루틴 여부와 무관) 연결은 `idle_timeout` 후 해제되며, `min_connections`(`min_connections`는 0 가능)에 도달할 때까지 줄어듭니다.

## 예제

```php
<?php
namespace app\controller;

use support\Request;
use support\Redis;

class UserController
{
    public function db(Request $request)
    {
        $key = 'test_key';
        Redis::set($key, rand());
        return response(Redis::get($key));
    }
}
```

## Redis 인터페이스

```php
Redis::append($key, $value)
Redis::bitCount($key)
Redis::decr($key, $value)
Redis::decrBy($key, $value)
Redis::get($key)
Redis::getBit($key, $offset)
Redis::getRange($key, $start, $end)
Redis::getSet($key, $value)
Redis::incr($key, $value)
Redis::incrBy($key, $value)
Redis::incrByFloat($key, $value)
Redis::mGet(array $keys)
Redis::getMultiple(array $keys)
Redis::mSet($pairs)
Redis::mSetNx($pairs)
Redis::set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
Redis::setBit($key, $offset, $value)
Redis::setEx($key, $ttl, $value)
Redis::pSetEx($key, $ttl, $value)
Redis::setNx($key, $value)
Redis::setRange($key, $offset, $value)
Redis::strLen($key)
Redis::del(...$keys)
Redis::exists(...$keys)
Redis::expire($key, $ttl)
Redis::expireAt($key, $timestamp)
Redis::select($dbIndex)
```

다음과 동일합니다:

```php
$redis = Redis::connection('default');
$redis->append($key, $value)
$redis->bitCount($key)
$redis->decr($key, $value)
$redis->decrBy($key, $value)
$redis->get($key)
$redis->getBit($key, $offset)
...
```

> **주의**
> `Redis::select($db)` 인터페이스는 신중히 사용하세요. webman은 상주형 메모리 프레임워크이므로, 한 요청에서 `Redis::select($db)`로 데이터베이스를 전환하면 이후 요청에 영향을 줍니다. 여러 데이터베이스를 사용할 경우 각 `$db`를 서로 다른 Redis 연결로 설정하는 것이 좋습니다.

## 여러 Redis 연결 사용

예: `config/redis.php` 구성 파일

```php
return [
    'default' => [
        'host'     => '127.0.0.1',
        'username' => null,
        'password' => null,
        'port'     => 6379,
        'database' => 0,
    ],

    'cache' => [
        'host'     => '127.0.0.1',
        'password' => null,
        'port'     => 6379,
        'database' => 1,
    ],

]
```

기본값은 `default`에 설정된 연결입니다. `Redis::connection()`으로 사용할 Redis 연결을 선택할 수 있습니다.

```php
$redis = Redis::connection('cache');
$redis->get('test_key');
```

## 클러스터 구성

애플리케이션이 Redis 클러스터를 사용하는 경우, 구성 파일에서 `clusters` 키로 정의합니다:

```php
return [
    'clusters' => [
        'default' => [
            [
                'host'     => 'localhost',
                'username' => null,
                'password' => null,
                'port'     => 6379,
                'database' => 0,
            ],
        ],
    ],

];
```

기본적으로 클러스터는 노드에서 클라이언트 샤딩을 수행하여 노드 풀과 대량의 메모리를 확보할 수 있습니다. 클라이언트 샤딩은 장애를 처리하지 않으므로, 다른 주 데이터베이스에서 캐시 데이터를 가져오는 용도에 적합합니다. Redis 네이티브 클러스터를 사용하려면 구성의 `options`에서 다음과 같이 지정합니다:

```php
return[
    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],
];
```

## 파이프라인 명령

한 번에 여러 명령을 보내야 할 때는 파이프라인을 사용하는 것이 좋습니다. `pipeline` 메서드는 Redis 인스턴스의 클로저를 받으며, 전달된 명령은 한 번의 작업으로 실행됩니다:

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
