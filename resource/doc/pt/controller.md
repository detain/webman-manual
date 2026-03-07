# Controlador

Crie um novo arquivo de controlador `app/controller/FooController.php`.

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Ao acessar `http://127.0.0.1:8787/foo`, a página retornará `hello index`.

Ao acessar `http://127.0.0.1:8787/foo/hello`, a página retornará `hello webman`.

Você pode alterar as regras de roteamento na configuração de rotas, consulte [Rotas](route.md).

> **Dica**
> Em caso de erro 404, abra `config/app.php`, defina `controller_suffix` como `Controller` e reinicie.

## Sufixo do controlador
A partir da versão 1.3, o webman permite definir o sufixo do controlador em `config/app.php`. Se `controller_suffix` estiver definido como vazio `''`, o controlador terá este formato:

`app\controller\Foo.php`.

```php
<?php
namespace app\controller;

use support\Request;

class Foo
{
    public function index(Request $request)
    {
        return response('hello index');
    }
    
    public function hello(Request $request)
    {
        return response('hello webman');
    }
}
```

Recomenda-se fortemente definir o sufixo do controlador como `Controller` para evitar conflitos com nomes de classes de modelo e aumentar a segurança.

## Explicação
- O framework passa automaticamente o objeto `support\Request` ao controlador, com o qual é possível obter os dados de entrada do utilizador (get, post, header, cookie, etc.), consulte [Pedido](request.md).
- O controlador pode retornar números, strings ou objetos `support\Response`, mas não outros tipos de dados.
- Os objetos `support\Response` podem ser criados com funções auxiliares como `response()`, `json()`, `xml()`, `jsonp()`, `redirect()`, etc.

## Vinculação de parâmetros do controlador

#### Exemplo
O webman suporta a vinculação automática dos parâmetros do pedido aos parâmetros dos métodos do controlador. Por exemplo:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

Pode passar os valores de `name` e `age` via `GET` ou `POST`, ou através dos parâmetros da rota. Por exemplo:

```php
Route::any('/user/{name}/{age}', [app\controller\UserController::class, 'create']);
```

A prioridade é: `parâmetros da rota` > `GET` > `POST`.

#### Valores predefinidos

Ao acessar `/user/create?name=tom`, obterá o seguinte erro:

```html
Missing input parameter age
```

O motivo é que não foi passado o parâmetro `age`. Pode resolver definindo um valor predefinido. Por exemplo:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age = 18): Response
    {
        return json(['name' => $name, 'age' => $age]);
    }
}
```

#### Tipos de parâmetros
Ao acessar `/user/create?name=tom&age=not_int`, obterá o seguinte erro:

> **Dica**
> Para facilitar os testes, passamos os parâmetros diretamente na barra de endereço. No desenvolvimento real, os parâmetros devem ser passados via `POST`.

```html
Input age must be of type int, string given
```

Os dados recebidos são convertidos conforme o tipo. Se a conversão falhar, é lançada a exceção `support\exception\InputTypeException`. Como `age` não pode ser convertido para `int`, aparece este erro.

#### Mensagens de erro personalizadas
Pode personalizar mensagens como `Missing input parameter age` e `Input age must be of type int, string given` através da tradução. Consulte os seguintes comandos:

```
composer require symfony/translation
mkdir resource/translations/zh_CN/ -p
echo "<?php
return [
    'Input :parameter must be of type :exceptType, :actualType given' => 'O parâmetro de entrada :parameter deve ser do tipo :exceptType, o tipo passado é :actualType',
    'Missing input parameter :parameter' => 'Falta o parâmetro de entrada :parameter',
];" > resource/translations/zh_CN/messages.php
php start.php restart
```

#### Outros tipos
O webman suporta tipos como `int`, `float`, `string`, `bool`, `array`, `object` e `instâncias de classe`. Por exemplo:

```php
<?php
namespace app\controller;
use support\Response;

