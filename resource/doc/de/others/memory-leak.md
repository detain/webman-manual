# Über Speicherverluste
Webman ist ein residentes Speicherframework, daher müssen wir etwas auf Speicherverluste achten. Entwickler müssen sich jedoch keine großen Sorgen machen, da Speicherverluste nur unter sehr extremen Bedingungen auftreten und leicht vermieden werden können. Die Entwicklung mit Webman entspricht im Wesentlichen herkömmlichen Frameworks; für Speichermanagement sind keine zusätzlichen Maßnahmen erforderlich.

> **Hinweis**
> Der mitgelieferte Monitor-Prozess von Webman überwacht den Speicherverbrauch aller Prozesse. Wenn der Speicherverbrauch eines Prozesses den in php.ini unter `memory_limit` gesetzten Wert erreicht, wird der entsprechende Prozess automatisch sicher neu gestartet, um Speicher freizugeben – ohne Auswirkung auf die Anwendung.

## Definition von Speicherverlusten
Dass der Speicherverbrauch von Webman mit steigender Anzahl von Anfragen zunimmt, ist normal. Allgemein gilt: Wenn ein Prozess eine bestimmte Zahl von Anfragen erreicht (typischerweise im Millionenbereich), stabilisiert sich der Speicherverbrauch oder wächst nur noch gelegentlich leicht.

Bei den meisten Anwendungen pendelt sich der Speicherverbrauch pro Prozess bei etwa 10M–100M ein. Solange der Verbrauch eines einzelnen Prozesses unter 100M bleibt, besteht kein Grund zur Sorge.

Wenn zudem große Dateien verarbeitet, große Anfragen bearbeitet oder große Datenmengen aus der Datenbank gelesen werden, beantragt PHP viel Speicher. PHP kann diesen Speicher teilweise behalten und wiederverwenden, statt ihn vollständig an das Betriebssystem zurückzugeben. Das führt zu höherem Speicherverbrauch. Da der Speicher aber wiederverwendet wird, besteht hier ebenfalls kein Grund zur Sorge.

> **Hinweis**
> Bei per phar oder als Binärpaket verpackten Projekten kann der Speicherverbrauch über 100M liegen, wenn die Paketgröße groß ist – das ist normal.

## So erkennt man einen Speicherverlust
Wenn ein Prozess mehr als eine Million Anfragen bearbeitet hat, der Speicherverbrauch über 100M liegt und der Speicher nach jeder Anfrage weiter wächst, könnte ein Speicherverlust vorliegen.

## So findet man einen Speicherverlust
Eine einfache Methode ist, die einzelnen APIs unter Last zu testen und festzustellen, welche nach Millionen von Anfragen den Speicherverbrauch weiter erhöht.

Sobald eine problematische API gefunden ist, kann man mit der Bisektionsmethode vorgehen: jeweils die Hälfte des Geschäftscodes auskommentieren, bis die fehlerhafte Stelle eingegrenzt ist.

## Wie Speicherverluste entstehen
**Ein Speicherverlust tritt nur ein, wenn beide folgenden Bedingungen erfüllt sind:**
1. Es existiert ein Array mit **langer Lebensdauer** (normale Arrays sind unproblematisch)
2. Und dieses Array mit **langer Lebensdauer** wächst unbegrenzt (die Anwendung fügt ständig Daten hinzu und entfernt sie nie)

Nur wenn **beide** Bedingungen erfüllt sind, entsteht ein Speicherverlust. Ist eine Bedingung nicht erfüllt oder nur eine von beiden, liegt kein Speicherverlust vor.

## Arrays mit langer Lebensdauer

In Webman zählen dazu:
1. Arrays mit dem Schlüsselwort `static`
2. Array-Eigenschaften von Singletons
3. Arrays mit dem Schlüsselwort `global`

> **Hinweis**
> Lange Lebensdauer von Daten ist in Webman zulässig, aber die Daten müssen begrenzt sein und die Anzahl der Elemente darf nicht unbegrenzt wachsen.

Nachfolgend konkrete Beispiele.

### Unbegrenzt wachsendes static-Array
```php
class Foo
{
    public static $data = [];
    public function index(Request $request)
    {
        self::$data[] = time();
        return response('hallo');
    }
}
```

Das mit `static` definierte Array `$data` hat eine lange Lebensdauer. Im Beispiel wächst `$data` mit jeder Anfrage, was zu einem Speicherverlust führt.

### Unbegrenzt wachsendes Singleton-Array
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

Aufrufcode
```php
class Foo
{
    public function index(Request $request)
    {
        Cache::instance()->set(time(), time());
        return response('hallo');
    }
}
```

`Cache::instance()` liefert eine Cache-Singleton-Instanz mit langer Lebensdauer. Obwohl die Eigenschaft `$data` kein `static` verwendet, hat `$data` ebenfalls lange Lebensdauer, weil die Klasse selbst lange lebt. Durch ständiges Hinzufügen neuer Keys wächst der Speicherverbrauch und es kommt zum Speicherverlust.

> **Hinweis**
> Wenn die über `Cache::instance()->set(key, value)` hinzugefügten Keys in begrenzter Anzahl sind, entsteht kein Speicherverlust, weil das Array `$data` nicht unbegrenzt wächst.

### Unbegrenzt wachsendes global-Array
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
Arrays mit `global` werden nach Abschluss einer Funktion oder Methode nicht freigegeben, haben also lange Lebensdauer. Der obige Code verursacht bei steigender Anfragenzahl einen Speicherverlust. Ebenso sind Arrays mit `static` innerhalb von Funktionen oder Methoden Arrays mit langer Lebensdauer; wenn sie unbegrenzt wachsen, kommt es ebenfalls zum Speicherverlust, z. B.:
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

## Empfehlungen
Entwickler sollten Speicherverluste nicht übermäßig beachten, da sie selten auftreten. Wenn doch einer auftritt, hilft Lasttesting, den verursachenden Code zu finden. Selbst wenn Entwickler die Stelle nicht finden, startet Webmans Monitor-Dienst den betroffenen Prozess rechtzeitig sicher neu und gibt Speicher frei.

Wer Speicherverluste möglichst vermeiden möchte, kann folgende Punkte beachten:
1. Arrays mit `global` oder `static` möglichst vermeiden; falls verwendet, sicherstellen, dass sie nicht unbegrenzt wachsen.
2. Bei unbekannten Klassen eher `new` zur Initialisierung nutzen als Singletons. Bei Singletons prüfen, ob Array-Eigenschaften unbegrenzt wachsen können.
