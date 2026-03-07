# Кэш

[webman/cache](https://github.com/webman-php/cache) — компонент кэширования на основе [symfony/cache](https://github.com/symfony/cache), совместимый с корутиновыми и некорутиновыми окружениями, с поддержкой пула соединений.

## Установка

```php
composer require -W webman/cache
```

## Пример
```php
<?php
namespace app\controller;

use support\Request;
use support\Cache;

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

## Расположение файла конфигурации
Файл конфигурации находится в `config/cache.php`. Создайте его вручную, если он отсутствует.

## Содержимое файла конфигурации
```php
<?php
return [
    'default' => 'file',
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default'
        ],
        'array' => [
            'driver' => 'array'
        ],
        'apcu' => [
            'driver' => 'apcu'
        ]
    ]
];
```
`stores.driver` поддерживает четыре драйвера: **file**, **redis**, **array** и **apcu**.

#### Драйвер file
Драйвер по умолчанию. Без внешних зависимостей. Поддерживает совместное использование кэша между процессами. Не поддерживает совместное использование между несколькими серверами.

#### Драйвер array
Хранение в памяти с наилучшей производительностью, но расходом памяти. Не поддерживает совместное использование между процессами и серверами. Данные теряются при перезапуске процесса. Подходит для проектов с малым объёмом кэша.

#### Драйвер apcu
Хранение в памяти. Производительность уступает только array. Поддерживает совместное использование кэша между процессами. Не поддерживает совместное использование между несколькими серверами. Данные теряются при перезапуске процесса. Подходит для проектов с малым объёмом кэша.

> Требуется установка и включение [расширения APCu](https://pecl.php.net/package/APCu). Не рекомендуется для сценариев с частой записью/удалением кэша, так как может привести к заметному снижению производительности.

#### Драйвер redis
Зависит от компонента [webman/redis](./redis.md). Поддерживает совместное использование кэша между процессами и серверами.

**stores.redis.connection**

`stores.redis.connection` соответствует ключу, определённому в `config/redis.php`. При использовании Redis повторно используется конфигурация `webman/redis`, включая настройки пула соединений.

**Рекомендуется добавить отдельную конфигурацию Redis для кэша в `config/redis.php`, например:**

```php
<?php
return [
    'default' => [
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 0,
    ],
    'cache' => [ // <==== Добавить
        'password' => 'abc123',
        'host' => '127.0.0.1',
        'port' => 6379,
        'database' => 1,
        'prefix' => 'webman_cache-',
    ]
];
```

Затем установите `stores.redis.connection` в `cache`. Итоговый `config/cache.php` будет выглядеть так:

```php
<?php
return [
    'default' => 'redis', // <====
    'stores' => [
        'file' => [
            'driver' => 'file',
            'path' => runtime_path('cache')
        ],
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache' // <====
        ],
        'array' => [
            'driver' => 'array'
        ]
    ]
];
```

## Переключение хранилищ
Можно вручную переключать хранилища для использования разных драйверов, например:

```php
Cache::store('redis')->set('key', 'value');
Cache::store('array')->set('key', 'value');
```

> **Подсказка**
> Имена ключей кэша ограничены [PSR-6](https://www.php-fig.org/psr/psr-6/#definitions) и не должны содержать ни один из символов `{}()/\@:`. Начиная с `symfony/cache` 7.2.4 эту проверку можно обойти, задав опцию PHP ini `zend.assertions=-1`.

## Использование других компонентов кэширования

См. [Другие базы данных](others.md#ThinkCache) для компонента [ThinkCache](https://github.com/webman-php/think-cache).
