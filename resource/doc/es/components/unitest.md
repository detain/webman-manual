# Pruebas unitarias

## Instalación

```php
composer require --dev phpunit/phpunit
```

## Configuración

Añada el archivo de configuración `phpunit.xml` en el directorio raíz del proyecto. Puede personalizarlo según las necesidades de su proyecto. Consulte la [documentación de configuración de phpunit.xml](https://docs.phpunit.de/en/12.4/configuration.html).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
    bootstrap="support/bootstrap.php"
    cacheDirectory=".phpunit.cache"
    executionOrder="depends,defects"
    shortenArraysForExportThreshold="10"
    requireCoverageMetadata="false"
    beStrictAboutCoverageMetadata="true"
    beStrictAboutOutputDuringTests="true"
    displayDetailsOnPhpunitDeprecations="true"
    failOnPhpunitDeprecation="true"
    failOnRisky="true"
    failOnWarning="true"
    colors="true">
    <testsuites>
        <testsuite name="tests">
            <directory>./tests</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory suffix=".php">./app</directory>
        </include>
    </source>
</phpunit>

```

## Uso

Cree el archivo `tests/TestConfig.php` para probar la configuración de la aplicación:

```php
<?php
use PHPUnit\Framework\TestCase;

class TestConfig extends TestCase
{
    public function testAppConfig()
    {
        $config = config('app');
        self::assertIsArray($config);
        self::assertArrayHasKey('debug', $config);
        self::assertIsBool($config['debug']);
        self::assertArrayHasKey('default_timezone', $config);
        self::assertIsString($config['default_timezone']);
    }
}
```
  
## Ejecución

Ejecute todos los tests desde el directorio raíz del proyecto:

```bash
./vendor/bin/phpunit
```

Ejecute un test específico desde el directorio raíz del proyecto:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

El resultado será similar a:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
