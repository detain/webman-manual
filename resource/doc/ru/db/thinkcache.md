# think-cache

think-cache — компонент, извлечённый из фреймворка thinkphp, с добавленной поддержкой пула соединений. Автоматически поддерживает как корутинное, так и некорутинное окружение.

## Установка
`composer require -W webman/think-cache`

После установки требуется перезапуск (restart) (reload неэффективен)

### Файл конфигурации

Файл конфигурации — `config/think-cache.php`

## Использование

  ```php
  <?php
  namespace app\controller;
    
  use support\Request;
  use support\think\Cache;
  
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
## Доступные методы
```php
// Установить кэш
Cache::set('val','value',600);
// Проверить наличие кэша
Cache::has('val');
// Получить кэш
Cache::get('val');
// Удалить кэш
Cache::delete('val');
// Очистить кэш
Cache::clear();
// Прочитать и удалить кэш
Cache::pull('val');
// Записать, если не существует
Cache::remember('val',10);

// Для числовых данных в кэше
// Увеличить кэш на 1
Cache::inc('val');
// Увеличить кэш на 5
Cache::inc('val',5);
// Уменьшить кэш на 1
Cache::dec('val');
// Уменьшить кэш на 5
Cache::dec('val',5);

// Использование тегов кэша
Cache::tag('tag_name')->set('val','value',600);
// Удалить кэш по тегу
Cache::tag('tag_name')->clear();
// Поддержка нескольких тегов
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Удалить кэш по нескольким тегам
Cache::tag(['tag1','tag2'])->clear();

// Использование различных хранилищ кэша
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


