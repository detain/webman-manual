# Journal
L'utilisation de la classe de journal est similaire à celle de la base de données
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Pour réutiliser la configuration de journal du projet principal, utilisez directement :
```php
use support\Log;
Log::info('Contenu du journal');
// En supposant que le projet principal a une configuration de journal nommée test
Log::channel('test')->info('Contenu du journal');
```
