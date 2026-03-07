# Cơ sở dữ liệu

Vì hầu hết các plugin đều cài đặt [webman-admin](https://www.workerman.net/plugin/82), nên khuyến nghị tái sử dụng trực tiếp cấu hình cơ sở dữ liệu của `webman-admin`.

Các model có lớp cơ sở là `plugin\admin\app\model\Base` sẽ tự động sử dụng cấu hình cơ sở dữ liệu của webman-admin.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

Bạn cũng có thể truy cập cơ sở dữ liệu webman-admin qua `plugin.admin.mysql`, ví dụ:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Sử dụng cơ sở dữ liệu riêng

Các plugin cũng có thể chọn sử dụng cơ sở dữ liệu riêng. Ví dụ, nội dung của `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql là tên kết nối
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'cơ_sở_dữ_liệu',
            'username'    => 'tên_đăng_nhập',
            'password'    => 'mật_khẩu',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin là tên kết nối
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'cơ_sở_dữ_liệu',
            'username'    => 'tên_đăng_nhập',
            'password'    => 'mật_khẩu',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

Định dạng tham chiếu là `Db::connection('plugin.{plugin}.{tên_kết_nối}');`, ví dụ:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Để sử dụng cơ sở dữ liệu của dự án chính, hãy gọi trực tiếp:

```php
use support\Db;
Db::table('user')->first();
// Giả sử dự án chính cũng có cấu hình kết nối admin
Db::connection('admin')->table('admin')->first();
```

#### Cấu hình cơ sở dữ liệu cho Model

Bạn có thể tạo lớp Base cho Model và đặt thuộc tính `$connection` để sử dụng kết nối cơ sở dữ liệu của chính plugin:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

Như vậy tất cả model trong plugin kế thừa từ Base sẽ tự động sử dụng cơ sở dữ liệu riêng của plugin.

## Tự động nhập cơ sở dữ liệu
Chạy `php webman app-plugin:create foo` sẽ tự động tạo plugin foo, bao gồm `plugin/foo/api/Install.php` và `plugin/foo/install.sql`.

> **Mẹo**
> Nếu tệp install.sql không được tạo, hãy thử nâng cấp `webman/console`: `composer require webman/console ^1.3.6`

#### Nhập cơ sở dữ liệu khi cài đặt plugin
Khi cài đặt plugin, phương thức `install` trong Install.php sẽ được thực thi, phương thức này tự động chạy các câu lệnh SQL trong `install.sql`, qua đó nhập các bảng cơ sở dữ liệu.

Nội dung của `install.sql` là tạo bảng và các câu lệnh SQL thay đổi lịch sử bảng, lưu ý mỗi câu lệnh phải kết thúc bằng `;`, ví dụ:
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Khóa chính',
  `order_id` varchar(50) NOT NULL COMMENT 'ID đơn hàng',
  `user_id` int NOT NULL COMMENT 'ID người dùng',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Số tiền thanh toán',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Đơn hàng';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Khóa chính',
  `name` varchar(50) NOT NULL COMMENT 'Tên',
  `price` int NOT NULL COMMENT 'Giá',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Sản phẩm';
```

**Thay đổi kết nối cơ sở dữ liệu**

Đích nhập của `install.sql` mặc định là cơ sở dữ liệu của webman-admin. Nếu muốn nhập vào cơ sở dữ liệu khác, hãy sửa thuộc tính `$connection` trong `Install.php`, ví dụ:
```php
<?php

class Install
{
    // Chỉ định cơ sở dữ liệu riêng của plugin
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Kiểm tra**

Chạy `php webman app-plugin:install foo` để cài đặt plugin, sau đó kiểm tra cơ sở dữ liệu, các bảng `foo_orders` và `foo_goods` đã được tạo.

#### Thay đổi cấu trúc bảng khi nâng cấp plugin
Đôi khi nâng cấp plugin cần tạo bảng mới hoặc thay đổi cấu trúc bảng, có thể thêm trực tiếp các câu lệnh tương ứng vào cuối `install.sql`, lưu ý mỗi câu lệnh kết thúc bằng `;`, ví dụ thêm bảng `foo_user` và thêm cột `status` vào bảng `foo_orders`
```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Khóa chính',
  `order_id` varchar(50) NOT NULL COMMENT 'ID đơn hàng',
  `user_id` int NOT NULL COMMENT 'ID người dùng',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Số tiền thanh toán',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Đơn hàng';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Khóa chính',
 `name` varchar(50) NOT NULL COMMENT 'Tên',
 `price` int NOT NULL COMMENT 'Giá',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Sản phẩm';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Khóa chính',
 `name` varchar(50) NOT NULL COMMENT 'Tên'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Người dùng';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Trạng thái';
```

Khi nâng cấp, phương thức `update` trong Install.php sẽ được thực thi, phương thức này cũng chạy các câu lệnh trong `install.sql`, câu lệnh mới sẽ được thực thi, câu lệnh cũ sẽ bỏ qua, qua đó thực hiện thay đổi cơ sở dữ liệu khi nâng cấp.

#### Xóa cơ sở dữ liệu khi gỡ cài đặt plugin
Khi gỡ cài đặt plugin, phương thức `uninstall` trong Install.php sẽ được gọi, nó tự động phân tích các câu lệnh CREATE TABLE trong `install.sql` và tự động xóa các bảng đó, đạt mục đích xóa bảng cơ sở dữ liệu khi gỡ cài đặt plugin.
Nếu khi gỡ cài đặt chỉ muốn thực thi `uninstall.sql` riêng, không thực thi thao tác xóa bảng tự động, chỉ cần tạo `plugin/{tên_plugin}/uninstall.sql`, như vậy phương thức `uninstall` chỉ thực thi các câu lệnh trong tệp `uninstall.sql`.
