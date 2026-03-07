# Konfigurationsdatei

Die Konfiguration von Plugins entspricht der eines normalen webman-Projekts. Allerdings gilt die Plugin-Konfiguration in der Regel nur für das jeweilige Plugin und hat auf das Hauptprojekt keinen Einfluss.
So betrifft der Wert von `plugin.foo.app.controller_suffix` nur die Controller-Suffixe des Plugins und hat keine Auswirkung auf das Hauptprojekt.
So betrifft der Wert von `plugin.foo.app.controller_reuse` nur, ob das Plugin Controller wiederverwendet, und hat keine Auswirkung auf das Hauptprojekt.
So betrifft der Wert von `plugin.foo.middleware` nur die Middleware des Plugins und hat keine Auswirkung auf das Hauptprojekt.
So betrifft der Wert von `plugin.foo.view` nur die vom Plugin verwendeten Views und hat keine Auswirkung auf das Hauptprojekt.
So betrifft der Wert von `plugin.foo.container` nur den vom Plugin verwendeten Container und hat keine Auswirkung auf das Hauptprojekt.
So betrifft der Wert von `plugin.foo.exception` nur die Exception-Behandlungsklasse des Plugins und hat keine Auswirkung auf das Hauptprojekt.

Da das Routing jedoch global ist, wirken sich die vom Plugin konfigurierten Routen ebenfalls global aus.

## Konfiguration abrufen
Um die Konfiguration eines Plugins abzurufen, verwenden Sie `config('plugin.{plugin}.{spezifische_config}');`. Beispiel: Um die gesamte Konfiguration von `plugin/foo/config/app.php` abzurufen, verwenden Sie `config('plugin.foo.app')`.
Ebenso können das Hauptprojekt oder andere Plugins mit `config('plugin.foo.xxx')` die Konfiguration des foo-Plugins abrufen.

## Nicht unterstützte Konfigurationen
Anwendungs-Plugins unterstützen weder die `server.php`- und `session.php`-Konfiguration noch die Konfigurationen `app.request_class`, `app.public_path` und `app.runtime_path`.
