# think-cache

think-cache là một thành phần được tách ra từ framework thinkphp, có thêm hỗ trợ connection pool. Tự động hỗ trợ cả môi trường coroutine và non-coroutine.

## Cài đặt
`composer require -W webman/think-cache`

Cần khởi động lại (restart) sau khi cài đặt (reload không có hiệu lực)

### File cấu hình

File cấu hình là `config/think-cache.php`

## Sử dụng

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
  class UserController
  {
      public function db(Request $request)
      {
          $key = 'test_key';
          Cache::set($key, rand());
          return response(Cache::get($key));
      }
  }
  ```
## API cung cấp
```php
// Thiết lập cache
Cache::set('val','value',600);
// Kiểm tra cache có tồn tại
Cache::has('val');
// Lấy cache
Cache::get('val');
// Xóa cache
Cache::delete('val');
// Xóa sạch cache
Cache::clear();
// Đọc và xóa cache
Cache::pull('val');
// Ghi nếu chưa tồn tại
Cache::remember('val',10);

// Cho dữ liệu cache dạng số
// Tăng cache thêm 1
Cache::inc('val');
// Tăng cache thêm 5
Cache::inc('val',5);
// Giảm cache đi 1
Cache::dec('val');
// Giảm cache đi 5
Cache::dec('val',5);

// Sử dụng tag cache
Cache::tag('tag_name')->set('val','value',600);
// Xóa cache dưới một tag
Cache::tag('tag_name')->clear();
// Hỗ trợ nhiều tag
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Xóa cache dưới nhiều tag
Cache::tag(['tag1','tag2'])->clear();

// Sử dụng nhiều loại cache store
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


