# Sử dụng giao dịch đúng cách

Cách sử dụng giao dịch cơ sở dữ liệu trong webman giống các framework khác. Dưới đây là những điểm cần lưu ý.

## Cấu trúc mã

Cấu trúc mã giống các framework khác (ví dụ Laravel, think-orm tương tự):

```php
Db::beginTransaction();
try {
    // ..xử lý nghiệp vụ bỏ qua...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Quan trọng:** Cần **bắt buộc** dùng `\Throwable` chứ **không** dùng `\Exception`, vì trong quá trình xử lý nghiệp vụ có thể kích hoạt `Error`, và `Error` không kế thừa từ `Exception`.

## Kết nối cơ sở dữ liệu

Khi thao tác với model trong giao dịch, cần chú ý model có thiết lập kết nối hay không. Nếu model đã chỉ định kết nối thì khi bắt đầu giao dịch phải chỉ định kết nối đó; nếu không giao dịch sẽ không có hiệu lực (think-orm tương tự). Ví dụ:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Kết nối đã chỉ định cho model
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Khi model đã chỉ định kết nối, việc bắt đầu giao dịch, commit và rollback đều phải chỉ định kết nối:

```php
Db::connection('mysql')->beginTransaction();
try {
    // Xử lý nghiệp vụ
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Tìm các yêu cầu có giao dịch chưa commit

Đôi khi lỗi trong mã nghiệp vụ khiến giao dịch của một yêu cầu không được commit. Để nhanh chóng xác định phương thức controller nào không commit giao dịch, bạn có thể cài đặt component `webman/log`. Component này sau khi mỗi yêu cầu kết thúc sẽ tự động kiểm tra xem có giao dịch chưa commit hay không và ghi vào nhật ký. Từ khóa trong nhật ký là `Uncommitted transactions`.

**Cách cài đặt webman/log**

`composer require webman/log`

> **Lưu ý**
> Sau khi cài đặt cần **restart** để khởi động lại; reload không có hiệu lực.
