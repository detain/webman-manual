# Компонент капчи

Ссылка на проект https://github.com/webman-php/captcha

## Установка
```
composer require webman/captcha
```

## Использование

**Создать файл `app/controller/LoginController.php`**

```php
<?php
namespace app\controller;

use support\Request;
use Webman\Captcha\CaptchaBuilder;

class LoginController
{
    /**
     * Тестовая страница
     */
    public function index(Request $request)
    {
        return view('login/index');
    }

    /**
     * Вывод изображения капчи
     */
    public function captcha(Request $request)
    {
        // Инициализация класса капчи
        $builder = new CaptchaBuilder;
        // Генерация капчи
        $builder->build();
        // Сохранение значения капчи в сессию
        $request->session()->set('captcha', strtolower($builder->getPhrase()));
        // Получение двоичных данных изображения капчи
        $img_content = $builder->get();
        // Вывод двоичных данных капчи
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }

    /**
     * Проверка капчи
     */
    public function check(Request $request)
    {
        // Получение поля captcha из POST-запроса
        $captcha = $request->post('captcha');
        // Сравнение с значением капчи из сессии
        if (strtolower($captcha) !== $request->session()->get('captcha')) {
            return json(['code' => 400, 'msg' => 'Введённая капча неверна']);
        }
        return json(['code' => 0, 'msg' => 'ok']);
    }

}
```

**Создать файл шаблона `app/view/login/index.html`**

```html
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Тест капчи</title>  
</head>
<body>
    <form method="post" action="/login/check">
       <img src="/login/captcha" /><br>
        <input type="text" name="captcha" />
        <input type="submit" value="Отправить" />
    </form>
</body>
</html>
```

Перейдите на страницу `http://127.0.0.1:8787/login`, интерфейс будет похож на следующий:
  ![](../../assets/img/captcha.png)

## Настройка общих параметров
```php
    /**
     * Вывод изображения капчи
     */
    public function captcha(Request $request)
    {
        $builder = new PhraseBuilder(4, 'abcdefghjkmnpqrstuvwxyzABCDEFGHJKMNPQRSTUVWXYZ');
        $captcha = new CaptchaBuilder(null, $builder);
        $captcha->build();
        $request->session()->set('join', strtolower($captcha->getPhrase()));
        $img_content = $captcha->get();
        return response($img_content, 200, ['Content-Type' => 'image/jpeg']);
    }
```

Подробнее об интерфейсах и параметрах см. https://github.com/webman-php/captcha
