# Bộ lập lịch crontab

## Mô tả

`workerman/crontab` tương tự như crontab của Linux, khác biệt là hỗ trợ lập lịch theo giây.

Định dạng thời gian:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[Có thể bỏ qua, nếu không có vị trí 0 thì đơn vị nhỏ nhất là phút]
```

## Địa chỉ dự án

https://github.com/walkor/crontab
  
## Cài đặt
 
```php
composer require workerman/crontab
```
  
## Sử dụng

**Bước 1: Tạo tệp tiến trình `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // Thực hiện mỗi giây
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Thực hiện mỗi 5 giây
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Thực hiện mỗi phút
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Thực hiện mỗi 5 phút
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Thực hiện ở giây đầu tiên của mỗi phút
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Thực hiện lúc 7:50 hàng ngày (vị trí giây được bỏ qua ở đây)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Bước 2: Cấu hình tiến trình khởi động cùng webman**
  
Mở tệp cấu hình `config/process.php`, thêm cấu hình sau:

```php
return [
    ....Cấu hình khác, ở đây lược bỏ....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Bước 3: Khởi động lại webman**

> Chú ý: Công việc lập lịch không thực hiện ngay; tất cả bắt đầu đếm và thực hiện từ phút tiếp theo.

## Giải thích
Crontab không phải bất đồng bộ. Ví dụ: trong một tiến trình task đặt hai bộ đếm A và B, cả hai đều thực hiện mỗi giây, nhưng nhiệm vụ A mất 10 giây thì B phải chờ A hoàn thành mới chạy được, gây trì hoãn cho B.
Nếu logic nhạy với khoảng thời gian, cần chạy các công việc lập lịch nhạy cảm thời gian trong tiến trình riêng để tránh bị ảnh hưởng. Ví dụ cấu hình `config/process.php` như sau:

```php
return [
    ....Cấu hình khác, ở đây lược bỏ....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Đặt các công việc nhạy cảm thời gian vào `process/Task1.php`, các công việc khác vào `process/Task2.php`.

Chi tiết cấu hình `config/process.php`, xem [Tiến trình tùy chỉnh](../process.md).
