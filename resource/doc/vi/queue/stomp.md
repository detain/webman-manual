# Hàng đợi Stomp

Stomp là giao thức tin nhắn định hướng văn bản đơn giản (streaming), cung cấp định dạng kết nối tương tác cho phép máy khách Stomp tương tác với bất kỳ broker tin nhắn Stomp nào. [workerman/stomp](https://github.com/walkor/stomp) triển khai máy khách Stomp, chủ yếu dùng cho hàng đợi tin nhắn như RabbitMQ, Apollo, ActiveMQ, v.v.

## Cài đặt
`composer require webman/stomp`

## Cấu hình
Tệp cấu hình nằm ở `config/plugin/webman/stomp`

## Gửi tin nhắn
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // Hàng đợi
        $queue = 'examples';
        // Dữ liệu (khi truyền mảng cần tự serialize, ví dụ dùng json_encode, serialize, v.v.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Thực hiện gửi
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Để tương thích với dự án khác, thành phần Stomp không cung cấp tự động serialize và unserialize. Nếu gửi dữ liệu mảng, cần tự serialize khi gửi và tự unserialize khi tiêu thụ.

## Tiêu thụ tin nhắn
Tạo mới `app/queue/stomp/MyMailSend.php` (tên lớp tùy ý, theo chuẩn PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Tên hàng đợi
    public $queue = 'examples';

    // Tên kết nối, tương ứng với kết nối trong stomp.php
    public $connection = 'default';

    // Khi giá trị là client, cần gọi $ack_resolver->ack() để thông báo cho máy chủ đã tiêu thụ thành công
    // Khi giá trị là auto, không cần gọi $ack_resolver->ack()
    public $ack = 'auto';

    // Tiêu thụ
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Nếu dữ liệu là mảng, cần tự unserialize
        var_export(json_decode($data, true)); // In ra ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Thông báo cho máy chủ đã tiêu thụ thành công
        $ack_resolver->ack(); // Khi ack là auto có thể bỏ qua lệnh gọi này
    }
}
```

# Bật giao thức Stomp trong RabbitMQ
RabbitMQ mặc định không bật giao thức Stomp, cần chạy lệnh sau để bật:
```
rabbitmq-plugins enable rabbitmq_stomp
```
Sau khi bật, cổng mặc định của Stomp là 61613.
