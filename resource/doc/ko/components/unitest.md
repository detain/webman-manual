# 단위 테스트

## 설치

```php
composer require --dev phpunit/phpunit
```

## 설정

프로젝트 루트 디렉토리에 설정 파일 `phpunit.xml`을 추가하세요. 프로젝트 요구 사항에 맞게 사용자 지정할 수 있습니다. [phpunit.xml 설정 옵션 문서](https://docs.phpunit.de/en/12.4/configuration.html)를 참조하세요.

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

## 사용

애플리케이션 설정 테스트용으로 `tests/TestConfig.php` 파일을 새로 만드세요:

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
  
## 실행

프로젝트 루트 디렉토리에서 모든 테스트 케이스를 실행합니다:

```bash
./vendor/bin/phpunit
```

프로젝트 루트 디렉토리에서 특정 테스트 케이스를 실행합니다:

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

결과는 다음과 유사합니다:

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
