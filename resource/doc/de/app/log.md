# Logging
Die Verwendung der Log-Klasse entspricht der Verwendung der Datenbank
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Um die Log-Konfiguration des Hauptprojekts wiederzuverwenden, verwenden Sie direkt:
```php
use support\Log;
Log::info('Log-Inhalt');
// Angenommen, das Hauptprojekt hat eine Log-Konfiguration namens test
Log::channel('test')->info('Log-Inhalt');
```
