# ユニットテスト

## インストール

```php
composer require --dev phpunit/phpunit
```

## 設定

プロジェクトのルートディレクトリに設定ファイル `phpunit.xml` を追加します。プロジェクトのニーズに合わせてカスタマイズできます。[phpunit.xml 設定オプションのドキュメント](https://docs.phpunit.de/en/12.4/configuration.html) をご参照ください。

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

## 使用法

アプリケーション設定のテスト用に `tests/TestConfig.php` ファイルを作成します：

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
  
## 実行

プロジェクトのルートディレクトリで全てのテストケースを実行します：

```bash
./vendor/bin/phpunit
```

プロジェクトのルートディレクトリで指定したテストケースを実行します：

```bash
./vendor/bin/phpunit tests/TestConfig.php
```

以下のような結果が表示されます：

```
PHPUnit 9.5.10 by Sebastian Bergmann and contributors.

.                                                                   1 / 1 (100%)

Time: 00:00.010, Memory: 6.00 MB

OK (1 test, 5 assertions)
```
