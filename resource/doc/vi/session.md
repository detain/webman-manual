# Quản lý phiên webman

## Ví dụ
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Lấy thể hiện `Workerman\Protocols\Http\Session` qua `$request->session();` và dùng các phương thức của nó để thêm, sửa hoặc xóa dữ liệu phiên.

> **Lưu ý**
> Dữ liệu phiên được lưu tự động khi đối tượng phiên bị hủy.
> Lưu đối tượng phiên vào biến toàn cục sẽ ngăn nó bị hủy nên không lưu tự động. Trong trường hợp đó cần gọi thủ công `$session->save()` để lưu dữ liệu.

## Lấy tất cả dữ liệu phiên
```php
$session = $request->session();
$all = $session->all();
```
Trả về mảng. Nếu không có dữ liệu phiên thì trả về mảng rỗng.


## Lấy một giá trị từ phiên
```php
$session = $request->session();
$name = $session->get('name');
```
Trả về null nếu dữ liệu không tồn tại.

Bạn có thể truyền giá trị mặc định làm tham số thứ hai cho phương thức `get`. Nếu không tìm thấy giá trị tương ứng trong mảng phiên thì trả về giá trị mặc định. Ví dụ:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Lưu dữ liệu phiên
Dùng phương thức `set` để lưu một dữ liệu.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Phương thức `set` không trả về giá trị. Dữ liệu phiên được lưu tự động khi đối tượng phiên bị hủy.

Dùng phương thức `put` để lưu nhiều giá trị.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Tương tự, phương thức `put` không trả về giá trị.

## Xóa dữ liệu phiên
Dùng phương thức `forget` để xóa một hoặc nhiều dữ liệu phiên.
```php
$session = $request->session();
// Xóa một mục
$session->forget('name');
// Xóa nhiều mục
$session->forget(['name', 'age']);
```

Hệ thống cũng có phương thức `delete`. Khác `forget`, nó chỉ xóa được một mục.
```php
$session = $request->session();
// Tương đương $session->forget('name');
$session->delete('name');
```


## Lấy và xóa giá trị từ phiên
```php
$session = $request->session();
$name = $session->pull('name');
```
Tương đương đoạn mã sau:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Trả về null nếu giá trị phiên tương ứng không tồn tại.


## Xóa tất cả dữ liệu phiên
```php
$request->session()->flush();
```
Không trả về giá trị. Dữ liệu phiên sẽ được xóa khỏi lưu trữ tự động khi đối tượng phiên bị hủy.


## Kiểm tra giá trị phiên có tồn tại không
```php
$session = $request->session();
$has = $session->has('name');
```
Trả về false nếu giá trị phiên không tồn tại hoặc là null; ngược lại trả về true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Đoạn mã trên cũng dùng để kiểm tra giá trị phiên có tồn tại không. Khác ở chỗ `exists` vẫn trả về true khi giá trị là null.

## Hàm trợ giúp session()

webman cung cấp hàm trợ giúp `session()` để thực hiện chức năng tương tự.
```php
// Lấy thể hiện phiên
$session = session();
// Tương đương
$session = $request->session();

// Lấy giá trị
$value = session('key', 'default');
// Tương đương
$value = session()->get('key', 'default');
// Tương đương
$value = $request->session()->get('key', 'default');

// Gán giá trị cho phiên
session(['key1'=>'value1', 'key2' => 'value2']);
// Tương đương
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Tương đương
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Tệp cấu hình
Tệp cấu hình phiên nằm ở `config/session.php`. Nội dung tương tự như sau:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class hoặc RedisSessionHandler::class hoặc RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Nếu handler là FileSessionHandler::class thì giá trị là 'file',
    // nếu handler là RedisSessionHandler::class thì giá trị là 'redis'
    // nếu handler là RedisClusterSessionHandler::class thì giá trị là 'redis_cluster' (cụm Redis)
    'type'    => 'file',

    // Mỗi handler dùng cấu hình khác nhau
    'config' => [
        // Cấu hình khi type là 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Cấu hình khi type là 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Tên cookie lưu session_id
    'auto_update_timestamp' => false,  // Tự động làm mới phiên hay không, mặc định: tắt
    'lifetime' => 7*24*60*60,          // Thời gian hết hạn phiên
    'cookie_lifetime' => 365*24*60*60, // Thời gian hết hạn cookie chứa session_id
    'cookie_path' => '/',              // Đường dẫn cookie chứa session_id
    'domain' => '',                    // Tên miền cookie chứa session_id
    'http_only' => true,               // Bật httpOnly hay không, mặc định: bật
    'secure' => false,                 // Chỉ bật phiên qua HTTPS, mặc định: tắt
    'same_site' => '',                 // Phòng CSRF và theo dõi người dùng, giá trị: strict/lax/none
    'gc_probability' => [1, 1000],     // Xác suất thu hồi phiên
];
```

## Bảo mật
Không nên lưu trực tiếp thể hiện lớp vào phiên, đặc biệt từ nguồn không tin cậy. Giải tuần tự hóa có thể gây rủi ro bảo mật.

