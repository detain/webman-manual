# База данных

Поскольку большинство плагинов устанавливают [webman-admin](https://www.workerman.net/plugin/82), рекомендуется напрямую повторно использовать конфигурацию базы данных `webman-admin`.

Модели, базовый класс которых — `plugin\admin\app\model\Base`, автоматически используют конфигурацию базы данных webman-admin.
```php
<?php

namespace plugin\foo\app\model;

use plugin\admin\app\model\Base;

class Orders extends Base
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'foo_orders';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $primaryKey = 'id';
    
}
```

К базе данных webman-admin также можно обратиться через `plugin.admin.mysql`, например:

```php
Db::connection('plugin.admin.mysql')->table('user')->first();
```


## Использование собственной базы данных

Плагины также могут выбрать использование собственной базы данных. Например, содержимое `plugin/foo/config/database.php`:

```php
return  [
    'default' => 'mysql',
    'connections' => [
        'mysql' => [ // mysql — имя подключения
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'база_данных',
            'username'    => 'пользователь',
            'password'    => 'пароль',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
        'admin' => [ // admin — имя подключения
            'driver'      => 'mysql',
            'host'        => '127.0.0.1',
            'port'        => 3306,
            'database'    => 'база_данных',
            'username'    => 'пользователь',
            'password'    => 'пароль',
            'charset'     => 'utf8mb4',
            'collation'   => 'utf8mb4_general_ci',
        ],
    ],
];
```

Формат обращения: `Db::connection('plugin.{плагин}.{имя_подключения}');`, например:

```php
use support\Db;
Db::connection('plugin.foo.mysql')->table('user')->first();
Db::connection('plugin.foo.admin')->table('admin')->first();
```

Для использования базы данных основного проекта вызывайте её напрямую:

```php
use support\Db;
Db::table('user')->first();
// Предполагается, что в основном проекте также настроено подключение admin
Db::connection('admin')->table('admin')->first();
```

#### Настройка базы данных для Model

Можно создать базовый класс Base для Model и задать свойство `$connection` для использования подключения к базе данных самого плагина:

```php
<?php

namespace plugin\foo\app\model;

use DateTimeInterface;
use support\Model;

class Base extends Model
{
    /**
     * @var string
     */
    protected $connection = 'plugin.foo.mysql';

}
```

Все модели в плагине, наследующие Base, будут автоматически использовать собственную базу данных плагина.

## Автоматический импорт базы данных

Выполнение `php webman app-plugin:create foo` создаёт плагин foo, включая `plugin/foo/api/Install.php` и `plugin/foo/install.sql`.

> **Подсказка**
> Если файл install.sql не генерируется, попробуйте обновить `webman/console`: `composer require webman/console ^1.3.6`

#### Импорт базы данных при установке плагина

При установке плагина выполняется метод `install` в Install.php, который автоматически выполняет SQL-инструкции из `install.sql`, импортируя таким образом таблицы базы данных.

Содержимое `install.sql` должно включать создание таблиц и исторические изменения схемы. Каждая инструкция должна заканчиваться на `;`, например:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Первичный ключ',
  `order_id` varchar(50) NOT NULL COMMENT 'ID заказа',
  `user_id` int NOT NULL COMMENT 'ID пользователя',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Сумма к оплате',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Заказы';

CREATE TABLE `foo_goods` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Первичный ключ',
  `name` varchar(50) NOT NULL COMMENT 'Название',
  `price` int NOT NULL COMMENT 'Цена',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Товары';
```

**Изменение подключения к базе данных**

По умолчанию `install.sql` импортируется в базу данных webman-admin. Для импорта в другую базу данных измените свойство `$connection` в `Install.php`:

```php
<?php

class Install
{
    // Указать собственную базу данных плагина
    protected static $connection = 'plugin.admin.mysql';
    
    // ...
}
```

**Проверка**

Выполните `php webman app-plugin:install foo` для установки плагина. Затем проверьте базу данных — таблицы `foo_orders` и `foo_goods` должны быть созданы.

#### Изменение структуры таблиц при обновлении плагина

Когда обновление плагина требует новых таблиц или изменений схемы, добавьте соответствующие SQL-инструкции в конец `install.sql`. Каждая инструкция должна заканчиваться на `;`. Например, добавление таблицы `foo_user` и столбца `status` в `foo_orders`:

```sql
CREATE TABLE `foo_orders` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'Первичный ключ',
  `order_id` varchar(50) NOT NULL COMMENT 'ID заказа',
  `user_id` int NOT NULL COMMENT 'ID пользователя',
  `total_amount` decimal(10,2) NOT NULL COMMENT 'Сумма к оплате',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Заказы';

CREATE TABLE `foo_goods` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Первичный ключ',
 `name` varchar(50) NOT NULL COMMENT 'Название',
 `price` int NOT NULL COMMENT 'Цена',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Товары';


CREATE TABLE `foo_user` (
 `id` int NOT NULL AUTO_INCREMENT COMMENT 'Первичный ключ',
 `name` varchar(50) NOT NULL COMMENT 'Имя'
 PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT='Пользователь';

ALTER TABLE `foo_orders` ADD `status` tinyint NOT NULL DEFAULT 0 COMMENT 'Статус';
```

При обновлении метод `update` в Install.php выполняет инструкции из `install.sql`. Новые выполняются; уже применённые пропускаются, что корректно применяет изменения базы данных при обновлениях.

#### Удаление базы данных при деинсталляции плагина

При деинсталляции плагина вызывается метод `uninstall` в Install.php. Он автоматически анализирует инструкции CREATE TABLE в `install.sql` и удаляет эти таблицы.

Если нужно выполнить только свой `uninstall.sql` вместо автоматического удаления таблиц, создайте `plugin/{имя_плагина}/uninstall.sql`. В этом случае метод `uninstall` выполнит только инструкции из этого файла.
