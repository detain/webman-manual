# Quy trình tạo và phát hành plugin cơ bản

## Nguyên lý
1. Lấy plugin cross-origin làm ví dụ: plugin gồm ba phần – file middleware cross-origin, file cấu hình `middleware.php` và `Install.php` được tạo tự động bằng lệnh.
2. Dùng lệnh để đóng gói ba file này và phát hành lên Composer.
3. Khi người dùng cài plugin cross-origin bằng Composer, `Install.php` sao chép file middleware và cấu hình vào `{dự án chính}/config/plugin` để webman tải và kích hoạt cross-origin.
4. Khi người dùng gỡ plugin bằng Composer, `Install.php` xóa các file middleware và cấu hình tương ứng, thực hiện gỡ cài tự động.

## Quy định
1. Tên plugin gồm hai phần: `vendor` và `tên plugin`, ví dụ `webman/push`, tương ứng với tên gói Composer.
2. File cấu hình plugin đặt trong `config/plugin/vendor/tên plugin/` (lệnh console tự tạo thư mục cấu hình). Nếu plugin không cần cấu hình thì phải xóa thư mục tự tạo.
3. Thư mục cấu hình plugin chỉ hỗ trợ: `app.php` (cấu hình chính), `bootstrap.php` (khởi động tiến trình), `route.php` (định tuyến), `middleware.php` (middleware), `process.php` (tiến trình tùy chỉnh), `database.php` (cơ sở dữ liệu), `redis.php` (Redis), `thinkorm.php` (thinkorm). webman tự nhận diện các file này.
4. Truy cập cấu hình: `config('plugin.vendor.tên plugin.file cấu hình.mục');`, ví dụ `config('plugin.webman.push.app.app_key')`.
5. Nếu plugin có cấu hình cơ sở dữ liệu riêng: `illuminate/database` dùng `Db::connection('plugin.vendor.tên plugin.kết nối')`, `thinkorm` dùng `Db::connect('plugin.vendor.tên plugin.kết nối')`.
6. Nếu plugin cần đặt file nghiệp vụ trong `app/` thì phải tránh xung đột với dự án chính và plugin khác.
7. Plugin nên tránh sao chép file hoặc thư mục sang dự án chính. Ví dụ plugin cross-origin chỉ sao chép cấu hình; file middleware để trong `vendor/webman/cros/src`.
8. Khuyến nghị dùng PascalCase cho namespace của plugin, ví dụ `Webman/Console`.

## Ví dụ

**Cài đặt `webman/console` dòng lệnh**

`composer require webman/console`

### Tạo plugin

Giả sử tên plugin cần tạo là `foo/admin` (cũng là tên dự án sẽ phát hành trên Composer, phải viết thường). Chạy:

`php webman plugin:create --name=foo/admin`

Sẽ tạo `vendor/foo/admin` (file plugin) và `config/plugin/foo/admin` (cấu hình).

> Ghi chú
> `config/plugin/foo/admin` hỗ trợ: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Định dạng giống webman, tự động merge.
> Truy cập dùng tiền tố `plugin`, ví dụ `config('plugin.foo.admin.app')`.


### Xuất plugin

Sau khi phát triển xong, chạy:

`php webman plugin:export --name=foo/admin`

Xuất

> Giải thích
> Khi xuất sẽ sao chép `config/plugin/foo/admin` vào `vendor/foo/admin/src` và tạo `Install.php`, chạy khi cài đặt/gỡ bỏ.
> Cài đặt mặc định: sao chép cấu hình từ `vendor/foo/admin/src` vào `config/plugin` của dự án.
> Gỡ bỏ mặc định: xóa file cấu hình plugin trong `config/plugin` của dự án.
> Có thể sửa `Install.php` để thêm logic tùy chỉnh khi cài đặt/gỡ bỏ.

### Gửi plugin
* Giả sử bạn đã có tài khoản [GitHub](https://github.com) và [Packagist](https://packagist.org).
* Tạo repository `admin` trên [GitHub](https://github.com) và đẩy mã nguồn, ví dụ `https://github.com/ten-dang-nhap/admin`.
* Truy cập `https://github.com/ten-dang-nhap/admin/releases/new` tạo release, ví dụ `v1.0.0`.
* Trên [Packagist](https://packagist.org) bấm `Submit`, gửi `https://github.com/ten-dang-nhap/admin` để phát hành plugin.

> **Mẹo**
> Nếu Packagist báo trùng tên thì chọn vendor khác, ví dụ đổi `foo/admin` thành `myfoo/admin`.

Khi cập nhật: đẩy mã nguồn lên GitHub, tạo release mới tại `https://github.com/ten-dang-nhap/admin/releases/new`, rồi bấm `Update` tại `https://packagist.org/packages/foo/admin`.

## Thêm lệnh cho plugin
Một số plugin cần lệnh tùy chỉnh. Ví dụ cài `webman/redis-queue` thì dự án có lệnh `redis-queue:consumer`. Chạy `php webman redis-queue:consumer send-mail` sẽ nhanh chóng tạo lớp consumer `SendMail.php`, hỗ trợ phát triển.

Để thêm lệnh `foo-admin:add` cho plugin `foo/admin`:

### Tạo lệnh

**Tạo file `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Mô tả lệnh';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Ghi chú**
> Để tránh xung đột lệnh giữa các plugin, dùng định dạng `vendor-plugin:lệnh`. Ví dụ mọi lệnh của plugin `foo/admin` phải có tiền tố `foo-admin:`, ví dụ `foo-admin:add`.

### Thêm cấu hình
**Tạo `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Thêm thêm nếu cần...
];
```

> **Mẹo**
> `command.php` dùng để đăng ký lệnh tùy chỉnh của plugin. Mỗi phần tử là một lớp lệnh. `webman/console` tự động tải. Xem thêm [Lệnh console](console.md).

### Chạy xuất
Chạy `php webman plugin:export --name=foo/admin` để xuất plugin và phát hành lên Packagist. Sau khi cài `foo/admin` sẽ có lệnh `foo-admin:add`. Chạy `php webman foo-admin:add jerry` sẽ in `Admin add jerry`.
