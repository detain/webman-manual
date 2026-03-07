# Xử lý công việc chậm

Đôi khi chúng ta cần xử lý các công việc chậm để tránh ảnh hưởng đến việc xử lý các yêu cầu khác của webman. Tùy theo tình huống, các công việc này có thể sử dụng các phương án xử lý khác nhau.

## Phương án 1: Sử dụng hàng đợi tin nhắn
Tham khảo [hàng đợi Redis](../queue/redis.md) [hàng đợi Stomp](../queue/stomp.md)

#### Ưu điểm
Có thể xử lý các yêu cầu xử lý công việc lớn đột ngột

#### Nhược điểm
Không thể trả kết quả trực tiếp cho khách hàng. Nếu cần đẩy kết quả phải phối hợp với dịch vụ khác, ví dụ sử dụng [webman/push](https://www.workerman.net/plugin/2) để đẩy kết quả xử lý.

## Phương án 2: Thêm cổng HTTP mới

Thêm cổng HTTP mới để xử lý các yêu cầu chậm. Những yêu cầu chậm này qua cổng này được một nhóm quy trình cụ thể xử lý, sau khi xử lý trả kết quả trực tiếp cho khách hàng.

#### Ưu điểm
Có thể trả dữ liệu trực tiếp cho khách hàng

#### Nhược điểm
Không thể xử lý các yêu cầu lớn đột ngột

#### Các bước thực hiện
Thêm cấu hình sau vào `config/process.php`.
```php
return [
    // ... Bỏ qua các cấu hình khác ở đây ...
    
    'task' => [
        'handler' => \Webman\App::class,
        'listen' => 'http://0.0.0.0:8686',
        'count' => 8, // Số lượng quy trình
        'user' => '',
        'group' => '',
        'reusePort' => true,
        'constructor' => [
            'requestClass' => \support\Request::class, // Cài đặt lớp request
            'logger' => \support\Log::channel('default'), // Thể hiện logger
            'appPath' => app_path(), // Vị trí thư mục app
            'publicPath' => public_path() // Vị trí thư mục public
        ]
    ]
];
```

Như vậy các giao diện chậm có thể đi qua nhóm quy trình tại `http://127.0.0.1:8686/` mà không ảnh hưởng đến việc xử lý công việc của các quy trình khác.

Để giao diện người dùng không cảm nhận sự khác biệt về cổng, có thể thêm proxy tới cổng 8686 trong nginx. Giả sử đường dẫn yêu cầu giao diện chậm đều bắt đầu bằng `/task`, cấu hình nginx sẽ tương tự như sau:
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

# Thêm upstream 8686 mới
upstream task {
   server 127.0.0.1:8686;
   keepalive 10240;
}

server {
  server_name webman.com;
  listen 80;
  access_log off;
  root /path/webman/public;

  # Các yêu cầu bắt đầu bằng /task đi qua cổng 8686, vui lòng thay đổi /task thành tiền tố cần thiết theo tình hình
  location /task {
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_http_version 1.1;
      proxy_set_header Connection "";
      proxy_pass http://task;
  }

  # Các yêu cầu khác đi qua cổng 8787 ban đầu
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

Như vậy khi khách hàng truy cập `domain.com/task/xxx` sẽ được xử lý qua cổng 8686 riêng biệt, không ảnh hưởng đến việc xử lý yêu cầu cổng 8787.

## Phương án 3: Sử dụng HTTP Chunked gửi dữ liệu phân đoạn bất đồng bộ

#### Ưu điểm
Có thể trả dữ liệu trực tiếp cho khách hàng

**Cài đặt workerman/http-client**

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
            $connection->send(new Chunk('')); // Gửi chunk rỗng biểu thị kết thúc phản hồi
        });
        // Gửi header HTTP trước, dữ liệu sau được gửi bất đồng bộ
        return response()->withHeaders([
            "Transfer-Encoding" => "chunked",
        ]);
    }
}
```

> **Gợi ý**
> Ví dụ này sử dụng client `workerman/http-client` để lấy kết quả HTTP bất đồng bộ và trả dữ liệu. Cũng có thể sử dụng client bất đồng bộ khác như [AsyncTcpConnection](https://www.workerman.net/doc/workerman/async-tcp-connection/construct.html).
