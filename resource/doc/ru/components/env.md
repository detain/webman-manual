# Компонент ENV vlucas/phpdotenv

## Описание
`vlucas/phpdotenv` — компонент для загрузки переменных окружения, позволяющий разделять конфигурации для разных окружений (разработка, тестирование и т.д.).

## Ссылка на проект

https://github.com/vlucas/phpdotenv
  
## Установка
 
```php
composer require vlucas/phpdotenv
 ```
  
## Использование

### Создайте файл `.env` в корне проекта
**.env**
```
DB_HOST = 127.0.0.1
DB_PORT = 3306
DB_NAME = test
DB_USER = foo
DB_PASSWORD = 123456
```

### Измените файл конфигурации
**config/database.php**
```php
return [
    // База данных по умолчанию
    'default' => 'mysql',

    // Конфигурации различных баз данных
    'connections' => [
        'mysql' => [
            'driver'      => 'mysql',
            'host'        => getenv('DB_HOST'),
            'port'        => getenv('DB_PORT'),
            'database'    => getenv('DB_NAME'),
            'username'    => getenv('DB_USER'),
            'password'    => getenv('DB_PASSWORD'),
            'unix_socket' => '',
            'charset'     => 'utf8',
            'collation'   => 'utf8_unicode_ci',
            'prefix'      => '',
            'strict'      => true,
            'engine'      => null,
        ],
    ],
];
```

> **Совет**
> Рекомендуется добавить файл `.env` в `.gitignore`, чтобы не заносить его в репозиторий. Добавьте в репозиторий файл-образец `.env.example`. При развёртывании скопируйте `.env.example` в `.env` и при необходимости измените настройки. Так проект будет загружать разные конфигурации в разных окружениях.

> **Важно**
> `vlucas/phpdotenv` может работать некорректно с PHP в режиме TS (Thread Safe). Используйте режим NTS (Non-Thread-Safe). Текущую версию PHP можно проверить командой `php -v`.

## Дополнительно

Посетите https://github.com/vlucas/phpdotenv
  
