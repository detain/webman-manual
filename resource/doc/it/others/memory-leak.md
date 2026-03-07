# Riguardo alle perdite di memoria
Webman è un framework residente in memoria, quindi è necessario prestare una certa attenzione alle perdite di memoria. Tuttavia, gli sviluppatori non devono preoccuparsi eccessivamente, poiché le perdite si verificano solo in condizioni molto estreme e sono facili da evitare. Lo sviluppo con webman è sostanzialmente uguale a quello con framework tradizionali; non sono necessarie operazioni extra per la gestione della memoria.

> **Nota**
> Il processo di monitoraggio integrato di webman controlla l'utilizzo della memoria di tutti i processi. Quando l'utilizzo della memoria di un processo sta per raggiungere il valore impostato in `memory_limit` in php.ini, il processo corrispondente viene riavviato automaticamente in modo sicuro per liberare memoria, senza impatto sull'applicazione.

## Definizione di perdita di memoria
È normale che l'utilizzo della memoria di webman aumenti con il numero di richieste. In generale, quando un processo raggiunge un certo volume di richieste (tipicamente nell'ordine dei milioni), la memoria si stabilizza o aumenta solo occasionalmente.

Nella maggior parte delle applicazioni, l'utilizzo della memoria per processo finisce per stabilizzarsi intorno a 10M–100M. Non c'è motivo di preoccupazione finché l'utilizzo per processo resta sotto i 100M.

Inoltre, durante l'elaborazione di file grandi, richieste pesanti o letture di grandi quantità di dati dal database, PHP alloca molta memoria. PHP può conservare parte di questa memoria per riutilizzarla invece di restituirla tutta al sistema operativo, il che può portare a un utilizzo di memoria elevato. Poiché la memoria viene riutilizzata, non c'è motivo di preoccupazione.

> **Nota**
> Per progetti impacchettati in phar o binario, se la dimensione del pacchetto è grande, è normale che l'utilizzo della memoria superi i 100M.

## Come confermare una perdita di memoria
Se un processo ha gestito più di un milione di richieste, l'utilizzo della memoria supera i 100M e la memoria continua ad aumentare dopo ogni richiesta, potrebbe verificarsi una perdita di memoria.

## Come individuare una perdita di memoria
Un approccio semplice è sottoporre ogni API a test di carico e individuare quale continua ad aumentare l'utilizzo della memoria dopo milioni di richieste.

Una volta trovata l'API problematica, si può usare una ricerca binaria: commentare metà del codice di business alla volta fino a individuare il codice che causa il problema.

## Come si verificano le perdite di memoria
**Una perdita di memoria si verifica solo quando entrambe le seguenti condizioni sono soddisfatte:**
1. Esiste un array con **ciclo di vita lungo** (gli array normali non sono un problema)
2. E questo array con **ciclo di vita lungo** cresce indefinitamente (l'applicazione continua a inserire dati e non li elimina mai)

Solo quando **entrambe** le condizioni sono soddisfatte si ha una perdita. Se una condizione non è soddisfatta o ne è soddisfatta solo una, non c'è perdita.

## Array con ciclo di vita lungo

In webman gli array con ciclo di vita lungo includono:
1. Array con la parola chiave `static`
2. Proprietà di tipo array dei singleton
3. Array con la parola chiave `global`

> **Nota**
> webman consente dati con ciclo di vita lungo, ma bisogna garantire che i dati siano limitati e che il numero di elementi non cresca indefinitamente.

Di seguito esempi per ogni caso.

### Array static che cresce indefinitamente
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

L'array `$data` definito con `static` ha un ciclo di vita lungo. Nell'esempio, `$data` continua a crescere con ogni richiesta, provocando una perdita di memoria.

### Proprietà di array di singleton che cresce indefinitamente
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

Codice di chiamata
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

`Cache::instance()` restituisce un singleton Cache con ciclo di vita lungo. Anche se la proprietà `$data` non usa `static`, poiché la classe ha ciclo di vita lungo, anche `$data` è un array con ciclo di vita lungo. Man mano che si aggiungono chiavi diverse a `$data`, l'utilizzo della memoria del programma aumenta e si verifica la perdita.

> **Nota**
> Se le chiavi aggiunte con `Cache::instance()->set(key, value)` sono in numero limitato, non c'è perdita, perché l'array `$data` non cresce indefinitamente.

### Array global che cresce indefinitamente
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
Gli array definiti con `global` non vengono liberati al termine della funzione o del metodo, quindi hanno ciclo di vita lungo. Il codice sopra provoca una perdita man mano che aumentano le richieste. Allo stesso modo, gli array definiti con `static` dentro una funzione o un metodo hanno anch'essi ciclo di vita lungo; se crescono indefinitamente, provocano una perdita, ad esempio:
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

## Raccomandazioni
Si consiglia di non prestare eccessiva attenzione alle perdite di memoria, perché sono rare. Se ne compare una, i test di carico consentono di individuare il codice che la provoca. Anche se lo sviluppatore non trova la causa, il servizio di monitoraggio integrato di webman riavvierà in tempo il processo interessato per liberare memoria.

Se desiderate ridurre al minimo il rischio di perdite, potete seguire queste raccomandazioni:
1. Evitare, quando possibile, array con `global` o `static`; se li usate, assicuratevi che non crescano indefinitamente.
2. Per classi poco conosciute, preferire l'inizializzazione con `new` invece dei singleton. Se usate un singleton, verificare se ha proprietà di tipo array che possono crescere indefinitamente.
