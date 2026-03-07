# Sobre Vazamento de Memória
Webman é um framework de memória residente, por isso precisamos prestar atenção aos vazamentos de memória. No entanto, os desenvolvedores não precisam se preocupar em excesso, pois os vazamentos ocorrem apenas em condições muito extremas e são facilmente evitáveis. O desenvolvimento com webman é basicamente igual ao de frameworks tradicionais; não é necessário realizar operações extras de gestão de memória.

> **Dica**
> O processo de monitoramento embutido do webman monitora o uso de memória de todos os processos. Quando o uso de memória de um processo está prestes a atingir o valor definido em `memory_limit` no php.ini, o processo correspondente é reiniciado automaticamente de forma segura para libertar memória, sem afetar a aplicação.

## Definição de Vazamento de Memória
É normal que o uso de memória do webman aumente com o número de solicitações. Em geral, quando um processo atinge um certo volume de solicitações (tipicamente da ordem dos milhões), a memória estabiliza ou aumenta apenas ocasionalmente.

Na maioria das aplicações, o uso de memória por processo acaba por estabilizar em torno de 10M–100M. Não há motivo para preocupação enquanto o uso por processo se mantiver abaixo de 100M.

Além disso, ao processar ficheiros grandes, solicitações grandes ou ao ler grandes quantidades de dados da base de dados, o PHP reserva muita memória. O PHP pode conservar parte desta memória para reutilização em vez de a devolver toda ao sistema operativo, o que pode levar a um uso de memória elevado. Como a memória é reutilizada, não há motivo para preocupação.

> **Dica**
> Em projetos em pacote phar ou binário, se o tamanho do pacote for grande, é normal que o uso de memória exceda 100M.

## Como Confirmar um Vazamento de Memória
Se um processo tratou mais de um milhão de solicitações, o uso de memória excede 100M e a memória continua a aumentar após cada solicitação, pode estar a ocorrer um vazamento de memória.

## Como Localizar um Vazamento de Memória
Uma abordagem simples é submeter cada API a testes de carga e identificar qual continua a aumentar o uso de memória após milhões de solicitações.

Depois de encontrar a API problemática, pode usar pesquisa binária: comentar metade do código de negócio de cada vez até identificar o trecho que causa o problema.

## Como Ocorrem os Vazamentos de Memória
**Um vazamento de memória só ocorre quando as duas condições seguintes são cumpridas:**
1. Existe um array de **ciclo de vida longo** (arrays normais não são problema)
2. E esse array de **ciclo de vida longo** cresce indefinidamente (a aplicação continua a inserir dados e nunca os remove)

Só quando **ambas** as condições são cumpridas ocorre um vazamento. Se alguma condição não for cumprida ou apenas uma for cumprida, não há vazamento.

## Arrays de Ciclo de Vida Longo

No webman, arrays de ciclo de vida longo incluem:
1. Arrays com a palavra-chave `static`
2. Propriedades de tipo array em singletons
3. Arrays com a palavra-chave `global`

> **Nota**
> O webman permite dados de ciclo de vida longo, mas é necessário garantir que os dados sejam limitados e que o número de elementos não cresça indefinidamente.

Seguem exemplos de cada caso.

### Array static que Cresce Indefinidamente
```php
class Foo
{
    public static $data = [];
    public function index(Request $request)
    {
        self::$data[] = time();
        return response('hello');
    }
}
```

O array `$data` definido com `static` tem ciclo de vida longo. No exemplo, `$data` continua a crescer com cada solicitação, provocando um vazamento de memória.

### Propriedade de Array de Singleton que Cresce Indefinidamente
```php
class Cache
{
    protected static $instance;
    public $data = [];
    
    public function instance()
    {
        if (!self::$instance) {
            self::$instance = new self;
        }
        return self::$instance;
    }
    
    public function set($key, $value)
    {
        $this->data[$key] = $value;
    }
}
```

Código de chamada
```php
class Foo
{
    public function index(Request $request)
    {
        Cache::instance()->set(time(), time());
        return response('hello');
    }
}
```

`Cache::instance()` retorna uma instância singleton de Cache, com ciclo de vida longo. Embora a propriedade `$data` não use `static`, como a classe tem ciclo de vida longo, `$data` também é um array de ciclo de vida longo. À medida que se adicionam chaves diferentes a `$data`, o uso de memória do programa aumenta e ocorre o vazamento.

> **Nota**
> Se as chaves adicionadas com `Cache::instance()->set(key, value)` forem em número finito, não há vazamento, pois o array `$data` não cresce indefinidamente.

### Array global que Cresce Indefinidamente
```php
class Index
{
    public function index(Request $request)
    {
        global $data;
        $data[] = time();
        return response($foo->sayHello());
    }
}
```
Arrays definidos com `global` não são libertados quando a função ou o método termina, pelo que têm ciclo de vida longo. O código acima provoca um vazamento à medida que as solicitações aumentam. Do mesmo modo, arrays definidos com `static` dentro de uma função ou método também têm ciclo de vida longo; se crescerem indefinidamente, provocam vazamento, por exemplo:
```php
class Index
{
    public function index(Request $request)
    {
        static $data = [];
        $data[] = time();
        return response($foo->sayHello());
    }
}
```

## Recomendações
Recomenda-se que os desenvolvedores não prestem atenção excessiva aos vazamentos de memória, pois são raros. Se ocorrer um, testes de carga permitem localizar o código que o provoca. Mesmo que o desenvolvedor não encontre a causa, o serviço de monitorização do webman reiniciará a tempo o processo afetado para libertar memória.

Se quiser minimizar o risco de vazamentos, pode seguir estas recomendações:
1. Evitar, tanto quanto possível, arrays com `global` ou `static`; se os usar, garantir que não crescem indefinidamente.
2. Para classes pouco conhecidas, preferir inicialização com `new` em vez de singletons. Se usar um singleton, verificar se tem propriedades de array que possam crescer indefinidamente.
