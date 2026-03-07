# webman のインストール方法

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux：PHP + Composer 環境のインストール（既存環境の場合はスキップ可）
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **注意**
> 上記コマンドは Linux/Mac 向けです。Windows の場合は別途 PHP 環境をインストールしてください。

webman 公式が提供する[静的 PHP](https://www.workerman.net/download) を手動でダウンロードし、解凍して使用することもできます。

## 1. プロジェクトの作成

```php
composer create-project workerman/webman:~2.0
```

> **ヒント**
> エラーが出る場合は、問題のある Composer ミラーを使用している可能性があります。`composer config -g --unset repos.packagist` を実行してプロキシを解除してください。

## 2. 実行

webman ディレクトリに移動

#### Windows ユーザー
`windows.bat` をダブルクリックするか、`php windows.php` を実行して起動

> **ヒント**
> エラーが発生した場合、関数が無効化されている可能性があります。[無効化関数のチェック](others/disable-function-check.md) を参照して解除してください。

#### Linux ユーザー
**デバッグモード**（開発・デバッグ用：出力がターミナルに表示され、ターミナル終了時に webman サービスも停止します）

```php
php start.php start
```

**デーモンモード**（本番環境用：出力はターミナルに表示されず、ターミナル終了後も webman サービスは継続して動作します）

```php
php start.php start -d
```

#### Docker ユーザー

全サービスを起動してコンソールにアタッチ
```php
docker-compose up
```

バックグラウンドモードでサービスを実行
```php
docker-compose up -d
```

> **ヒント**
> エラーが発生した場合、関数が無効化されている可能性があります。[無効化関数のチェック](others/disable-function-check.md) を参照して解除してください。

## 3. アクセス

ブラウザで `http://IPアドレス:8787` にアクセスしてください。
