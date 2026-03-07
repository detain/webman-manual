# ページネーションコンポーネント

## プロジェクトのURL

https://github.com/jasongrimes/php-paginator
  
## インストール

```php
composer require "jasongrimes/paginator:^1.0.3"
```
  
## 使用方法

新規作成 `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;
use JasonGrimes\Paginator;

class UserController
{
    /**
     * ユーザー一覧
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
**テンプレート（ネイティブPHP）**
新規テンプレート `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrapページネーションスタイルの組み込みサポート -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**テンプレート（Twig）**
新規テンプレート `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrapページネーションスタイルの組み込みサポート -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**テンプレート（Blade）**
新規テンプレート `app/view/user/get.blade.php`
```html
<html>
<head>
  <!-- Bootstrapページネーションスタイルの組み込みサポート -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**テンプレート（ThinkPHP）**
新規テンプレート `app/view/user/get.blade.php`
```html
<html>
<head>
    <!-- Bootstrapページネーションスタイルの組み込みサポート -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

表示結果は以下のとおりです：
![](../../assets/img/paginator.png)
  
## 詳細情報

https://github.com/jasongrimes/php-paginator をご覧ください
  
