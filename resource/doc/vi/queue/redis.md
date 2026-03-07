# Hàng đợi Redis

Hàng đợi tin nhắn dựa trên Redis, hỗ trợ xử lý tin nhắn trễ.

## Cài đặt
`composer require webman/redis-queue`

## Tệp cấu hình
Tệp cấu hình Redis tự động tạo tại `{dự-án-chính}/config/plugin/webman/redis-queue/redis.php`, nội dung tương tự như sau:
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Mật khẩu, tùy chọn
            'db' => 0,            // Cơ sở dữ liệu
            'max_attempts'  => 5, // Số lần thử lại sau khi tiêu thụ thất bại
            'retry_seconds' => 5, // Khoảng cách thử lại (giây)
        ]
    ],
];
```

### Thử lại khi tiêu thụ thất bại
Nếu tiêu thụ thất bại (phát sinh ngoại lệ), tin nhắn sẽ được đưa vào hàng đợi trễ và chờ thử lại lần sau. Số lần thử lại do `max_attempts` điều khiển, khoảng cách do `retry_seconds` và `max_attempts` cùng điều khiển. VD: `max_attempts` là 5, `retry_seconds` là 10 thì lần 1 cách `1*10` giây, lần 2 `2*10` giây, lần 3 `3*10` giây… đến 5 lần. Nếu vượt quá số lần thử lại đã thiết lập trong `max_attempts`, tin nhắn sẽ được đưa vào hàng đợi thất bại có key `{redis-queue}-failed`.

## Gửi tin nhắn (đồng bộ)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Tên hàng đợi
        $queue = 'send-mail';
        // Dữ liệu, có thể truyền mảng trực tiếp, không cần serialize
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Gửi tin nhắn
        Redis::send($queue, $data);
        // Gửi tin nhắn trễ, sẽ xử lý sau 60 giây
        Redis::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
Khi gửi thành công `Redis::send()` trả về true, không thì false hoặc ném ngoại lệ.

> **Gợi ý**
> Thời gian tiêu thụ hàng đợi trễ có thể có sai lệch. VD: tốc độ tiêu thụ chậm hơn tốc độ sản xuất khiến hàng đợi tích tụ và tiêu thụ trễ. Giảm nhẹ: chạy thêm process tiêu thụ.

## Gửi tin nhắn (bất đồng bộ)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Tên hàng đợi
        $queue = 'send-mail';
        // Dữ liệu, có thể truyền mảng trực tiếp, không cần serialize
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Gửi tin nhắn
        Client::send($queue, $data);
        // Gửi tin nhắn trễ, sẽ xử lý sau 60 giây
        Client::send($queue, $data, 60);

        return response('redis queue test');
    }

}
```
`Client::send()` không trả về giá trị. Đây là gửi bất đồng bộ và không đảm bảo 100% tin nhắn đến Redis.

> **Gợi ý**
> Nguyên lý của `Client::send()` là tạo hàng đợi bộ nhớ cục bộ và đồng bộ tin nhắn sang Redis một cách bất đồng bộ (đồng bộ rất nhanh, khoảng 10.000 tin/giây). Nếu process khởi động lại khi dữ liệu hàng đợi bộ nhớ cục bộ chưa đồng bộ xong thì có thể mất tin. Gửi bất đồng bộ `Client::send()` phù hợp cho tin không quan trọng.

> **Gợi ý**
> `Client::send()` là bất đồng bộ và chỉ dùng được trong môi trường chạy Workerman. Script dòng lệnh hãy dùng interface đồng bộ `Redis::send()`.

## Gửi tin nhắn từ dự án khác
Đôi khi cần gửi tin từ dự án khác và không dùng được `webman\redis-queue`. Khi đó có thể tham khảo hàm sau để gửi tin vào hàng đợi.

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

Ở đây tham số `$redis` là instance Redis. VD: dùng redis extension tương tự:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Tiêu thụ
Tệp cấu hình process consumer ở `{dự-án-chính}/config/plugin/webman/redis-queue/process.php`. Thư mục consumer nằm dưới `{dự-án-chính}/app/queue/redis/`.

Lệnh `php webman redis-queue:consumer my-send-mail` sẽ tạo tệp `{dự-án-chính}/app/queue/redis/MyMailSend.php`.

