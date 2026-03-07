# पेजिनेशन कंपोनेंट

## प्रोजेक्ट पता

https://github.com/jasongrimes/php-paginator
  
## स्थापना

```php
composer require "jasongrimes/paginator:^1.0.3"
```
  
## उपयोग

नया बनाएं `app/controller/UserController.php`
```php
<?php
namespace app\controller;

use support\Request;
use JasonGrimes\Paginator;

class UserController
{
    /**
     * उपयोगकर्ता सूची
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
**टेम्पलेट (मूल PHP)**
नया टेम्पलेट बनाएं `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap पेजिनेशन स्टाइल के लिए अंतर्निहित समर्थन -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?= $paginator;?>

</body>
</html>
```

**टेम्पलेट (Twig)**
नया टेम्पलेट बनाएं `app/view/user/get.html`
```html
<html>
<head>
  <!-- Bootstrap पेजिनेशन स्टाइल के लिए अंतर्निहित समर्थन -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{% autoescape false %}
{{paginator}}
{% endautoescape %}

</body>
</html>
```

**टेम्पलेट (Blade)**
नया टेम्पलेट बनाएं `app/view/user/get.blade.php`
```html
<html>
<head>
  <!-- Bootstrap पेजिनेशन स्टाइल के लिए अंतर्निहित समर्थन -->
  <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

{!! $paginator !!}

</body>
</html>
```

**टेम्पलेट (ThinkPHP)**
नया टेम्पलेट बनाएं `app/view/user/get.blade.php`
```html
<html>
<head>
    <!-- Bootstrap पेजिनेशन स्टाइल के लिए अंतर्निहित समर्थन -->
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>

<?=$paginator?>

</body>
</html>
```

परिणाम निम्नानुसार है:
![](../../assets/img/paginator.png)
  
## अधिक जानकारी

https://github.com/jasongrimes/php-paginator पर जाएं
  
