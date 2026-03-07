# Управление сеансами webman

## Пример
```php
<?php
namespace app\controller;

use support\Request;

class UserController
{
    public function hello(Request $request)
    {
        $name = $request->get('name');
        $session = $request->session();
        $session->set('name', $name);
        return response('hello ' . $session->get('name'));
    }
}
```

Получите экземпляр `Workerman\Protocols\Http\Session` через `$request->session();` и используйте методы экземпляра для добавления, изменения и удаления данных сеанса.

> **Примечание**
> Данные сеанса автоматически сохраняются при уничтожении объекта сеанса.
> Если сохранить объект сеанса в глобальную переменную, он не будет уничтожен и не сохранится автоматически. В этом случае необходимо вручную вызвать `$session->save()` для сохранения данных.

## Получение всех данных сеанса
```php
$session = $request->session();
$all = $session->all();
```
Возвращается массив. Если данных сеанса нет, возвращается пустой массив.


## Получение значения из сеанса
```php
$session = $request->session();
$name = $session->get('name');
```
Если данные не существуют, возвращается null.

Можно передать значение по умолчанию вторым аргументом метода `get`. Если соответствующее значение в массиве сеанса не найдено, возвращается значение по умолчанию. Например:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```


## Сохранение данных сеанса
Для сохранения одного значения используйте метод `set`.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Метод `set` ничего не возвращает. Данные сеанса автоматически сохраняются при уничтожении объекта сеанса.

Для сохранения нескольких значений используйте метод `put`.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Метод `put` также ничего не возвращает.

## Удаление данных сеанса
Для удаления одного или нескольких значений сеанса используйте метод `forget`.
```php
$session = $request->session();
// Удалить одно значение
$session->forget('name');
// Удалить несколько значений
$session->forget(['name', 'age']);
```

Также доступен метод `delete`, который в отличие от `forget` может удалить только одно значение.
```php
$session = $request->session();
// Эквивалентно $session->forget('name');
$session->delete('name');
```


## Получение и удаление значения из сеанса
```php
$session = $request->session();
$name = $session->pull('name');
```
Это эквивалентно следующему коду:
```php
$session = $request->session();
$value = $session->get('name');
$session->delete('name');
```
Если соответствующее значение сеанса не существует, возвращается null.


## Удаление всех данных сеанса
```php
$request->session()->flush();
```
Метод ничего не возвращает. Данные сеанса автоматически удаляются из хранилища при уничтожении объекта сеанса.


## Проверка существования значения в сеансе
```php
$session = $request->session();
$has = $session->has('name');
```
Возвращает false, если значение сеанса не существует или равно null; в противном случае возвращает true.

```php
$session = $request->session();
$has = $session->exists('name');
```
Этот код также проверяет существование значения в сеансе. Отличие: `exists` возвращает true даже когда значение равно null.

## Функция-помощник session()

webman предоставляет функцию-помощник `session()` для тех же операций.
```php
// Получение экземпляра сеанса
$session = session();
// Эквивалентно
$session = $request->session();

// Получение значения
$value = session('key', 'default');
// Эквивалентно
$value = session()->get('key', 'default');
// Эквивалентно
$value = $request->session()->get('key', 'default');

// Присвоение значений сеансу
session(['key1'=>'value1', 'key2' => 'value2']);
// Эквивалентно
session()->put(['key1'=>'value1', 'key2' => 'value2']);
// Эквивалентно
$request->session()->put(['key1'=>'value1', 'key2' => 'value2']);

```


## Файл конфигурации
Файл конфигурации сеанса находится в `config/session.php`. Содержимое примерно такое:
```php
use Webman\Session\FileSessionHandler;
use Webman\Session\RedisSessionHandler;
use Webman\Session\RedisClusterSessionHandler;

return [
    // FileSessionHandler::class или RedisSessionHandler::class или RedisClusterSessionHandler::class 
    'handler' => FileSessionHandler::class,
    
    // Для handler FileSessionHandler::class значение 'file',
    // для RedisSessionHandler::class — 'redis',
    // для RedisClusterSessionHandler::class — 'redis_cluster' (кластер Redis)
    'type'    => 'file',

    // Разные обработчики используют разные настройки
    'config' => [
        // Настройки для type 'file'
        'file' => [
            'save_path' => runtime_path() . '/sessions',
        ],
        // Настройки для type 'redis'
        'redis' => [
            'host'      => '127.0.0.1',
            'port'      => 6379,
            'auth'      => '',
            'timeout'   => 2,
            'database'  => '',
            'prefix'    => 'redis_session_',
        ],
        'redis_cluster' => [
            'host'    => ['127.0.0.1:7000', '127.0.0.1:7001', '127.0.0.1:7001'],
            'timeout' => 2,
            'auth'    => '',
            'prefix'  => 'redis_session_',
        ]
        
    ],

    'session_name' => 'PHPSID', // Имя cookie для хранения session_id
    'auto_update_timestamp' => false,  // Автоматическое обновление сеанса, по умолчанию выключено
    'lifetime' => 7*24*60*60,          // Время жизни сеанса
    'cookie_lifetime' => 365*24*60*60, // Время жизни cookie для session_id
    'cookie_path' => '/',              // Путь cookie для session_id
    'domain' => '',                    // Домен cookie для session_id
    'http_only' => true,               // Включение httpOnly, по умолчанию включено
    'secure' => false,                 // Сеанс только по HTTPS, по умолчанию выключено
    'same_site' => '',                 // Защита от CSRF и отслеживания, значения: strict/lax/none
    'gc_probability' => [1, 1000],     // Вероятность очистки сеанса
];
```

## Безопасность
Не рекомендуется сохранять в сеансе экземпляры классов напрямую, особенно из ненадёжных источников. Десериализация может создать риски безопасности.

