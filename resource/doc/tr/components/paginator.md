# Sayfalama Bileşeni

## Proje Adresi

https://github.com/jasongrimes/php-paginator
  
## Kurulum

```php
composer require "jasongrimes/paginator:^1.0.3"
```
  
## Kullanım

Yeni oluştur `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;
use JasonGrimes\Paginator;

class UserController
{
    /**
     * Kullanıcı listesi
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
**Şablon (Yerel PHP)**
Yeni şablon oluştur `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap sayfalama stilleri için yerleşik destek -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**Şablon (Twig)**
Yeni şablon oluştur `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap sayfalama stilleri için yerleşik destek -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**Şablon (Blade)**
Yeni şablon oluştur `app/view/user/get.blade.php`
```html
<html>
<head>
  <!-- Bootstrap sayfalama stilleri için yerleşik destek -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**Şablon (ThinkPHP)**
Yeni şablon oluştur `app/view/user/get.blade.php`
```html
<html>
<head>
    <!-- Bootstrap sayfalama stilleri için yerleşik destek -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

Sonuç aşağıdaki gibidir:
![](../../assets/img/paginator.png)
  
## Daha Fazla Bilgi

https://github.com/jasongrimes/php-paginator adresini ziyaret edin
  
