# Автозагрузка

## Загрузка файлов PSR-0 через Composer
webman следует спецификации автозагрузки `PSR-4`. Если проекту нужно загружать библиотеки, совместимые с PSR-0, выполните следующие шаги:

- Создайте каталог `extend` для хранения библиотек PSR-0
- Отредактируйте `composer.json` и добавьте в `autoload` следующее:

```json
"psr-0" : {
    "": "extend/"
}
```
Итоговый результат будет выглядеть примерно так:
![](../../assets/img/psr0.png)

- Выполните `composer dumpautoload`
- Выполните `php start.php restart` для перезапуска webman (примечание: для применения изменений необходим полный перезапуск)

## Загрузка определённых файлов через Composer

- Отредактируйте `composer.json` и добавьте в `autoload.files` нужные файлы:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Выполните `composer dumpautoload`
- Выполните `php start.php restart` для перезапуска webman (примечание: для применения изменений необходим полный перезапуск)

> **Примечание**
> Файлы, настроенные в `autoload.files` composer.json, загружаются до запуска webman. Файлы, загружаемые через `config/autoload.php` фреймворка, загружаются после запуска webman.
> Изменения в файлах из `autoload.files` composer.json вступают в силу только после полного перезапуска (restart); reload недостаточен. Файлы из `config/autoload.php` фреймворка поддерживают горячую перезагрузку; изменения применяются при reload.

## Загрузка определённых файлов через фреймворк
Некоторые файлы могут не соответствовать спецификации PSR и не загружаются автоматически. Их можно загрузить, настроив `config/autoload.php`, например:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Примечание**
 > В `autoload.php` задана загрузка `support/Request.php` и `support/Response.php`, поскольку такие же файлы есть в `vendor/workerman/webman-framework/src/support/`. Через `autoload.php` загружаются версии из корня проекта, что позволяет изменять эти файлы без правки файлов в `vendor`. Если не нужно их менять, эти две записи можно опустить.
