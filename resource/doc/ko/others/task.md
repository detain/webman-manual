# 느린 비즈니스 처리

때로는 느린 비즈니스를 처리해야 합니다. 느린 비즈니스가 webman의 다른 요청 처리에 영향을 주지 않도록 상황에 따라 다양한 처리 방식을 사용할 수 있습니다.

## 방안 1: 메시지 큐 사용
[Redis 큐](../queue/redis.md) [Stomp 큐](../queue/stomp.md) 참조

#### 장점
갑작스러운 대량 비즈니스 처리 요청에 대응할 수 있습니다.

#### 단점
클라이언트에 직접 결과를 반환할 수 없습니다. 결과를 푸시해야 하는 경우 [webman/push](https://www.workerman.net/plugin/2)로 처리 결과를 푸시하는 등 다른 서비스와 연동해야 합니다.

## 방안 2: 새 HTTP 포트 추가

새 HTTP 포트를 추가하여 느린 요청을 처리합니다. 이러한 느린 요청은 이 포트로 접속하여 특정 프로세스 그룹에서 처리되며, 처리 후 결과가 클라이언트에 직접 반환됩니다.

#### 장점
데이터를 직접 클라이언트에 반환할 수 있습니다.

#### 단점
갑작스러운 대량 요청에 대응할 수 없습니다.

#### 구현 단계
`config/process.php`에 다음과 같이 설정을 추가합니다.
```php
return [
    // ... 기타 설정은 생략 ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // 프로세스 수
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // 요청 클래스 설정
            'logger' => \support\Log::channel('default'), // 로거 인스턴스
            'appPath' => app_path(), // app 디렉토리 위치
            'publicPath' => public_path() // public 디렉토리 위치
        ]
    ]
];
```

이렇게 하면 느린 인터페이스는 `http://127.0.0.1:8686/` 프로세스 그룹을 통해 처리되며 다른 프로세스의 비즈니스 처리에 영향을 주지 않습니다.

프론트엔드가 포트 차이를 인지하지 못하도록 nginx에 8686 포트로의 프록시를 추가할 수 있습니다. 느린 인터페이스 요청 경로가 모두 `/task`로 시작한다고 가정하면 nginx 설정은 다음과 비슷합니다:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# 새 8686 upstream 추가
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # /task로 시작하는 요청은 8686 포트로, 필요에 따라 /task를 원하는 접두사로 변경하세요
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # 기타 요청은 기존 8787 포트로
  location / {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      if (!-f $request_filename){
          proxy_pass http://webman;
      }
  }
}
```

이렇게 하면 클라이언트가 `도메인.com/task/xxx`에 접속할 때 별도의 8686 포트에서 처리되어 8787 포트의 요청 처리에 영향을 주지 않습니다.

## 방안 3: HTTP Chunked를 이용한 비동기 분할 데이터 전송

#### 장점
데이터를 직접 클라이언트에 반환할 수 있습니다.

**workerman/http-client 설치**

```
composer require workerman/http-client
```

**app/controller/IndexController.php**
```php
<?php
namespace app\controller;

use support\Request;
use support\Response;
use Workerman\Protocols\Http\Chunk;

class IndexController
{
    public function index(Request $request)
    {
        $connection = $request->connection;
        $http = new \Workerman\Http\Client();
        $http->get('https://example.com/', function ($response) use ($connection) {
            $connection->send(new Chunk($response->getBody()));
            $connection->send(new Chunk('')); // 빈 chunk 전송으로 응답 종료 표시
        });
        // 먼저 HTTP 헤더를 전송하고, 이후 데이터는 비동기로 전송
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **참고**
> 본 예제는 `workerman/http-client` 클라이언트로 HTTP 결과를 비동기 조회하여 데이터를 반환합니다. [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html) 등 다른 비동기 클라이언트도 사용할 수 있습니다.
