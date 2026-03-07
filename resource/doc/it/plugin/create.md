# Processo base di generazione e pubblicazione dei plugin

## Principio
1. Prendendo come esempio un plugin cross-origin, il plugin è composto da tre parti: il file del middleware cross-origin, il file di configurazione `middleware.php` e un `Install.php` generato automaticamente da comando.
2. Si usa un comando per impacchettare questi tre file e pubblicarli su Composer.
3. Quando l'utente installa il plugin cross-origin con Composer, `Install.php` copia il file middleware e la configurazione in `{progetto principale}/config/plugin` affinché webman li carichi, attivando così la configurazione cross-origin.
4. Quando l'utente disinstalla il plugin con Composer, `Install.php` rimuove i file middleware e di configurazione corrispondenti, realizzando la disinstallazione automatica del plugin.

## Specifiche
1. Il nome del plugin è composto da `vendor` e `nome del plugin`, ad esempio `webman/push`, corrispondente al nome del pacchetto Composer.
2. I file di configurazione del plugin vanno in `config/plugin/vendor/nome del plugin/` (il comando console crea automaticamente la directory di configurazione). Se il plugin non richiede configurazione, la directory creata automaticamente va eliminata.
3. La directory di configurazione del plugin supporta solo: `app.php` (configurazione principale), `bootstrap.php` (avvio processo), `route.php` (route), `middleware.php` (middleware), `process.php` (processi personalizzati), `database.php` (database), `redis.php` (Redis), `thinkorm.php` (thinkorm). Questi vengono riconosciuti automaticamente da webman.
4. Accesso alla configurazione: `config('plugin.vendor.nome del plugin.file di config.voce specifica');`, ad es. `config('plugin.webman.push.app.app_key')`.
5. Se il plugin ha la propria configurazione DB: per `illuminate/database` usare `Db::connection('plugin.vendor.nome del plugin.connessione')`, per `thinkorm` usare `Db::connect('plugin.vendor.nome del plugin.connessione')`.
6. Se un plugin inserisce file di business in `app/`, deve evitare conflitti con il progetto principale e gli altri plugin.
7. I plugin dovrebbero evitare di copiare file o directory nel progetto principale. Ad es., per il plugin cross-origin si copia solo la configurazione; i file middleware restano in `vendor/webman/cros/src`.
8. Per i namespace dei plugin si raccomanda PascalCase, ad es. `Webman/Console`.

## Esempio

**Installare la riga di comando `webman/console`**

`composer require webman/console`

### Creare un plugin

Supponiamo che il plugin da creare si chiami `foo/admin` (nome del progetto Composer da pubblicare, deve essere in minuscolo). Eseguire:

`php webman plugin:create --name=foo/admin`

Questo crea `vendor/foo/admin` per i file del plugin e `config/plugin/foo/admin` per la configurazione.

> Nota
> `config/plugin/foo/admin` supporta: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Stesso formato di webman, merge automatico.
> Accesso con prefisso `plugin`, ad es. `config('plugin.foo.admin.app')`.


### Esportare il plugin

Al termine dello sviluppo, eseguire:

`php webman plugin:export --name=foo/admin`

Esporta

> Spiegazione
> L'esportazione copia `config/plugin/foo/admin` in `vendor/foo/admin/src` e genera `Install.php`, eseguito in installazione e disinstallazione.
> Installazione predefinita: copia la configurazione da `vendor/foo/admin/src` in `config/plugin` del progetto.
> Disinstallazione predefinita: rimuove i file di configurazione del plugin da `config/plugin` del progetto.
> Si può modificare `Install.php` per aggiungere logica personalizzata in installazione e disinstallazione.

### Inviare il plugin
* Si suppone di avere già un account [GitHub](https://github.com) e [Packagist](https://packagist.org).
* Creare un repository `admin` su [GitHub](https://github.com) e inviare il codice, ad es. `https://github.com/tuousername/admin`.
* Andare su `https://github.com/tuousername/admin/releases/new` e creare un release, ad es. `v1.0.0`.
* Su [Packagist](https://packagist.org) cliccare su `Submit` e inviare `https://github.com/tuousername/admin` per pubblicare il plugin.

> **Suggerimento**
> Se Packagist segnala conflitto di nome, usare un altro vendor, ad es. cambiare `foo/admin` in `myfoo/admin`.

Per gli aggiornamenti: inviare il codice su GitHub, creare un nuovo release su `https://github.com/tuousername/admin/releases/new` e cliccare su `Update` su `https://packagist.org/packages/foo/admin`.

## Aggiungere comandi al plugin
Alcuni plugin richiedono comandi personalizzati. Ad es., con `webman/redis-queue` installato il progetto ha il comando `redis-queue:consumer`. Eseguendo `php webman redis-queue:consumer send-mail` si genera rapidamente una classe consumer `SendMail.php`, utile per lo sviluppo rapido.

Per aggiungere il comando `foo-admin:add` al plugin `foo/admin`:

### Creare un comando

**Creare il file `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Descrizione del comando';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Nota**
> Per evitare conflitti di comandi tra plugin, usare il formato `vendor-plugin:comando`. Ad es., tutti i comandi di `foo/admin` devono avere il prefisso `foo-admin:`, come `foo-admin:add`.

### Aggiungere la configurazione
**Creare `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Aggiungere altri comandi se necessario...
];
```

> **Suggerimento**
> `command.php` registra i comandi personalizzati del plugin. Ogni voce è una classe comando. `webman/console` li carica automaticamente. Vedi [Comandi console](console.md).

### Eseguire l'esportazione
Eseguire `php webman plugin:export --name=foo/admin` per esportare e pubblicare su Packagist. Dopo l'installazione di `foo/admin` sarà disponibile il comando `foo-admin:add`. Eseguendo `php webman foo-admin:add jerry` verrà stampato `Admin add jerry`.
