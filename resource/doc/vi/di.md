# Tiêm phụ thuộc tự động

Trong webman, tiêm phụ thuộc tự động là tính năng tùy chọn và mặc định bị tắt. Nếu cần tiêm phụ thuộc tự động, nên dùng [php-di](https://php-di.org/doc/getting-started.html). Dưới đây mô tả cách dùng `php-di` với webman.

## Cài đặt

```
composer require php-di/php-di:^7.0
```

Chỉnh sửa cấu hình `config/container.php`. Nội dung cuối cùng phải như sau:

```php
$builder = new \DI\ContainerBuilder();
$builder->addDefinitions(config('dependence', []));
$builder->useAutowiring(true);
$builder->useAttributes(true);
return $builder->build();
```

> File `config/container.php` cuối cùng phải trả về một instance container tuân thủ đặc tả PSR-11. Nếu không muốn dùng `php-di`, bạn có thể tạo và trả về một instance container tuân thủ PSR-11 khác tại đây. Cấu hình mặc định chỉ cung cấp chức năng container cơ bản của webman.

## Tiêm qua constructor

Tạo file `app/service/Mailer.php` (tạo thư mục nếu chưa có) với nội dung sau:

```php
<?php
namespace app\service;

class Mailer
{
    public function mail($email, $content)
    {
        // Mã gửi email được bỏ qua
    }
}
```

Nội dung `app/controller/UserController.php`:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{

    public function __construct(private Mailer $mailer)
    {
    }

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Xin chào và chào mừng!');
        return response('ok');
    }
}
```

Thông thường, cần đoạn code sau để khởi tạo instance `app\controller\UserController`:

```php
$mailer = new Mailer;
$user = new UserController($mailer);
```

Khi dùng `php-di`, lập trình viên không cần khởi tạo thủ công `Mailer` trong controller — webman sẽ làm tự động. Nếu quá trình khởi tạo `Mailer` có phụ thuộc lớp khác, webman cũng tự động khởi tạo và tiêm. Không cần thao tác khởi tạo nào từ lập trình viên.

> **Lưu ý**
> Chỉ instance do framework hoặc `php-di` tạo mới hỗ trợ tiêm phụ thuộc tự động. Instance tạo thủ công bằng `new` không thể dùng. Muốn tiêm, dùng interface `support\Container` thay cho `new`, ví dụ:

```php
use app\service\UserService;
use app\service\LogService;
use support\Container;

// Instance tạo bằng new không hỗ trợ tiêm phụ thuộc
$user_service = new UserService;
// Instance tạo bằng new không hỗ trợ tiêm phụ thuộc
$log_service = new LogService($path, $name);

// Instance tạo bằng Container hỗ trợ tiêm phụ thuộc
$user_service = Container::get(UserService::class);
// Instance tạo bằng Container hỗ trợ tiêm phụ thuộc
$log_service = Container::make(LogService::class, [$path, $name]);
```

## Tiêm qua thuộc tính (Attributes)

Ngoài tiêm qua constructor, có thể dùng tiêm qua thuộc tính. Tiếp tục ví dụ trước, sửa `app\controller\UserController` như sau:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private Mailer $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Xin chào và chào mừng!');
        return response('ok');
    }
}
```

Ví dụ này dùng thuộc tính `#[Inject]` để tiêm và tự động tiêm instance vào biến thành viên theo kiểu đối tượng. Hiệu quả tương đương tiêm qua constructor nhưng code gọn hơn.

> **Lưu ý**
> webman không hỗ trợ tiêm tham số controller trước phiên bản 1.4.6. Ví dụ đoạn code sau không được hỗ trợ khi webman<=1.4.6:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\Mailer;

class UserController
{
    // Tiêm tham số controller không hỗ trợ trước phiên bản 1.4.6
    public function register(Request $request, Mailer $mailer)
    {
        $mailer->mail('hello@webman.com', 'Xin chào và chào mừng!');
        return response('ok');
    }
}
```

## Tiêm constructor tùy chỉnh

Đôi khi tham số constructor có thể không phải instance của lớp mà là chuỗi, số, mảng hoặc dữ liệu khác. Ví dụ constructor Mailer cần IP và cổng máy chủ SMTP:

```php
<?php
namespace app\service;

class Mailer
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Mã gửi email được bỏ qua
    }
}
```

Trường hợp này không thể dùng tiêm tự động qua constructor vì `php-di` không xác định được giá trị `$smtp_host` và `$smtp_port`. Có thể dùng tiêm tùy chỉnh.

Thêm đoạn code sau vào `config/dependence.php` (tạo file nếu chưa có):

```php
return [
    // ... Cấu hình khác được bỏ qua

    app\service\Mailer::class =>  new app\service\Mailer('192.168.1.11', 25);
];
```

Khi tiêm phụ thuộc cần lấy instance `app\service\Mailer`, sẽ tự động dùng instance được tạo trong cấu hình này.

Lưu ý rằng `config/dependence.php` dùng `new` để khởi tạo lớp `Mailer`. Trong ví dụ này không vấn đề gì, nhưng nếu lớp `Mailer` phụ thuộc lớp khác hoặc dùng tiêm qua thuộc tính bên trong, khởi tạo bằng `new` sẽ không thực hiện tiêm phụ thuộc tự động. Cách xử lý là dùng tiêm tùy chỉnh qua interface và khởi tạo lớp qua `Container::get(tên lớp)` hoặc `Container::make(tên lớp, [tham số constructor])`.

## Tiêm tùy chỉnh qua interface

Trong dự án thực tế, nên lập trình hướng interface hơn là lớp cụ thể. Ví dụ `app\controller\UserController` nên phụ thuộc `app\service\MailerInterface` thay vì `app\service\Mailer`.

Định nghĩa interface `MailerInterface`:

```php
<?php
namespace app\service;

