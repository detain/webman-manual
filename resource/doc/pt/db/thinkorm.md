# think-orm

[webman/think-orm](https://github.com/webman-php/think-orm) é um componente de banco de dados baseado em [top-think/think-orm](https://github.com/top-think/think-orm). Suporta pool de conexões e funciona em ambientes coroutine e não-coroutine.

## Instalação

`composer require -W webman/think-orm`

É necessário reiniciar (restart) após a instalação (reload não tem efeito).

## Arquivo de configuração

Modifique o arquivo de configuração `config/think-orm.php` conforme suas necessidades.

## Documentação

https://www.kancloud.cn/manual/think-orm

## Utilização

```php
<?php
namespace app\controller;

use support\Request;
use support\think\Db;

class FooController
{
    public function get(Request $request)
    {
        $user = Db::table('user')->where('uid', '>', 1)->find();
        return json($user);
    }
}
```

## Criação de modelos

Os modelos think-orm estendem `support\think\Model`, como mostrado abaixo:

```
<?php
namespace app\model;

use support\think\Model;

class User extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'user';

    /**
     * The primary key associated with the table.
     *
     * @var string
     */
    protected $pk = 'id';

}
```

Você também pode criar modelos think-orm com o seguinte comando:

```
php webman make:model nome_tabela
```

> **Dica**
> Este comando requer `webman/console`. Instale-o com `composer require webman/console ^1.2.13`.

> **Nota**
> Se `make:model` detectar que o projeto principal usa `illuminate/database`, criará arquivos de modelo baseados em Illuminate em vez de think-orm. Nesse caso, adicione o parâmetro `tp`: `php webman make:model nome_tabela tp` (atualize o `webman/console` se não funcionar).
