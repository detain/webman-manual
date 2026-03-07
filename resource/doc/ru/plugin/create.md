# Базовый процесс создания и публикации плагина

## Принцип
1. На примере cross-origin плагина: плагин состоит из трёх частей — файл middleware cross-origin, файл конфигурации `middleware.php` и автоматически создаваемый командой `Install.php`.
2. Команда используется для упаковки этих трёх файлов и публикации через Composer.
3. При установке плагина cross-origin через Composer, `Install.php` копирует файл middleware и конфигурацию в `{основной проект}/config/plugin` для загрузки webman, включая таким образом cross-origin.
4. При удалении плагина через Composer, `Install.php` удаляет соответствующие файлы middleware и конфигурации, выполняя автоматическую деинсталляцию плагина.

## Спецификация
1. Имя плагина состоит из `vendor` и `имени плагина`, напр. `webman/push`, что соответствует имени пакета Composer.
2. Файлы конфигурации плагина размещаются в `config/plugin/vendor/имя плагина/` (команда консоли создаёт каталог конфигурации автоматически). Если плагину не нужна конфигурация, созданный каталог следует удалить.
3. Каталог конфигурации плагина поддерживает только: `app.php` (основная конфигурация), `bootstrap.php` (запуск процессов), `route.php` (маршруты), `middleware.php` (middleware), `process.php` (пользовательские процессы), `database.php` (БД), `redis.php` (Redis), `thinkorm.php` (thinkorm). Они распознаются webman автоматически.
4. Доступ к конфигурации: `config('plugin.vendor.имя плагина.файл конфига.конкретный пункт');`, напр. `config('plugin.webman.push.app.app_key')`.
5. При собственной конфигурации БД: для `illuminate/database` использовать `Db::connection('plugin.vendor.имя плагина.подключение')`, для `thinkorm` — `Db::connect('plugin.vendor.имя плагина.подключение')`.
6. При размещении бизнес-файлов в `app/` плагин не должен конфликтовать с основным проектом и другими плагинами.
7. Плагинам следует избегать копирования файлов или каталогов в основной проект. Напр., для cross-origin плагина копируется только конфигурация; файлы middleware остаются в `vendor/webman/cros/src`.
8. Для пространств имён плагинов рекомендуется PascalCase, напр. `Webman/Console`.

## Пример

**Установить командную строку `webman/console`**

`composer require webman/console`

### Создание плагина

Предположим, создаётся плагин с именем `foo/admin` (имя проекта Composer для публикации, в нижнем регистре). Выполнить:

`php webman plugin:create --name=foo/admin`

Создаются `vendor/foo/admin` (файлы плагина) и `config/plugin/foo/admin` (конфигурация).

> Примечание
> `config/plugin/foo/admin` поддерживает: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Формат как у webman, автоматическое слияние.
> Доступ с префиксом `plugin`, напр. `config('plugin.foo.admin.app')`.


### Экспорт плагина

После завершения разработки выполнить:

`php webman plugin:export --name=foo/admin`

Экспорт

> Пояснение
> При экспорте `config/plugin/foo/admin` копируется в `vendor/foo/admin/src` и создаётся `Install.php`, выполняемый при установке и удалении.
> Установка по умолчанию: копирует конфигурацию из `vendor/foo/admin/src` в `config/plugin` проекта.
> Удаление по умолчанию: удаляет файлы конфигурации плагина из `config/plugin` проекта.
> Можно изменить `Install.php` для добавления своей логики при установке и удалении.

### Публикация плагина
* Предполагается, что есть аккаунты [GitHub](https://github.com) и [Packagist](https://packagist.org).
* Создать репозиторий `admin` на [GitHub](https://github.com) и загрузить код, напр. `https://github.com/ваш-username/admin`.
* Перейти на `https://github.com/ваш-username/admin/releases/new` и создать релиз, напр. `v1.0.0`.
* На [Packagist](https://packagist.org) нажать `Submit` и указать `https://github.com/ваш-username/admin` для публикации плагина.

> **Подсказка**
> При конфликте имён на Packagist выбрать другое имя vendor, напр. изменить `foo/admin` на `myfoo/admin`.

При обновлении: загрузить код на GitHub, создать новый релиз на `https://github.com/ваш-username/admin/releases/new`, затем нажать `Update` на `https://packagist.org/packages/foo/admin`.

## Добавление команд к плагину
Некоторым плагинам нужны свои команды. Напр., при установке `webman/redis-queue` проект получает команду `redis-queue:consumer`. Команда `php webman redis-queue:consumer send-mail` быстро генерирует класс consumer `SendMail.php`, что ускоряет разработку.

Чтобы добавить команду `foo-admin:add` к плагину `foo/admin`:

### Создание команды

**Создать файл `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Описание команды';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Примечание**
> Чтобы избежать конфликтов команд между плагинами, использовать формат `vendor-plugin:команда`. Напр., все команды плагина `foo/admin` должны иметь префикс `foo-admin:`, например `foo-admin:add`.

### Добавление конфигурации
**Создать `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Добавить другие при необходимости...
];
```

> **Подсказка**
> `command.php` регистрирует пользовательские команды плагина. Каждая запись — класс команды. `webman/console` загружает их автоматически. См. [Команды консоли](console.md).

### Выполнение экспорта
Выполнить `php webman plugin:export --name=foo/admin` для экспорта и публикации на Packagist. После установки `foo/admin` будет доступна команда `foo-admin:add`. Команда `php webman foo-admin:add jerry` выведет `Admin add jerry`.
