# webman installieren

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: PHP + Composer-Umgebung installieren (bei bestehender Umgebung überspringen)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Hinweis**
> Der obige Befehl gilt für Linux- und macOS-Systeme. Windows-Benutzer müssen PHP separat installieren.

Sie können auch die von webman bereitgestellte [statische PHP-Version](https://www.workerman.net/download) manuell herunterladen und entpacken.

## 1. Projekt erstellen

```php
composer create-project workerman/webman:~2.0
```

> **Tipp**
> Bei Fehlern verwenden Sie möglicherweise einen defekten Composer-Mirror. Führen Sie `composer config -g --unset repos.packagist` aus, um den Proxy zu entfernen.

## 2. Ausführen

Wechseln Sie in das webman-Verzeichnis

#### Windows-Benutzer
Doppelklicken Sie auf `windows.bat` oder führen Sie `php windows.php` aus, um zu starten

> **Tipp**
> Bei Fehlern könnten Funktionen deaktiviert sein. Siehe [Prüfung deaktivierter Funktionen](others/disable-function-check.md) zur Aufhebung.

#### Linux-Benutzer
**Debug-Modus** (für Entwicklung: Ausgabe erscheint im Terminal, Service wird beim Schließen des Terminals beendet)

```php
php start.php start
```

**Daemon-Modus** (für Produktion: Keine Terminal-Ausgabe, Service läuft nach Schließen des Terminals weiter)

```php
php start.php start -d
```

#### Docker-Benutzer

Alle Dienste starten und an die Konsole anbinden
```php
docker-compose up
```

Dienste im Hintergrund ausführen
```php
docker-compose up -d
```

> **Tipp**
> Bei Fehlern könnten Funktionen deaktiviert sein. Siehe [Prüfung deaktivierter Funktionen](others/disable-function-check.md) zur Aufhebung.

## 3. Zugriff

Öffnen Sie im Browser `http://IP-Adresse:8787`.
