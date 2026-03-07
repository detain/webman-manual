# Phar-Packung

Phar ist eine Art von Paketdatei in PHP, ähnlich wie JAR. Sie können Phar verwenden, um Ihr Webman-Projekt in eine einzelne Phar-Datei zu verpacken und bequem zu deployen.

**Ein großes Dankeschön an [fuzqing](https://github.com/fuzqing) für den Pull Request.**

> **Hinweis**
> Sie müssen die Phar-Konfigurationsoption in `php.ini` deaktivieren, indem Sie `phar.readonly = 0` setzen.

## Installation des Befehlszeilenwerkzeugs
`composer require webman/console`

## Packen
Führen Sie den Befehl `php webman build:phar` im Stammverzeichnis des Webman-Projekts aus. Dies erstellt eine `webman.phar`-Datei im `build`-Verzeichnis.

> Die Konfiguration für das Packen befindet sich in `config/plugin/webman/console/app.php`.

## Befehle zum Starten und Beenden
**Starten**
`php webman.phar start` oder `php webman.phar start -d`

**Stoppen**
`php webman.phar stop`

**Status anzeigen**
`php webman.phar status`

**Verbindungsstatus anzeigen**
`php webman.phar connections`

**Neustarten**
`php webman.phar restart` oder `php webman.phar restart -d`

## Hinweise
* Verpackte Projekte unterstützen kein Reload; zum Aktualisieren des Codes ist ein Neustart erforderlich.

* Um übermäßige Paketgröße und Speicherverbrauch zu vermeiden, können Sie die Optionen `exclude_pattern` und `exclude_files` in `config/plugin/webman/console/app.php` setzen, um unnötige Dateien auszuschließen.

* Wenn Sie webman.phar ausführen, wird im Verzeichnis von webman.phar ein `runtime`-Verzeichnis erstellt, in dem temporäre Dateien wie Protokolle abgelegt werden.

* Wenn Ihr Projekt eine `.env`-Datei verwendet, müssen Sie die `.env`-Datei im Verzeichnis von webman.phar platzieren.

* Speichern Sie niemals vom Benutzer hochgeladene Dateien im Phar-Paket, da die Verarbeitung solcher Dateien über das `phar://`-Protokoll sehr gefährlich ist (Phar-Deserialisierungs-Schwachstelle). Benutzerhochgeladene Dateien müssen separat außerhalb des Phar-Pakets auf der Festplatte gespeichert werden. Siehe unten.

* Wenn Ihr Geschäftsvorgang das Hochladen von Dateien in das `public`-Verzeichnis erfordert, müssen Sie das `public`-Verzeichnis auslagern und im Verzeichnis von webman.phar platzieren. In diesem Fall müssen Sie `config/app.php` konfigurieren.
```php
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
Das Geschäft kann die Hilfsfunktion `public_path($relativer_pfad)` verwenden, um den tatsächlichen Speicherort des `public`-Verzeichnisses zu finden.

* webman.phar unterstützt keine benutzerdefinierten Prozesse unter Windows.
