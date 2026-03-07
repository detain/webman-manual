# Paketierung

Beispielsweise das Paketieren des Foo-Plug-ins:

* Setzen Sie die Versionsnummer in `plugin/foo/config/app.php` (**wichtig**)
* Löschen Sie unnötige Dateien in `plugin/foo`, insbesondere temporäre Dateien zur Überprüfung der Upload-Funktion unter `plugin/foo/public`
* Wenn Ihr Projekt Datenbanktabellen-Erstellung usw. enthält, konfigurieren Sie `plugin/foo/install.sql` entsprechend. Siehe [Datenbank](database.md)
* Wenn Ihr Projekt eigene Datenbank- und Redis-Konfiguration hat, entfernen Sie diese zuerst. Diese sollten bei erstem Zugriff auf die Anwendung über einen Installations-Assistenten abgefragt werden (Sie müssen dies selbst implementieren), damit der Administrator sie manuell ausfüllen und generieren kann.
* Wenn Ihr Projekt webman-admin-Backend-Menüs enthält, konfigurieren Sie `plugin/foo/config/menu.php`, damit diese Menüs bei der Plugin-Installation automatisch gesetzt werden. Siehe [webman-admin Menüs importieren](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Stellen Sie alle anderen Dateien wieder her, die in ihren ursprünglichen Zustand zurückversetzt werden müssen
* Nach Abschluss der oben genannten Schritte wechseln Sie in das Verzeichnis `{Hauptprojekt}/plugin/`
* Linux-Benutzer: Führen Sie `zip -r foo.zip foo` aus, um foo.zip zu erstellen
* Windows-Benutzer: Klicken Sie mit der rechten Maustaste auf den Ordner foo und wählen Sie „Als ZIP-Datei komprimieren“, um foo.zip zu erstellen

**foo.zip ist die gepackte Datei. Siehe das nächste Kapitel [Plugin veröffentlichen](publish.md)**
