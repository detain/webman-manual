# اختبار الوحدة

## التثبيت

```php
composer require --dev phpunit/phpunit
```

## التكوين

أضف ملف التكوين `phpunit.xml` في الدليل الجذري للمشروع. يمكنك تخصيصه وفقًا لاحتياجات مشروعك. راجع [وثائق تكوين phpunit.xml](https://docs.phpunit.de/en/12.4/configuration.html).

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

## الاستخدام

أنشئ ملفًا جديدًا `tests/TestConfig.php` لاختبار تكوين التطبيق:

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
  
## التشغيل

نفّذ جميع حالات الاختبار من الدليل الجذري للمشروع:

```bash
./vendor/bin/phpunit
```

نفّذ حالة اختبار معينة من الدليل الجذري للمشروع:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

ستكون النتيجة مشابهة لما يلي:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
