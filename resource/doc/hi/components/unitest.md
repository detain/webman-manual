# इकाई परीक्षण

## स्थापना

```php
composer require --dev phpunit/phpunit
```

## कॉन्फ़िगरेशन

प्रोजेक्ट रूट फ़ोल्डर में `phpunit.xml` कॉन्फ़िगरेशन फ़ाइल जोड़ें। अपने प्रोजेक्ट की आवश्यकताओं के अनुसार इसे अनुकूलित कर सकते हैं। [phpunit.xml कॉन्फ़िगरेशन दस्तावेज़](https://docs.phpunit.de/en/12.4/configuration.html) देखें।

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

## उपयोग

एप्लिकेशन कॉन्फ़िगरेशन की जांच के लिए `tests/TestConfig.php` नई फ़ाइल बनाएं:

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
  
## चलाना

प्रोजेक्ट रूट फ़ोल्डर से सभी टेस्ट केस चलाएँ:

```bash
./vendor/bin/phpunit
```

प्रोजेक्ट रूट फ़ोल्डर से निर्दिष्ट टेस्ट केस चलाएँ:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

नतीजा निम्नलिखित जैसा होगा:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
