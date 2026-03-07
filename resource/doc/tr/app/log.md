# Günlük
Günlük sınıfının kullanımı veritabanı kullanımıyla aynıdır
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Ana projenin günlük yapılandırmasını yeniden kullanmak istiyorsanız doğrudan şunu kullanın:
```php
use support\Log;
Log::info('Günlük içeriği');
// Ana projenin test adında bir günlük yapılandırması olduğunu varsayalım
Log::channel('test')->info('Günlük içeriği');
```
