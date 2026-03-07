# Stomp 대기열

Stomp은 간단한(스트리밍) 텍스트 지향 메시징 프로토콜로, 상호 운용 가능한 연결 형식을 제공하여 STOMP 클라이언트가 임의의 STOMP 메시지 브로커(Broker)와 통신할 수 있게 합니다. [workerman/stomp](https://github.com/walkor/stomp)은 Stomp 클라이언트를 구현하며, 주로 RabbitMQ, Apollo, ActiveMQ 등의 메시지 대기열 시나리오에 사용됩니다.

## 설치
`composer require webman/stomp`

## 설정
설정 파일은 `config/plugin/webman/stomp`에 있습니다.

## 메시지 발송
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // 대기열
        $queue = 'examples';
        // 데이터 (배열 전달 시 json_encode, serialize 등을 사용해 직접 직렬화해야 함)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // 발송 실행
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> 다른 프로젝트와의 호환성을 위해 Stomp 컴포넌트는 자동 직렬화·역직렬화 기능을 제공하지 않습니다. 배열 데이터를 발송할 경우 직접 직렬화하고, 소비 시 직접 역직렬화해야 합니다.

## 메시지 소비
`app/queue/stomp/MyMailSend.php`를 새로 만듭니다 (클래스명은 PSR-4 규칙을 따르면 임의로 지정 가능).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // 대기열 이름
    public $queue = 'examples';

    // 연결 이름, stomp.php의 연결과 대응
    public $connection = 'default';

    // 값이 client일 때는 $ack_resolver->ack()를 호출해 서버에 소비 완료를 알려야 함
    // 값이 auto일 때는 $ack_resolver->ack()를 호출할 필요 없음
    public $ack = 'auto';

    // 소비
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // 데이터가 배열이면 직접 역직렬화해야 함
        var_export(json_decode($data, true)); // ['to' => 'tom@gmail.com', 'content' => 'hello'] 출력
        // 서버에 소비 완료 알림
        $ack_resolver->ack(); // ack가 auto일 때는 이 호출 생략 가능
    }
}
```

# RabbitMQ에서 Stomp 프로토콜 활성화
RabbitMQ는 기본적으로 Stomp 프로토콜을 활성화하지 않습니다. 다음 명령을 실행하여 활성화해야 합니다.
```
rabbitmq-plugins enable rabbitmq_stomp
```
활성화 후 Stomp 기본 포트는 61613입니다.