> **Gợi ý**
> Lệnh này cần cài plugin [Console](../plugin/console.md). Nếu không muốn cài, có thể tự tạo tệp tương tự như sau:

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Tên hàng đợi cần tiêu thụ
    public $queue = 'send-mail';

    // Tên kết nối, tương ứng kết nối trong plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Tiêu thụ
    public function consume($data)
    {
        // Không cần deserialize
        var_export($data); // Xuất ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Callback khi tiêu thụ thất bại
    /* 
    $package = [
        'id' => 1357277951, // ID tin nhắn
        'time' => 1709170510, // Thời gian tin nhắn
        'delay' => 0, // Thời gian trễ
        'attempts' => 2, // Số lần tiêu thụ
        'queue' => 'send-mail', // Tên hàng đợi
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Nội dung tin nhắn
        'max_attempts' => 5, // Số lần thử lại tối đa
        'error' => 'Thông báo lỗi' // Thông báo lỗi
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Không cần deserialize
        var_export($package); 
    }
}
```

> **Lưu ý**
> Không ném ngoại lệ và Error trong lúc tiêu thụ thì coi là tiêu thụ thành công; không thì thất bại và tin nhắn vào hàng đợi thử lại. redis-queue không có cơ chế ack; có thể coi như tự động ack (khi không có ngoại lệ hay Error). Muốn đánh dấu tin hiện tại tiêu thụ không thành công thì ném ngoại lệ thủ công để đưa tin vào hàng đợi thử lại. Thực tế không khác cơ chế ack.

> **Gợi ý**
> Consumer hỗ trợ đa máy chủ, đa process, và cùng một tin nhắn **không** bị tiêu thụ hai lần. Tin đã tiêu thụ sẽ tự xóa khỏi hàng đợi; không cần xóa thủ công.

> **Gợi ý**
> Process consumer có thể tiêu thụ nhiều hàng đợi khác nhau cùng lúc. Thêm hàng đợi mới không cần sửa cấu hình trong `process.php`. Khi thêm consumer hàng đợi mới, chỉ cần thêm class `Consumer` tương ứng dưới `app/queue/redis` và dùng thuộc tính `$queue` để chỉ định tên hàng đợi cần tiêu thụ.

> **Gợi ý**
> Người dùng Windows cần chạy `php windows.php` để khởi động webman, không thì process consumer sẽ không chạy.

> **Gợi ý**
> Callback onConsumeFailure được gọi mỗi khi tiêu thụ thất bại. Có thể xử lý logic sau thất bại tại đây. (Tính năng này cần `webman/redis-queue>=1.3.2` và `workerman/redis-queue>=1.2.1`)

## Đặt process consumer khác nhau cho hàng đợi khác nhau
Mặc định mọi consumer dùng chung một process. Đôi khi cần tách tiêu thụ một số hàng đợi—vd: nghiệp vụ tiêu thụ chậm vào một nhóm process, nghiệp vụ tiêu thụ nhanh vào nhóm khác. Để làm vậy có thể chia consumer thành hai thư mục, vd `app_path() . '/queue/redis/fast'` và `app_path() . '/queue/redis/slow'` (cần cập nhật namespace của class consumer tương ứng). Cấu hình như sau:
```php
return [
    ...cấu hình khác bỏ qua...
    
    'redis_consumer_fast'  => [ // key là tùy chỉnh, không giới hạn định dạng, ở đây đặt redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Thư mục class consumer
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // key là tùy chỉnh, không giới hạn định dạng, ở đây đặt redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Thư mục class consumer
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Như vậy consumer nghiệp vụ nhanh vào thư mục `queue/redis/fast`, consumer nghiệp vụ chậm vào `queue/redis/slow`, đạt mục đích gán process consumer cho từng hàng đợi.

## Cấu hình Redis nhiều
#### Cấu hình
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Mật khẩu, kiểu chuỗi, tùy chọn
            'db' => 0,            // Cơ sở dữ liệu
            'max_attempts'  => 5, // Số lần thử lại sau tiêu thụ thất bại
            'retry_seconds' => 5, // Khoảng cách thử lại (giây)
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Mật khẩu, kiểu chuỗi, tùy chọn
            'db' => 0,            // Cơ sở dữ liệu
            'max_attempts'  => 5, // Số lần thử lại sau tiêu thụ thất bại
            'retry_seconds' => 5, // Khoảng cách thử lại (giây)
        ]
    ],
];
```

Lưu ý: đã thêm cấu hình Redis với key `other`.

#### Gửi tin nhắn tới Redis nhiều

```php
// Gửi tin đến hàng đợi có key `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Giống
Client::send($queue, $data);
Redis::send($queue, $data);

// Gửi tin đến hàng đợi có key `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Tiêu thụ từ Redis nhiều
Tiêu thụ tin nhắn từ hàng đợi có key `other` trong cấu hình:
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Tên hàng đợi cần tiêu thụ
    public $queue = 'send-mail';

    // === Đặt 'other' ở đây để tiêu thụ từ hàng đợi có key 'other' trong cấu hình ===
    public $connection = 'other';

    // Tiêu thụ
    public function consume($data)
    {
        // Không cần deserialize
        var_export($data);
    }
}
```

## Câu hỏi thường gặp

**Vì sao có lỗi `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)`?**

Lỗi này chỉ xảy ra với interface gửi bất đồng bộ `Client::send()`. Gửi bất đồng bộ trước tiên lưu tin trong bộ nhớ cục bộ, rồi gửi sang Redis khi process rảnh. Nếu Redis nhận tin chậm hơn tốc độ sản xuất, hoặc process bận việc khác không đủ thời gian đồng bộ tin từ bộ nhớ sang Redis, sẽ tích tụ tin. Tin tích tụ quá 600 giây sẽ kích hoạt lỗi này.

Giải pháp: dùng interface gửi đồng bộ `Redis::send()` để gửi tin.
