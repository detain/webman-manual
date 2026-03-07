# Redis 큐

Redis 기반 메시지 큐로, 메시지 지연 처리를 지원합니다.

## 설치
`composer require webman/redis-queue`

## 설정 파일
Redis 설정 파일은 `{메인-프로젝트}/config/plugin/webman/redis-queue/redis.php`에 자동 생성되며, 내용은 다음과 비슷합니다:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // 비밀번호, 선택
            'db' => 0,            // 데이터베이스
            'max_attempts'  => 5, // 소비 실패 시 재시도 횟수
            'retry_seconds' => 5, // 재시도 간격(초)
        ]
    ],
];
```

### 소비 실패 시 재시도
소비가 실패하면(예외 발생 시) 메시지는 지연 큐에 들어가 다음 재시도를 기다립니다. 재시도 횟수는 `max_attempts`로, 재시도 간격은 `retry_seconds`와 `max_attempts`로 함께 제어됩니다. 예: `max_attempts`가 5, `retry_seconds`가 10이면 1차 재시도 간격 `1*10`초, 2차 `2*10`초, 3차 `3*10`초… 5회까지. `max_attempts` 설정 횟수를 초과하면 메시지는 `{redis-queue}-failed` 키의 실패 큐에 들어갑니다.

## 메시지 전송 (동기)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // 큐 이름
        $queue = 'send-mail';
        // 데이터, 배열을 직접 전달 가능, 직렬화 불필요
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // 메시지 전송
        Redis::send($queue, $data);
        // 지연 메시지 전송, 60초 후 처리
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
전송 성공 시 `Redis::send()`는 true, 그렇지 않으면 false 또는 예외를 반환합니다.

> **팁**
> 지연 큐 소비 시각에 오차가 있을 수 있습니다. 예: 소비 속도가 생산 속도보다 느리면 큐가 쌓여 소비가 지연됩니다. 완화: 소비 프로세스를 더 많이 실행하세요.

## 메시지 전송 (비동기)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // 큐 이름
        $queue = 'send-mail';
        // 데이터, 배열을 직접 전달 가능, 직렬화 불필요
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // 메시지 전송
        Client::send($queue, $data);
        // 지연 메시지 전송, 60초 후 처리
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()`는 반환값이 없습니다. 비동기 푸시이며 Redis까지 100% 전달을 보장하지 않습니다.

> **팁**
> `Client::send()`의 원리는 로컬 메모리에 메모리 큐를 만들고 메시지를 비동기로 Redis에 동기화하는 것입니다(동기화는 빠름, 초당 약 1만 건). 프로세스가 재시작될 때 메모리 큐 데이터가 아직 동기화되지 않았으면 메시지가 손실될 수 있습니다. `Client::send()` 비동기 전송은 중요하지 않은 메시지에 적합합니다.

> **팁**
> `Client::send()`는 비동기라 Workerman 실행 환경에서만 사용할 수 있습니다. 명령줄 스크립트에서는 동기 인터페이스 `Redis::send()`를 사용하세요.

## 다른 프로젝트에서 메시지 전송
다른 프로젝트에서 메시지를 보내야 하는데 `webman\redis-queue`를 쓸 수 없을 때는 다음 함수를 참고해 큐에 메시지를 보낼 수 있습니다.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

여기서 `$redis` 매개변수는 Redis 인스턴스입니다. 예: redis 확장 사용:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## 소비
소비 프로세스 설정 파일은 `{메인-프로젝트}/config/plugin/webman/redis-queue/process.php`에 있습니다. consumer 디렉터리는 `{메인-프로젝트}/app/queue/redis/` 아래입니다.

`php webman redis-queue:consumer my-send-mail` 명령을 실행하면 `{메인-프로젝트}/app/queue/redis/MyMailSend.php` 파일이 생성됩니다.

