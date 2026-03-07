# Registo
O uso da classe de registo é semelhante ao uso da base de dados
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

Para reutilizar a configuração de registo do projeto principal, use diretamente:
```php
use support\Log;
Log::info('Conteúdo do registo');
// Assumindo que o projeto principal tem uma configuração de registo chamada test
Log::channel('test')->info('Conteúdo do registo');
```
