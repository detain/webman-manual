# Componente de tarefas agendadas Crontab

## Descrição

`workerman/crontab` é semelhante ao crontab do Linux; a diferença é que suporta agendamento ao nível de segundos.

Formato de tempo:

```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[opcional; se omitido, a granularidade mínima é o minuto]
```

## URL do projeto

https://github.com/walkor/crontab
  
## Instalação
 
```php
composer require workerman/crontab
```
  
## Uso

**Passo 1: Criar o arquivo de processo `app/process/Task.php`**

```php
<?php
namespace app\process;

use Workerman\Crontab\Crontab;

class Task
{
    public function onWorkerStart()
    {
    
        // Executar a cada segundo
        new Crontab('*/1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Executar a cada 5 segundos
        new Crontab('*/5 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Executar a cada minuto
        new Crontab('0 */1 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Executar a cada 5 minutos
        new Crontab('0 */5 * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
        // Executar no primeiro segundo de cada minuto
        new Crontab('1 * * * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
      
        // Executar às 7:50 diariamente (o campo de segundos é omitido aqui)
        new Crontab('50 7 * * *', function(){
            echo date('Y-m-d H:i:s')."\n";
        });
        
    }
}
```
  
**Passo 2: Configurar o processo para iniciar com webman**
  
Abrir o arquivo de configuração `config/process.php` e adicionar o seguinte:

```php
return [
    ....outras configurações omitidas....
  
    'task'  => [
        'handler'  => app\process\Task::class
    ],
];
```
  
**Passo 3: Reiniciar webman**

> Observação: As tarefas agendadas não são executadas imediatamente; começam a contar e executar a partir do próximo minuto.

## Notas
O crontab não é assíncrono. Exemplo: um processo task configura dois timers A e B, ambos executados a cada segundo. Se a tarefa A demorar 10 segundos, B precisa esperar A terminar antes de executar, causando atraso em B.
Se a lógica for sensível ao intervalo de tempo, executar as tarefas sensíveis em processos separados para evitar interferência. Exemplo em `config/process.php`:

```php
return [
    ....outras configurações omitidas....
  
    'task1'  => [
        'handler'  => process\Task1::class
    ],
    'task2'  => [
        'handler'  => process\Task2::class
    ],
];
```
Colocar as tarefas sensíveis ao tempo em `process/Task1.php` e as restantes em `process/Task2.php`.

Para mais informações sobre `config/process.php`, consultar [Processos personalizados](../process.md).