> **팁**
> 이 명령에는 [콘솔](../plugin/console.md) 플러그인 설치가 필요합니다. 설치하지 않으려면 아래와 같은 파일을 수동으로 만들 수 있습니다:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // 소비할 큐 이름
    public $queue = 'send-mail';

    // 연결 이름, plugin/webman/redis-queue/redis.php의 연결과 대응
    public $connection = 'default';

    // 소비
    public function consume($data)
    {
        // 역직렬화 불필요
        var_export($data); // 출력 ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // 소비 실패 콜백
    /* 
    $package = [
        'id' => 1357277951, // 메시지 ID
        'time' => 1709170510, // 메시지 시각
        'delay' => 0, // 지연 시간
        'attempts' => 2, // 소비 횟수
        'queue' => 'send-mail', // 큐 이름
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // 메시지 내용
        'max_attempts' => 5, // 최대 재시도 횟수
        'error' => '오류 메시지' // 오류 메시지
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // 역직렬화 불필요
        var_export($package); 
    }
}
```

> **참고**
> 소비 중 예외나 Error가 throw되지 않으면 소비 성공, 아니면 실패로 재시도 큐에 들어갑니다. redis-queue에는 ack 메커니즘이 없으며 자동 ack로 보면 됩니다(예외·Error가 없을 때). 현재 메시지를 소비 실패로 표시하려면 수동으로 예외를 throw해 재시도 큐로 보내면 됩니다. 실질적으로 ack 메커니즘과 같습니다.

> **팁**
> consumer는 다중 서버·다중 프로세스를 지원하며, 같은 메시지는 두 번 소비되지 않습니다. 소비된 메시지는 큐에서 자동 삭제되며 수동 삭제는 필요 없습니다.

> **팁**
> 소비 프로세스는 여러 서로 다른 큐를 동시에 소비할 수 있습니다. 새 큐 추가 시 `process.php` 설정 변경은 필요 없습니다. 새 큐 consumer 추가 시 `app/queue/redis` 아래에 해당 `Consumer` 클래스를 추가하고, 클래스 속성 `$queue`로 소비할 큐 이름을 지정하면 됩니다.

> **팁**
> Windows 사용자는 webman 시작에 `php windows.php`를 실행해야 합니다. 그렇지 않으면 소비 프로세스가 시작되지 않습니다.

> **팁**
> onConsumeFailure 콜백은 소비가 실패할 때마다 호출됩니다. 여기서 실패 후 처리를 할 수 있습니다. (이 기능에는 `webman/redis-queue>=1.3.2` 및 `workerman/redis-queue>=1.2.1` 필요)

## 큐별로 다른 소비 프로세스 설정
기본적으로 모든 consumer가 같은 소비 프로세스를 공유합니다. 일부 큐의 소비를 분리하고 싶을 때(예: 느린 소비 업무를 한 프로세스 그룹, 빠른 소비 업무를 다른 그룹에) consumer를 두 디렉터리로 나눌 수 있습니다. 예: `app_path() . '/queue/redis/fast'`와 `app_path() . '/queue/redis/slow'` (consumer 클래스 네임스페이스도 이에 맞게 변경 필요). 설정 예:
```php
return [
    ...기타 설정 생략...
    
    'redis_consumer_fast'  => [ // 키는 사용자 정의, 형식 제한 없음, 여기선 redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer 클래스 디렉터리
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // 키는 사용자 정의, 형식 제한 없음, 여기선 redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // consumer 클래스 디렉터리
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

이렇게 하면 빠른 업무 consumer는 `queue/redis/fast` 디렉터리에, 느린 업무 consumer는 `queue/redis/slow`에 두어 큐별 소비 프로세스를 지정할 수 있습니다.

## 다중 Redis 설정
#### 설정
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // 비밀번호, 문자열 타입, 선택
            'db' => 0,            // 데이터베이스
            'max_attempts'  => 5, // 소비 실패 시 재시도 횟수
            'retry_seconds' => 5, // 재시도 간격(초)
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // 비밀번호, 문자열 타입, 선택
            'db' => 0,            // 데이터베이스
            'max_attempts'  => 5, // 소비 실패 시 재시도 횟수
            'retry_seconds' => 5, // 재시도 간격(초)
        ]
    ],
];
```

설정에 `other` 키의 Redis 설정이 추가되어 있습니다.

#### 다중 Redis로 메시지 전송

```php
// 키 `default`의 큐로 메시지 전송
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// 아래와 동일
Client::send($queue, $data);
Redis::send($queue, $data);

// 키 `other`의 큐로 메시지 전송
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### 다중 Redis에서 소비
설정에서 키 `other`의 큐에서 메시지 소비:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // 소비할 큐 이름
    public $queue = 'send-mail';

    // === 설정의 키 'other' 큐에서 소비하려면 여기에 'other' 설정 ===
    public $connection = 'other';

    // 소비
    public function consume($data)
    {
        // 역직렬화 불필요
        var_export($data);
    }
}
```

## 자주 묻는 질문

**`Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` 오류가 나는 이유**

이 오류는 비동기 전송 인터페이스 `Client::send()`에서만 발생합니다. 비동기 전송은 먼저 메시지를 로컬 메모리에 저장한 뒤 프로세스가 유휴일 때 Redis로 보냅니다. Redis 수신 속도가 메시지 생산 속도보다 느리거나, 프로세스가 다른 작업으로 바빠 메모리 메시지를 Redis에 동기화할 시간이 부족하면 메시지가 쌓입니다. 600초 이상 쌓이면 이 오류가 발생합니다.

해결: 메시지 전송에는 동기 전송 인터페이스 `Redis::send()`를 사용하세요.
