# পেজিনেশন কম্পোনেন্ট

## প্রকল্পের ঠিকানা

https://github.com/jasongrimes/php-paginator
  
## ইনস্টলেশন

```php
composer require "jasongrimes/paginator:^1.0.3"
```
  
## ব্যবহার

নতুন তৈরি করুন `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;
use JasonGrimes\Paginator;

class UserController
{
    /**
     * ব্যবহারকারীর তালিকা
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
**টেম্পলেট (নেটিভ PHP)**
নতুন টেম্পলেট তৈরি করুন `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap পেজিনেশন স্টাইলের অন্তর্নির্মিত সমর্থন -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**টেম্পলেট (Twig)**
নতুন টেম্পলেট তৈরি করুন `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap পেজিনেশন স্টাইলের অন্তর্নির্মিত সমর্থন -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**টেম্পলেট (Blade)**
নতুন টেম্পলেট তৈরি করুন `app/view/user/get.blade.php`
```html
<html>
<head>
  <!-- Bootstrap পেজিনেশন স্টাইলের অন্তর্নির্মিত সমর্থন -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**টেম্পলেট (ThinkPHP)**
নতুন টেম্পলেট তৈরি করুন `app/view/user/get.blade.php`
```html
<html>
<head>
    <!-- Bootstrap পেজিনেশন স্টাইলের অন্তর্নির্মিত সমর্থন -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

ফলাফল নিম্নরূপ:
![](../../assets/img/paginator.png)
  
## আরও তথ্য

পরিদর্শন করুন https://github.com/jasongrimes/php-paginator
  
