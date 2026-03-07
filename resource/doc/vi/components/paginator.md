# Thành phần phân trang

## Địa chỉ dự án

https://github.com/jasongrimes/php-paginator
  
## Cài đặt

```php
composer require "jasongrimes/paginator:^1.0.3"
```
  
## Sử dụng

Tạo mới `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;
use JasonGrimes\Paginator;

class UserController
{
    /**
     * Danh sách người dùng
     */
    public function get(Request $request)
    {
        $total_items = 1000;
        $items_perPage = 50;
        $current_page = (int)$request->get('page', 1);
        $url_pattern = '/user/get?page=(:num)';
        $paginator = new Paginator($total_items, $items_perPage, $current_page, $url_pattern);
        return view('user/get', ['paginator' => $paginator]);
    }
    
}
```
**Mẫu (PHP thuần)**
Tạo mới mẫu `app/view/user/get.html`
```html
<html>
<head>
  <!-- Hỗ trợ sẵn có cho kiểu dáng phân trang Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**Mẫu (Twig)**
Tạo mới mẫu `app/view/user/get.html`
```html
<html>
<head>
  <!-- Hỗ trợ sẵn có cho kiểu dáng phân trang Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**Mẫu (Blade)**
Tạo mới mẫu `app/view/user/get.blade.php`
```html
<html>
<head>
  <!-- Hỗ trợ sẵn có cho kiểu dáng phân trang Bootstrap -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**Mẫu (ThinkPHP)**
Tạo mới mẫu `app/view/user/get.blade.php`
```html
<html>
<head>
    <!-- Hỗ trợ sẵn có cho kiểu dáng phân trang Bootstrap -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

Kết quả như sau:
![](../../assets/img/paginator.png)
  
## Thêm thông tin

Truy cập https://github.com/jasongrimes/php-paginator
  
