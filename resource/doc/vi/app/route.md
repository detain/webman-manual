# Tệp cấu hình định tuyến
Tệp cấu hình định tuyến của plugin nằm tại `plugin/tên_plugin/config/route.php`.

## Định tuyến mặc định
Tất cả đường dẫn URL của ứng dụng plugin đều bắt đầu bằng `/app`, ví dụ URL của `plugin\foo\app\controller\UserController` là `http://127.0.0.1:8787/app/foo/user`.

## Tắt định tuyến mặc định
Để tắt định tuyến mặc định của một ứng dụng plugin, thêm nội dung sau vào cấu hình định tuyến:
```php
Route::disableDefaultRoute('foo');
```

## Xử lý callback 404
Để thiết lập fallback cho một ứng dụng plugin, cần truyền tên plugin qua tham số thứ hai. Ví dụ:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
