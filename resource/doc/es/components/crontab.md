# Componente de tareas programadas Crontab

## Descripción

`workerman/crontab` es similar al crontab de Linux, con la diferencia de que admite programación a nivel de segundos.

Formato de tiempo:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[opcional; si falta, la granularidad mínima es el minuto]
```

## URL del proyecto

https://github.com/walkor/crontab
  
## Instalación
 
```php
composer require workerman/crontab
```
  
## Uso

**Paso 1: Crear el archivo de proceso `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // Ejecutar cada segundo
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Ejecutar cada 5 segundos
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Ejecutar cada minuto
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Ejecutar cada 5 minutos
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Ejecutar en el primer segundo de cada minuto
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Ejecutar a las 7:50 cada día (aquí se omite el campo de segundos)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Paso 2: Configurar el proceso para que arranque con webman**
  
Abrir el archivo de configuración `config/process.php` y añadir lo siguiente:

```php
return [
    ....otras configuraciones omitidas....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Paso 3: Reiniciar webman**

> Nota: Las tareas programadas no se ejecutan de inmediato; empiezan a contar y a ejecutarse desde el siguiente minuto.

## Notas
Crontab no es asíncrono. Por ejemplo: un proceso task tiene dos temporizadores A y B que se ejecutan cada segundo. Si la tarea A tarda 10 segundos, B debe esperar a que A termine antes de ejecutarse, lo que retrasa a B.
Si la lógica es sensible al intervalo de tiempo, ejecutar las tareas sensibles en procesos separados para evitar interferencias. Ejemplo en `config/process.php`:

```php
return [
    ....otras configuraciones omitidas....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Colocar las tareas sensibles al tiempo en `process/Task1.php` y el resto en `process/Task2.php`.

Para más información sobre `config/process.php`, consultar [Procesos personalizados](../process.md).
