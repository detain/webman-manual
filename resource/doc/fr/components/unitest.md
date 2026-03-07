# Tests unitaires

## Installation

```php
composer require --dev phpunit/phpunit
```

## Configuration

Créez le fichier de configuration `phpunit.xml` à la racine du projet. Vous pouvez le personnaliser selon les besoins de votre projet. Consultez la [documentation de configuration phpunit.xml](https://docs.phpunit.de/en/12.4/configuration.html).

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

## Utilisation

Créez le fichier `tests/TestConfig.php` pour tester la configuration de l'application :

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
  
## Exécution

Exécutez tous les tests depuis la racine du projet :

```bash
./vendor/bin/phpunit
```

Exécutez un test spécifique depuis la racine du projet :

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

Le résultat sera similaire à :

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
