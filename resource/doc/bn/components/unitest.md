# ইউনিট টেস্ট

## ইনস্টলেশন

```php
composer require --dev phpunit/phpunit
```

## কনফিগারেশন

প্রজেক্টের রুট ফোল্ডারে `phpunit.xml` কনফিগারেশন ফাইল যোগ করুন। প্রোজেক্টের প্রয়োজনের উপর ভিত্তি করে কাস্টমাইজ করতে পারবেন। [phpunit.xml কনফিগারেশন ডকুমেন্টেশন](https://docs.phpunit.de/en/12.4/configuration.html) দেখুন।

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

## ব্যবহার

অ্যাপ্লিকেশন কনফিগারেশন পরীক্ষার জন্য `tests/TestConfig.php` নতুন ফাইল তৈরি করুন:

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
  
## চালান

প্রজেক্টের রুট ফোল্ডার থেকে সব টেস্ট কেস চালান:

```bash
./vendor/bin/phpunit
```

প্রজেক্টের রুট ফোল্ডার থেকে নির্দিষ্ট টেস্ট কেস চালান:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

ফলাফল এরকম দেখা যাবে:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
