# การทดสอบหน่วย

## การติดตั้ง

```php
composer require --dev phpunit/phpunit
```

## การกำหนดค่า

เพิ่มไฟล์การกำหนดค่า `phpunit.xml` ในโฟลเดอร์รูทของโปรเจกต์ สามารถปรับแต่งได้ตามความต้องการของโปรเจกต์ ดู[เอกสารการกำหนดค่า phpunit.xml](https://docs.phpunit.de/en/12.4/configuration.html)

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

## การใช้

สร้างไฟล์ใหม่ `tests/TestConfig.php` สำหรับทดสอบการกำหนดค่าแอปพลิเคชัน:

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
  
## การรัน

รันชุดทดสอบทั้งหมดจากโฟลเดอร์รูทของโปรเจกต์:

```bash
./vendor/bin/phpunit
```

รันชุดทดสอบที่ระบุจากโฟลเดอร์รูทของโปรเจกต์:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

ผลลัพธ์คล้ายกับนี้:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