class UserController
{
    public function create(string $name, int $age, float $balance, bool $vip, array $extension): Response
    {
        return json([
            'name' => $name,
            'age' => $age,
            'balance' => $balance,
            'vip' => $vip,
            'extension' => $extension,
        ]);
    }
}
```

Ao acessar `/user/create?name=tom&age=18&balance=100.5&vip=true&extension[foo]=bar`, obterá:

```json
{
  "name": "tom",
  "age": 18,
  "balance": 100.5,
  "vip": true,
  "extension": {
    "foo": "bar"
  }
}
```

#### Instância de classe
O webman suporta passar instâncias de classes através de type hints. Por exemplo:

**app\service\Blog.php**
```php
<?php
namespace app\service;
class Blog
{
    private $title;
    private $content;
    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }
    public function get()
    {
        return [
            'title' => $this->title,
            'content' => $this->content,
        ];
    }
}
```

**app\controller\BlogController.php**
```php
<?php
namespace app\controller;
use app\service\Blog;
use support\Response;

class BlogController
{
    public function create(Blog $blog): Response
    {
        return json($blog->get());
    }
}
```

Ao acessar `/blog/create?blog[title]=hello&blog[content]=world`, obterá:

```json
{
  "title": "hello",
  "content": "world"
}
```

#### Instância de modelo

**app\model\User.php**
```php
<?php
namespace app\model;
use support\Model;
class User extends Model
{
    protected $connection = 'mysql';
    protected $table = 'user';
    protected $primaryKey = 'id';
    public $timestamps = false;
    // Definir aqui os campos preenchíveis para evitar campos inseguros do frontend
    protected $fillable = ['name', 'age'];
}
```

**app\controller\UserController.php**
```php
<?php
namespace app\controller;
use app\model\User;
class UserController
{
    public function create(User $user): int
    {
        $user->save();
        return $user->id;
    }
}
```

Ao acessar `/user/create?user[name]=tom&user[age]=18`, obterá um resultado semelhante a:

```json
1
```

## Ciclo de vida do controlador

Quando `controller_reuse` em `config/app.php` é `false`, cada pedido inicializa uma vez a instância do controlador, que é destruída após o fim do pedido. É o mesmo comportamento dos frameworks tradicionais.

Quando `controller_reuse` em `config/app.php` é `true`, todos os pedidos reutilizam a mesma instância do controlador, que permanece em memória após ser criada.

> **Nota**
> Com a reutilização ativada, os pedidos não devem alterar propriedades do controlador, pois as alterações afetariam pedidos subsequentes. Por exemplo:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    protected $model;
    
    public function update(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->update();
        return response('ok');
    }
    
    public function delete(Request $request, $id)
    {
        $model = $this->getModel($id);
        $model->delete();
        return response('ok');
    }
    
    protected function getModel($id)
    {
        // Este método mantém o modelo após o primeiro pedido update?id=1
        // Se chegar outro pedido delete?id=2, serão eliminados os dados do id 1
        if (!$this->model) {
            $this->model = Model::find($id);
        }
        return $this->model;
    }
}
```

> **Dica**
> Retornar dados no construtor `__construct()` do controlador não tem efeito. Por exemplo:

```php
<?php
namespace app\controller;

use support\Request;

class FooController
{
    public function __construct()
    {
        // O return no construtor não tem efeito; o browser não receberá esta resposta
        return response('hello'); 
    }
}
```

## Diferença entre controlador sem e com reutilização

#### Sem reutilização
Cada pedido cria uma nova instância do controlador, libertada após o fim do pedido. Comportamento tradicional, comum à maioria dos programadores. Desempenho ligeiramente inferior (cerca de 10% em benchmark helloworld, geralmente negligível em projetos reais).

#### Com reutilização
Uma instância é criada uma vez por processo e mantida em memória. Os pedidos seguintes reutilizam-na. Melhor desempenho, mas menos habitual para muitos programadores.

#### A reutilização não é possível quando:

O pedido altera propriedades do controlador; essas alterações afetariam pedidos subsequentes.

É feita inicialização no `__construct()` para cada pedido; o construtor só é chamado uma vez por processo, não em cada pedido.
