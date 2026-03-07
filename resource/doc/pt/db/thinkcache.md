# think-cache

think-cache é um componente extraído do framework thinkphp, com suporte a pool de conexões adicionado. Suporta automaticamente ambientes de corotina e não-corotina.

## Instalação
`composer require -W webman/think-cache`

É necessário reiniciar (restart) após a instalação (reload não tem efeito)

### Ficheiro de configuração

O ficheiro de configuração é `config/think-cache.php`

## Utilização

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
## API fornecida
```php
// Definir cache
Cache::set('val','value',600);
// Verificar se o cache existe
Cache::has('val');
// Obter cache
Cache::get('val');
// Eliminar cache
Cache::delete('val');
// Limpar cache
Cache::clear();
// Ler e eliminar cache
Cache::pull('val');
// Escrever se não existir
Cache::remember('val',10);

// Para dados de cache numéricos
// Incrementar cache em 1
Cache::inc('val');
// Incrementar cache em 5
Cache::inc('val',5);
// Decrementar cache em 1
Cache::dec('val');
// Decrementar cache em 5
Cache::dec('val',5);

// Usar etiquetas de cache
Cache::tag('tag_name')->set('val','value',600);
// Eliminar cache sob uma etiqueta
Cache::tag('tag_name')->clear();
// Suporta múltiplas etiquetas
Cache::tag(['tag1','tag2'])->set('val2','value',600);
// Eliminar cache sob múltiplas etiquetas
Cache::tag(['tag1','tag2'])->clear();

// Usar diferentes armazenamentos de cache
$redis = Cache::store('redis');

$redis->set('var','value',600);
$redis->get('var');
```


