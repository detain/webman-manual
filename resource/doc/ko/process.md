# 사용자 정의 프로세스

webman에서는 workerman과 마찬가지로 사용자 정의 리스너 또는 프로세스를 만들 수 있습니다.

> **주의**
> Windows 사용자는 사용자 정의 프로세스를 시작하려면 `php windows.php`를 사용하여 webman을 시작해야 합니다.

## 사용자 정의 HTTP 서비스
때로는 webman HTTP 서비스의 코어 코드를 변경해야 할 특별한 요구사항이 있을 수 있습니다. 이 경우 사용자 정의 프로세스를 통해 구현할 수 있습니다.

예를 들어 `app\Server.php`를 새로 만듭니다.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // 여기에서 Webman\App의 메서드를 재정의합니다.
}
```

`config/process.php`에 다음 구성을 추가합니다.

```php
use Workerman\Worker;

return [
    // ... 다른 구성들은 생략 ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // 프로세스 수
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // 요청 클래스 설정
            'logger' => \support\Log::channel('default'), // 로그 인스턴스
            'appPath' => app_path(), // 앱 디렉터리 위치
            'publicPath' => public_path() // public 디렉터리 위치
        ]
    ]
];
```

> **팁**
> webman의 기본 HTTP 프로세스를 종료하려면 config/server.php에서 `listen=>''`로 설정하면 됩니다.

## 사용자 정의 웹소켓 리스너 예제

`app/Pusher.php`를 만듭니다.

```php
<?php
namespace app;

use Workerman\Connection\TcpConnection;

class Pusher
{
    public function onConnect(TcpConnection $connection)
    {
        echo "onConnect\n";
    }

    public function onWebSocketConnect(TcpConnection $connection, $http_buffer)
    {
        echo "onWebSocketConnect\n";
    }

    public function onMessage(TcpConnection $connection, $data)
    {
        $connection->send($data);
    }

    public function onClose(TcpConnection $connection)
    {
        echo "onClose\n";
    }
}
```

> 주의: 모든 onXXX 메서드는 public이어야 합니다.

`config/process.php`에 다음 구성을 추가합니다.

```php
return [
    // ... 다른 프로세스 구성은 생략 ...
    
    // websocket_test는 프로세스 이름입니다.
    'websocket_test' => [
        // 여기에서 프로세스 클래스를 지정합니다. 위에서 정의한 Pusher 클래스입니다.
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## 사용자 정의 비리스닝 프로세스 예제

`app/TaskTest.php`를 만듭니다.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // 10초마다 새로 등록된 사용자가 있는지 데이터베이스를 확인합니다.
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

`config/process.php`에 다음 구성을 추가합니다.

```php
return [
    // ... 다른 프로세스 구성은 생략 ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> 주의: listen을 생략하면 어떤 포트도 리스닝하지 않으며, count를 생략하면 프로세스 수가 기본적으로 1로 설정됩니다.

## 구성 파일 설명

프로세스의 완전한 구성은 다음과 같습니다.

```php
return [
    // ... 
    
    // websocket_test는 프로세스 이름입니다.
    'websocket_test' => [
        // 여기에서 프로세스 클래스를 지정합니다.
        'handler' => app\Pusher::class,
        // 리스닝 프로토콜, IP 및 포트 (선택 사항)
        'listen'  => 'websocket://0.0.0.0:8888',
        // 프로세스 수 (선택 사항, 기본값 1)
        'count'   => 2,
        // 프로세스 실행 사용자 (선택 사항, 기본값 현재 사용자)
        'user'    => '',
        // 프로세스 실행 사용자 그룹 (선택 사항, 기본값 현재 사용자 그룹)
        'group'   => '',
        // 현재 프로세스가 reload를 지원하는지 여부 (선택 사항, 기본값 true)
        'reloadable' => true,
        // reusePort 활성화 여부
        'reusePort'  => true,
        // transport (선택 사항, ssl을 사용해야 할 경우 ssl로 설정, 기본값 tcp)
        'transport'  => 'tcp',
        // context (선택 사항, transport가 ssl인 경우 인증서 경로를 전달해야 함)
        'context'    => [], 
        // 프로세스 클래스 생성자 매개변수 (선택 사항)
        'constructor' => [],
        // 이 프로세스가 활성화되어 있는지 여부
        'enable' => true
    ],
];
```

## 요약

webman의 사용자 정의 프로세스는 사실상 workerman의 간단한 래핑으로, 구성과 비즈니스를 분리하고 workerman의 `onXXX` 콜백을 클래스 메서드로 구현했습니다. 다른 용도로의 사용은 workerman과 완전히 동일합니다.
