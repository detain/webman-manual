# Componente di attività pianificata Crontab

## Descrizione

`workerman/crontab` è simile al crontab di Linux, con la differenza che supporta la pianificazione al secondo.

Formato del tempo:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[opzionale; se manca la posizione 0, la granularità minima è il minuto]
```

## URL del progetto

https://github.com/walkor/crontab
  
## Installazione
 
```php
composer require workerman/crontab
```
  
## Utilizzo

**Passo 1: Creare il file di processo `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // Esegui ogni secondo
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Esegui ogni 5 secondi
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Esegui ogni minuto
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Esegui ogni 5 minuti
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Esegui al primo secondo di ogni minuto
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Esegui alle 7:50 ogni giorno (il campo secondi è omesso qui)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Passo 2: Configurare il processo per avviarsi con webman**
  
Aprire il file di configurazione `config/process.php` e aggiungere quanto segue:

```php
return [
    ....altre configurazioni omesse....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Passo 3: Riavviare webman**

> Nota: Le attività pianificate non si eseguono subito; iniziano dal minuto successivo.

## Note
Crontab non è asincrono. Esempio: un processo task imposta due timer A e B, entrambi eseguiti ogni secondo. Se il compito A richiede 10 secondi, B deve aspettare la fine di A prima di eseguirsi, causando un ritardo per B.
Se la logica è sensibile all'intervallo di tempo, eseguire le attività sensibili al tempo in processi separati per evitare interferenze. Esempio per `config/process.php`:

```php
return [
    ....altre configurazioni omesse....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Mettere le attività sensibili al tempo in `process/Task1.php` e le altre in `process/Task2.php`.

Per ulteriori informazioni su `config/process.php`, consultare [Processi personalizzati](../process.md).
