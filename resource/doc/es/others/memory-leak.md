# Sobre las fugas de memoria
Webman es un framework residente en memoria, por lo que conviene prestar cierta atención a las fugas de memoria. No obstante, los desarrolladores no deben preocuparse en exceso, ya que estas ocurren solo en condiciones muy extremas y son fáciles de evitar. El desarrollo con webman es prácticamente igual que con frameworks tradicionales; no hace falta realizar operaciones adicionales para la gestión de memoria.

> **Consejo**
> El proceso monitor integrado de webman supervisa el uso de memoria de todos los procesos. Si el uso de memoria de un proceso está a punto de alcanzar el valor configurado en `memory_limit` de php.ini, reiniciará automática y de forma segura el proceso correspondiente para liberar memoria, sin afectar a la aplicación.

## Definición de fuga de memoria
Que el uso de memoria de webman aumente con el número de peticiones es normal. En general, cuando un proceso alcanza un cierto volumen de peticiones (típicamente del orden de millones), el uso de memoria se estabiliza o crece solo ocasionalmente.

En la mayoría de aplicaciones, el uso de memoria por proceso termina estabilizándose en torno a 10M–100M. No hay motivo de preocupación mientras el uso por proceso se mantenga por debajo de 100M.

Además, al manejar archivos grandes, peticiones grandes o leer grandes cantidades de datos de la base de datos, PHP reserva mucha memoria. PHP puede conservar parte de esta memoria para reutilizarla en lugar de devolverla toda al sistema operativo, lo que puede provocar un uso de memoria alto. Como la memoria se reutiliza, no hay motivo de preocupación.

> **Consejo**
> En proyectos empaquetados en phar o binario, si el tamaño del paquete es grande, es normal que el uso de memoria supere los 100M.

## Cómo confirmar una fuga de memoria
Si un proceso ha atendido más de un millón de peticiones, el uso de memoria supera los 100M y la memoria sigue creciendo tras cada petición, es posible que haya una fuga de memoria.

## Cómo localizar una fuga de memoria
Un método sencillo es someter cada API a pruebas de estrés e identificar cuál sigue aumentando el uso de memoria tras millones de peticiones.

Una vez localizada la API problemática, se puede aplicar una búsqueda binaria: comentar la mitad del código de negocio cada vez hasta dar con el tramo que causa el problema.

## Cómo se producen las fugas de memoria
**Una fuga de memoria solo se produce cuando se cumplen estas dos condiciones:**
1. Existe un array de **ciclo de vida largo** (los arrays normales no son problema)
2. Y ese array de **ciclo de vida largo** crece indefinidamente (la aplicación sigue insertando datos y nunca los elimina)

Solo cuando se cumplen **ambas** condiciones hay fuga de memoria. Si falta alguna condición o solo se cumple una, no hay fuga.

## Arrays de ciclo de vida largo

En webman, los arrays de ciclo de vida largo incluyen:
1. Arrays con la palabra clave `static`
2. Propiedades de tipo array en singletons
3. Arrays con la palabra clave `global`

> **Nota**
> Webman permite datos de ciclo de vida largo, pero hay que garantizar que los datos sean acotados y que el número de elementos no crezca indefinidamente.

A continuación se muestran ejemplos de cada caso.

### Array static que crece indefinidamente
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

El array `$data` definido con `static` tiene un ciclo de vida largo. En el ejemplo, `$data` sigue creciendo con cada petición, lo que provoca una fuga de memoria.

### Propiedad de array en singleton que crece indefinidamente
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

Código de llamada
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

`Cache::instance()` devuelve una instancia singleton de Cache, que tiene ciclo de vida largo. Aunque la propiedad `$data` no usa `static`, como la clase tiene ciclo de vida largo, `$data` también es un array de ciclo de vida largo. A medida que se añaden distintas claves a `$data`, el uso de memoria del programa aumenta y se produce la fuga.

> **Nota**
> Si las claves añadidas con `Cache::instance()->set(key, value)` son finitas, no hay fuga, porque el array `$data` no crece indefinidamente.

### Array global que crece indefinidamente
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
Los arrays definidos con `global` no se liberan al terminar la función o el método, por lo que tienen ciclo de vida largo. El código anterior provoca una fuga a medida que aumentan las peticiones. Del mismo modo, los arrays definidos con `static` dentro de una función o método también tienen ciclo de vida largo; si crecen indefinidamente, provocan fuga, por ejemplo:
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

## Recomendaciones
Se recomienda no prestar atención excesiva a las fugas de memoria, ya que son poco frecuentes. Si ocurren, las pruebas de estrés permiten localizar el código que las provoca. Incluso si el desarrollador no encuentra la causa, el servicio monitor de webman reiniciará a tiempo el proceso afectado para liberar memoria.

Si se quiere minimizar el riesgo de fugas, se pueden seguir estas recomendaciones:
1. Evitar en lo posible arrays con `global` o `static`; si se usan, asegurarse de que no crezcan indefinidamente.
2. Para clases poco conocidas, preferir inicializar con `new` en lugar de singletons. Si se usa un singleton, comprobar si tiene propiedades de tipo array que puedan crecer indefinidamente.
