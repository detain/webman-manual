# Grundlegender Plugin-Erstellungs- und Veröffentlichungsprozess

## Prinzip
1. Am Beispiel eines Cross-Domain-Plugins besteht ein Plugin aus drei Teilen: einer Cross-Domain-Middleware-Programmdatei, einer Middleware-Konfigurationsdatei `middleware.php` und einer per Befehl automatisch generierten `Install.php`.
2. Wir nutzen Befehle, um diese drei Dateien zu bündeln und über Composer zu veröffentlichen.
3. Bei der Composer-Installation des Cross-Domain-Plugins kopiert `Install.php` die Middleware-Programmdatei und die Konfigurationsdatei nach `{Hauptprojekt}/config/plugin`, damit webman sie laden kann. So wird die Cross-Domain-Middleware automatisch aktiv.
4. Bei der Composer-Deinstallation entfernt `Install.php` die zugehörigen Middleware- und Konfigurationsdateien und führt damit die automatische Plugin-Deinstallation durch.

## Vorgaben
1. Ein Pluginname besteht aus `Vendor` und `Pluginname`, z. B. `webman/push`, passend zum Composer-Paketnamen.
2. Plugin-Konfigurationsdateien liegen unter `config/plugin/Vendor/Pluginname/` (der Konsolenbefehl erstellt das Konfigurationsverzeichnis automatisch). Braucht ein Plugin keine Konfiguration, muss das automatisch erstellte Verzeichnis gelöscht werden.
3. Das Plugin-Konfigurationsverzeichnis unterstützt nur: `app.php` (Hauptkonfiguration), `bootstrap.php` (Prozessstart), `route.php` (Routen), `middleware.php` (Middleware), `process.php` (eigene Prozesse), `database.php` (Datenbank), `redis.php` (Redis), `thinkorm.php` (thinkorm). Diese werden von webman automatisch erkannt.
4. Zugriff auf die Konfiguration: `config('plugin.Vendor.Pluginname.Konfigurationsdatei.Konfigurationseintrag');`, z. B. `config('plugin.webman.push.app.app_key')`.
5. Bei eigener DB-Konfiguration: `illuminate/database` nutzt `Db::connection('plugin.Vendor.Pluginname.Verbindung')`, `thinkorm` nutzt `Db::connect('plugin.Vendor.Pluginname.Verbindung')`.
6. Legt ein Plugin Dateien unter `app/` ab, darf es nicht mit dem Hauptprojekt oder anderen Plugins kollidieren.
7. Plugins sollten nach Möglichkeit keine Dateien oder Verzeichnisse ins Hauptprojekt kopieren. Beim Cross-Domain-Plug-in z. B. nur die Konfiguration; Middleware-Dateien bleiben unter `vendor/webman/cros/src`.
8. Plugin-Namespaces sollten PascalCase verwenden, z. B. `Webman/Console`.

## Beispiel

**`webman/console` installieren**

`composer require webman/console`

### Plugin erstellen

Angenommen, das zu erstellende Plugin heißt `foo/admin` (entspricht dem später über Composer veröffentlichten Projektnamen, Kleinbuchstaben). Befehl ausführen:

`php webman plugin:create --name=foo/admin`

Dabei entstehen `vendor/foo/admin` (für Plugin-Dateien) und `config/plugin/foo/admin` (für die Plugin-Konfiguration).

> Hinweis
> `config/plugin/foo/admin` unterstützt: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Format wie webman, automatische Zusammenführung.
> Zugriff mit Präfix `plugin`, z. B. `config('plugin.foo.admin.app')`.


### Plugin exportieren

Nach Abschluss der Entwicklung Export-Befehl ausführen:

`php webman plugin:export --name=foo/admin`

Export

> Erläuterung
> Beim Export wird `config/plugin/foo/admin` nach `vendor/foo/admin/src` kopiert und `Install.php` erzeugt. `Install.php` läuft bei Installation und Deinstallation.
> Standard-Installation: Konfiguration von `vendor/foo/admin/src` nach `config/plugin` des Projekts kopieren.
> Standard-Deinstallation: entsprechende Konfigurationsdateien unter `config/plugin` des Projekts entfernen.
> `Install.php` kann für eigene Install/Deinstall-Logik angepasst werden.

### Plugin einreichen
* Voraussetzung: [GitHub](https://github.com)- und [Packagist](https://packagist.org)-Konto.
* Auf [GitHub](https://github.com) ein `admin`-Repository anlegen und Code pushen, z. B. `https://github.com/dein-benutzername/admin`.
* Unter `https://github.com/dein-benutzername/admin/releases/new` einen Release erstellen, z. B. `v1.0.0`.
* Auf [Packagist](https://packagist.org) auf `Submit` klicken und `https://github.com/dein-benutzername/admin` eintragen, um das Plugin zu veröffentlichen.

> **Tipp**
> Bei Namenskonflikten auf Packagist einen anderen Vendor-Namen wählen, z. B. `foo/admin` in `myfoo/admin` ändern.

Bei Updates: Code nach GitHub pushen, unter `https://github.com/dein-benutzername/admin/releases/new` einen neuen Release erstellen und auf `https://packagist.org/packages/foo/admin` auf `Update` klicken.

## Befehle zum Plugin hinzufügen
Manche Plugins benötigen eigene Befehle. Z. B. fügt `webman/redis-queue` den Befehl `redis-queue:consumer` hinzu; mit `php webman redis-queue:consumer send-mail` lässt sich schnell eine `SendMail.php`-Consumer-Klasse erzeugen.

Um dem Plugin `foo/admin` den Befehl `foo-admin:add` hinzuzufügen:

### Befehl erstellen

**Datei `vendor/foo/admin/src/FooAdminAddCommand.php` anlegen**

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
    protected static $defaultDescription = 'Befehlsbeschreibung';

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

> **Hinweis**
> Um Befehls-Konflikte zwischen Plugins zu vermeiden, Format `Vendor-Plugin:Befehl` verwenden. Alle Befehle von `foo/admin` sollten mit `foo-admin:` beginnen, z. B. `foo-admin:add`.

### Konfiguration hinzufügen
**Datei `config/plugin/foo/admin/command.php` anlegen**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Bei Bedarf weitere Befehle ...
];
```

> **Tipp**
> `command.php` registriert benutzerdefinierte Befehle des Plugins. Jeder Eintrag ist eine Befehlsklasse. `webman/console` lädt diese automatisch. Siehe [Konsolenbefehle](console.md).

### Export ausführen
`php webman plugin:export --name=foo/admin` ausführen, Plugin exportieren und auf Packagist veröffentlichen. Nach Installation von `foo/admin` steht der Befehl `foo-admin:add` zur Verfügung. `php webman foo-admin:add jerry` gibt `Admin add jerry` aus.
