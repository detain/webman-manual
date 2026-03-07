# Về rò rỉ bộ nhớ
Webman là framework thường trú trong bộ nhớ, vì vậy chúng ta cần chú ý một chút đến tình trạng rò rỉ bộ nhớ. Tuy nhiên, nhà phát triển không cần quá lo lắng vì rò rỉ bộ nhớ chỉ xảy ra trong điều kiện cực kỳ hiếm và dễ tránh. Trải nghiệm phát triển với webman về cơ bản giống framework truyền thống; không cần thao tác quản lý bộ nhớ thừa.

> **Gợi ý**
> Tiến trình monitor tích hợp sẵn của webman giám sát mức sử dụng bộ nhớ của tất cả tiến trình. Khi mức sử dụng bộ nhớ của một tiến trình sắp đạt giá trị cài đặt `memory_limit` trong php.ini, tiến trình tương ứng sẽ được tự động khởi động lại an toàn để giải phóng bộ nhớ, không ảnh hưởng đến ứng dụng.

## Định nghĩa rò rỉ bộ nhớ
Mức sử dụng bộ nhớ của webman tăng theo số yêu cầu là hiện tượng bình thường. Thông thường, khi một tiến trình đạt đến một khối lượng yêu cầu nhất định (thường ở mức hàng triệu), bộ nhớ sẽ ngừng tăng hoặc thi thoảng tăng nhẹ.

Đối với hầu hết ứng dụng, mức sử dụng bộ nhớ của mỗi tiến trình cuối cùng sẽ ổn định quanh mức 10M–100M. Không cần lo lắng nếu bộ nhớ mỗi tiến trình không vượt quá 100M.

Ngoài ra, khi xử lý tệp lớn, yêu cầu lớn hoặc đọc lượng lớn dữ liệu từ cơ sở dữ liệu, PHP sẽ cấp phát nhiều bộ nhớ. PHP có thể giữ lại một phần bộ nhớ này để tái sử dụng thay vì trả lại toàn bộ cho hệ điều hành, khiến mức sử dụng bộ nhớ cao. Vì bộ nhớ được tái sử dụng nên không cần lo lắng.

> **Gợi ý**
> Đối với dự án đóng gói phar hoặc nhị phân, nếu kích thước gói lớn thì việc mức sử dụng bộ nhớ vượt 100M là bình thường.

## Cách xác nhận rò rỉ bộ nhớ
Nếu một tiến trình đã xử lý hơn một triệu yêu cầu, mức sử dụng bộ nhớ vượt 100M và bộ nhớ vẫn tăng sau mỗi yêu cầu thì có thể đang xảy ra rò rỉ bộ nhớ.

## Cách xác định vị trí rò rỉ bộ nhớ
Cách đơn giản là thực hiện kiểm tra tải trên từng API và xác định API nào tiếp tục tăng mức sử dụng bộ nhớ sau hàng triệu yêu cầu.

Sau khi tìm được API có vấn đề, dùng phương pháp tìm kiếm nhị phân: mỗi lần chú thích bỏ nửa mã nghiệp vụ cho đến khi xác định được đoạn mã gây ra sự cố.

## Rò rỉ bộ nhớ xảy ra như thế nào
**Rò rỉ bộ nhớ chỉ xảy ra khi đáp ứng đầy đủ cả hai điều kiện sau:**
1. Tồn tại mảng có **chu kỳ sống dài** (mảng thông thường không phải vấn đề)
2. Và mảng có **chu kỳ sống dài** này phình to vô hạn (ứng dụng liên tục chèn dữ liệu vào và không bao giờ dọn dẹp)

Chỉ khi **cả hai** điều kiện đều thỏa mãn thì mới xảy ra rò rỉ. Nếu thiếu một điều kiện hoặc chỉ thỏa mãn một điều kiện thì không phải rò rỉ.

## Mảng có chu kỳ sống dài

Trong webman, mảng có chu kỳ sống dài gồm:
1. Mảng dùng từ khóa `static`
2. Thuộc tính mảng của singleton
3. Mảng dùng từ khóa `global`

> **Lưu ý**
> Webman cho phép dùng dữ liệu có chu kỳ sống dài, nhưng cần đảm bảo dữ liệu có giới hạn và số phần tử không phình to vô hạn.

Sau đây là ví dụ cho từng trường hợp.

### Mảng static phình to vô hạn
```php
class Foo
{
    public static $data = [];
    public function index(Request $request)
    {
        self::$data[] = time();
        return response('hello');
    }
}
```

Mảng `$data` định nghĩa bằng từ khóa `static` có chu kỳ sống dài. Trong ví dụ, `$data` tiếp tục phình to theo mỗi yêu cầu, gây rò rỉ bộ nhớ.

### Thuộc tính mảng singleton phình to vô hạn
```php
class Cache
{
    protected static $instance;
    public $data = [];
    
    public function instance()
    {
        if (!self::$instance) {
            self::$instance = new self;
        }
        return self::$instance;
    }
    
    public function set($key, $value)
    {
        $this->data[$key] = $value;
    }
}
```

Mã gọi
```php
class Foo
{
    public function index(Request $request)
    {
        Cache::instance()->set(time(), time());
        return response('hello');
    }
}
```

`Cache::instance()` trả về singleton Cache có chu kỳ sống dài. Dù thuộc tính `$data` không dùng từ khóa `static`, do bản thân lớp có chu kỳ sống dài nên `$data` cũng là mảng có chu kỳ sống dài. Khi liên tục thêm dữ liệu với các khóa khác nhau vào `$data`, mức sử dụng bộ nhớ của chương trình tăng lên và xảy ra rò rỉ.

> **Lưu ý**
> Nếu số khóa thêm qua `Cache::instance()->set(key, value)` có hạn thì không xảy ra rò rỉ vì mảng `$data` không phình to vô hạn.

### Mảng global phình to vô hạn
```php
class Index
{
    public function index(Request $request)
    {
        global $data;
        $data[] = time();
        return response($foo->sayHello());
    }
}
```
Mảng định nghĩa bằng từ khóa `global` không được thu hồi sau khi hàm hoặc phương thức kết thúc, nên có chu kỳ sống dài. Đoạn mã trên sẽ gây rò rỉ khi số yêu cầu tăng dần. Tương tự, mảng định nghĩa bằng từ khóa `static` bên trong hàm hoặc phương thức cũng có chu kỳ sống dài; nếu mảng phình to vô hạn thì cũng gây rò rỉ, ví dụ:
```php
class Index
{
    public function index(Request $request)
    {
        static $data = [];
        $data[] = time();
        return response($foo->sayHello());
    }
}
```

## Khuyến nghị
Nên không tập trung quá mức vào rò rỉ bộ nhớ vì chúng rất hiếm. Khi xảy ra, có thể dùng kiểm tra tải để tìm đoạn mã gây rò rỉ. Dù nhà phát triển không tìm được vị trí rò rỉ, dịch vụ monitor tích hợp của webman vẫn sẽ khởi động lại tiến trình bị ảnh hưởng đúng lúc để giải phóng bộ nhớ.

Nếu muốn hạn chế tối đa rò rỉ bộ nhớ, có thể tham khảo các khuyến nghị sau:
1. Cố gắng không dùng mảng với từ khóa `global` hoặc `static`; nếu dùng thì đảm bảo chúng không phình to vô hạn.
2. Với lớp không quen thuộc, ưu tiên khởi tạo bằng từ khóa `new` thay vì singleton. Khi dùng singleton thì kiểm tra xem nó có thuộc tính mảng có thể phình to vô hạn không.
