# Log
L'uso della classe di log è simile a quello del database
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Per riutilizzare la configurazione di log del progetto principale, utilizzare direttamente:
```php
use support\Log;
Log::info('Contenuto del log');
// Supponendo che il progetto principale abbia una configurazione di log denominata test
Log::channel('test')->info('Contenuto del log');
```