interface MailerInterface
{
    public function mail($email, $content);
}
```

Định nghĩa implementation của `MailerInterface`:

```php
<?php
namespace app\service;

class Mailer implements MailerInterface
{
    private $smtpHost;

    private $smtpPort;

    public function __construct($smtp_host, $smtp_port)
    {
        $this->smtpHost = $smtp_host;
        $this->smtpPort = $smtp_port;
    }

    public function mail($email, $content)
    {
        // Mã gửi email được bỏ qua
    }
}
```

Dùng interface `MailerInterface` thay cho implementation cụ thể:

```php
<?php
namespace app\controller;

use support\Request;
use app\service\MailerInterface;
use DI\Attribute\Inject;

class UserController
{
    #[Inject]
    private MailerInterface $mailer;

    public function register(Request $request)
    {
        $this->mailer->mail('hello@webman.com', 'Xin chào và chào mừng!');
        return response('ok');
    }
}
```

Định nghĩa implementation của `MailerInterface` trong `config/dependence.php`:

```php
use Psr\Container\ContainerInterface;

return [
    app\service\MailerInterface::class => function(ContainerInterface $container) {
        return $container->make(app\service\Mailer::class, ['smtp_host' => '192.168.1.11', 'smtp_port' => 25]);
    }
];
```

Khi nghiệp vụ cần dùng interface `MailerInterface`, sẽ tự động dùng implementation `Mailer`.

> Lợi ích của lập trình hướng interface là khi cần thay thế thành phần không cần sửa code nghiệp vụ, chỉ thay đổi implementation cụ thể trong `config/dependence.php`. Rất hữu ích cho kiểm thử đơn vị.

## Tiêm tùy chỉnh khác

Ngoài phụ thuộc lớp, `config/dependence.php` còn có thể định nghĩa giá trị khác như chuỗi, số, mảng...

Ví dụ nếu `config/dependence.php` định nghĩa như sau:

```php
return [
    'smtp_host' => '192.168.1.11',
    'smtp_port' => 25
];
```

Có thể dùng `#[Inject]` để tiêm `smtp_host` và `smtp_port` vào thuộc tính lớp:

```php
<?php
namespace app\service;

use DI\Attribute\Inject;

class Mailer
{
    #[Inject("smtp_host")]
    private $smtpHost;

    #[Inject("smtp_port")]
    private $smtpPort;

    public function mail($email, $content)
    {
        // Mã gửi email được bỏ qua
        echo "{$this->smtpHost}:{$this->smtpPort}\n"; // Sẽ hiển thị 192.168.1.11:25
    }
}
```

# Tải lazy (Lazy Loading)

> Tải lazy là mẫu thiết kế trì hoãn việc tạo hoặc khởi tạo đối tượng cho đến khi thực sự cần dùng.

Tính năng này cần thêm dependency. Gói sau là fork của `ocramius/proxy-manager`; repo gốc không hỗ trợ PHP 8.

```
composer require friendsofphp/proxy-manager-lts
```

Cách dùng:

```php
<?php

use DI\Attribute\Injectable;
use DI\Attribute\Inject;

#[Injectable(lazy: true)]
class MyClass
{
    private string $name;

    public function __construct()
    {
        echo "MyClass được khởi tạo\n";
        $this->name = "Lazy Loaded Object";
    }

    public function getName(): string
    {
        return $this->name;
    }
}

class Controller
{
    #[Inject]
    public MyClass $myClass;

    public function getClass()
    {
        echo "Tên lớp proxy: " . get_class($this->myClass) . "\n";
        echo "name: " . $this->myClass->getName();

    }
}
```

Kết quả:

```
Tên lớp proxy: ProxyManagerGeneratedProxy\__PM__\app\web\MyClass\Generated98d2817da63e3c088c808a0d4f6e9ae0
MyClass được khởi tạo
name: Lazy Loaded Object
```

Ví dụ này cho thấy khi lớp khai báo thuộc tính `#[Injectable]` được tiêm, trước tiên tạo lớp proxy. Lớp thực chỉ được khởi tạo khi bất kỳ phương thức nào của nó được gọi.

# Phụ thuộc vòng (Circular Dependencies)

Phụ thuộc vòng xảy ra khi nhiều lớp phụ thuộc lẫn nhau, tạo chu trình khép kín.

- Phụ thuộc vòng trực tiếp
  - Module A phụ thuộc module B, module B phụ thuộc module A
  - Tạo chu trình: A → B → A

- Phụ thuộc vòng gián tiếp
  - Nhiều module trong chu trình phụ thuộc
  - Ví dụ: A → B → C → A

Khi dùng tiêm qua thuộc tính, `php-di` tự phát hiện phụ thuộc vòng và ném ngoại lệ. Nếu cần, dùng cách sau:

```php
class userController
{

    // Xóa đoạn code này
    // #[Inject]
    // private UserService userService;

    public function getUserName()
    {
        $userService = Container::get(UserService::class);
        return $userService->getName();
    }
}
```

## Thêm thông tin

Xem [tài liệu php-di](https://php-di.org/doc/getting-started.html).
