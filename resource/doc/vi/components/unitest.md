# Kiểm thử đơn vị

## Cài đặt

```php
composer require --dev phpunit/phpunit
```

## Cấu hình

Thêm tệp cấu hình `phpunit.xml` vào thư mục gốc của dự án. Bạn có thể tùy chỉnh theo nhu cầu dự án của mình. Xem [tài liệu cấu hình phpunit.xml](https://docs.phpunit.de/en/12.4/configuration.html).

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

## Sử dụng

Tạo tệp mới `tests/TestConfig.php` để kiểm thử cấu hình ứng dụng:

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
  
## Chạy

Chạy tất cả các trường hợp kiểm thử từ thư mục gốc của dự án:

```bash
./vendor/bin/phpunit
```

Chạy trường hợp kiểm thử cụ thể từ thư mục gốc của dự án:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

Kết quả tương tự như sau:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
