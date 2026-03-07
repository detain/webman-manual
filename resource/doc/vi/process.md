# Tiến trình tùy chỉnh

Trong webman, bạn có thể tùy chỉnh listener hoặc tiến trình giống như trong workerman.

> **Lưu ý**
> Người dùng Windows cần sử dụng `php windows.php` để khởi động webman mới có thể chạy tiến trình tùy chỉnh.

## Dịch vụ HTTP tùy chỉnh
Đôi khi bạn có nhu cầu đặc biệt cần sửa đổi mã lõi của dịch vụ HTTP webman. Trong trường hợp này có thể dùng tiến trình tùy chỉnh để thực hiện.

Ví dụ, tạo mới `app\Server.php`.

```php
<?php

namespace app;

use Webman\App;

class Server extends App
{
    // Ghi đè các phương thức trong Webman\App tại đây
}
```

Thêm cấu hình sau vào `config/process.php`.

```php
use Workerman\Worker;

return [
    // ... các cấu hình khác đã lược bỏ ...
    
    'my-http' => [
        'handler' => app\Server::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Số tiến trình
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Thiết lập lớp request
            'logger' => \support\Log::channel('default'), // Thể hiện log
            'appPath' => app_path(), // Vị trí thư mục app
            'publicPath' => public_path() // Vị trí thư mục public
        ]
    ]
];
```

> **Gợi ý**
> Nếu muốn tắt tiến trình HTTP đi kèm webman, chỉ cần đặt `listen=>''` trong config/server.php.

## Ví dụ listener WebSocket tùy chỉnh

Tạo mới `app/Pusher.php`.

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

> Lưu ý: Tất cả các phương thức onXXX phải là public.

Thêm cấu hình sau vào `config/process.php`.

```php
return [
    // ... các cấu hình tiến trình khác đã lược bỏ ...
    
    // websocket_test là tên tiến trình
    'websocket_test' => [
        // Chỉ định lớp tiến trình tại đây, tức là lớp Pusher đã định nghĩa ở trên
        'handler' => app\Pusher::class,
        'listen'  => 'websocket://0.0.0.0:8888',
        'count'   => 1,
    ],
];
```

## Ví dụ tiến trình không lắng nghe tùy chỉnh

Tạo mới `app/TaskTest.php`.

```php
<?php
namespace app;

use Workerman\Timer;
use support\Db;

class TaskTest
{
  
    public function onWorkerStart()
    {
        // Kiểm tra cơ sở dữ liệu mỗi 10 giây xem có người dùng mới đăng ký không
        Timer::add(10, function(){
            Db::table('users')->where('regist_timestamp', '>', time()-10)->get();
        });
    }
    
}
```

Thêm cấu hình sau vào `config/process.php`.

```php
return [
    // ... các cấu hình tiến trình khác đã lược bỏ ...
    
    'task' => [
        'handler'  => app\TaskTest::class
    ],
];
```

> Lưu ý: Nếu bỏ qua listen thì sẽ không lắng nghe bất kỳ cổng nào; nếu bỏ qua count thì số tiến trình mặc định là 1.

## Giải thích file cấu hình

Cấu hình đầy đủ của một tiến trình được định nghĩa như sau:

```php
return [
    // ... 
    
    // websocket_test là tên tiến trình
    'websocket_test' => [
        // Chỉ định lớp tiến trình tại đây
        'handler' => app\Pusher::class,
        // Giao thức, IP và cổng lắng nghe (tùy chọn)
        'listen'  => 'websocket://0.0.0.0:8888',
        // Số tiến trình (tùy chọn, mặc định 1)
        'count'   => 2,
        // Người dùng chạy tiến trình (tùy chọn, mặc định người dùng hiện tại)
        'user'    => '',
        // Nhóm người dùng chạy tiến trình (tùy chọn, mặc định nhóm hiện tại)
        'group'   => '',
        // Tiến trình hiện tại có hỗ trợ reload không (tùy chọn, mặc định true)
        'reloadable' => true,
        // Bật reusePort
        'reusePort'  => true,
        // transport (tùy chọn, đặt 'ssl' khi cần SSL, mặc định 'tcp')
        'transport'  => 'tcp',
        // context (tùy chọn, khi transport là ssl cần truyền đường dẫn chứng chỉ)
        'context'    => [], 
        // Tham số hàm tạo lớp tiến trình (tùy chọn)
        'constructor' => [],
        // Tiến trình này có được bật hay không
        'enable' => true
    ],
];
```

## Tổng kết

Tiến trình tùy chỉnh trong webman thực chất là một lớp bao đơn giản của workerman. Nó tách biệt cấu hình và nghiệp vụ, đồng thời triển khai callback `onXXX` của workerman thông qua các phương thức lớp. Cách sử dụng khác hoàn toàn giống workerman.
