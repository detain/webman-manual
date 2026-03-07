# Crontab-Zeitplanungskomponente

## Beschreibung

`workerman/crontab` ist ähnlich wie Linux crontab, mit dem Unterschied, dass es die Planung im Sekundentakt unterstützt.

Zeitformat:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[optional; wenn 0. Stelle fehlt, ist die Mindestgranularität die Minute]
```

## Projekt-URL

https://github.com/walkor/crontab
  
## Installation
 
```php
composer require workerman/crontab
```
  
## Verwendung

**Schritt 1: Prozessdatei `app/process/Task.php` erstellen**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {

        // Jede Sekunde ausführen
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Alle 5 Sekunden ausführen
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Jede Minute ausführen
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Alle 5 Minuten ausführen
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // In der ersten Sekunde jeder Minute ausführen
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Täglich um 7:50 ausführen (Sekunde hier weggelassen)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Schritt 2: Prozess mit webman starten konfigurieren**
  
Konfigurationsdatei `config/process.php` öffnen und Folgendes ergänzen:

```php
return [
    ....andere Konfiguration ausgelassen....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Schritt 3: webman neu starten**

> Hinweis: Geplante Aufgaben laufen nicht sofort; sie beginnen erst ab der nächsten Minute.

## Hinweise
Crontab ist nicht asynchron. Beispiel: Ein task-Prozess hat zwei Timer A und B, beide laufen jede Sekunde. Dauert Aufgabe A 10 Sekunden, muss B warten, bis A fertig ist – dadurch verzögert sich B.
Ist die Logik zeitkritisch, sollten zeitkritische Aufgaben in eigenen Prozessen laufen, damit andere sie nicht beeinträchtigen. Beispiel für `config/process.php`:

```php
return [
    ....andere Konfiguration ausgelassen....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Zeitkritische Aufgaben in `process/Task1.php`, übrige in `process/Task2.php`.

Mehr zu `config/process.php` siehe [Benutzerdefinierte Prozesse](../process.md).
