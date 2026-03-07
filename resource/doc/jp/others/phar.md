# pharパッケージング

pharはPHPでJARに似たパッケージングファイルであり、webmanプロジェクトを単一のpharファイルにパッケージ化してデプロイすることができます。

**ここで、[fuzqing](https://github.com/fuzqing) さんのPRに非常に感謝します。**

> **注意**
> `php.ini`のphar構成オプションを無効にする必要があります。つまり、`phar.readonly = 0`を設定してください。

## コマンドラインツールのインストール
`composer require webman/console`

## パッケージング
webmanプロジェクトのルートディレクトリで `php webman build:phar` コマンドを実行します。buildディレクトリに`webman.phar`ファイルが生成されます。

> パッケージング関連の設定は `config/plugin/webman/console/app.php` にあります。

## 開始・停止関連コマンド
**開始**
`php webman.phar start` または `php webman.phar start -d`

**停止**
`php webman.phar stop`

**状態の表示**
`php webman.phar status`

**接続状態の表示**
`php webman.phar connections`

**再起動**
`php webman.phar restart` または `php webman.phar restart -d`

## 説明
* パッケージ後のプロジェクトはreloadに対応しておらず、コードの更新にはrestartによる再起動が必要です。

* パッケージサイズを抑えメモリ使用量を減らすため、`config/plugin/webman/console/app.php`の`exclude_pattern`および`exclude_files`オプションで不要なファイルを除外できます。

* webman.pharを実行すると、webman.pharが存在するディレクトリにruntimeディレクトリが生成され、ログなどの一時ファイルが格納されます。

* プロジェクトで.envファイルを使用している場合は、.envファイルをwebman.pharが存在するディレクトリに配置する必要があります。

* ユーザーがアップロードしたファイルをpharパッケージ内に保存しないでください。`phar://`プロトコルでユーザーアップロードファイルを操作することは非常に危険です（pharデシリアライゼーション脆弱性）。ユーザーアップロードファイルはpharパッケージ外のディスクに個別に保存する必要があります。下記参照。

* ビジネスでpublicディレクトリにファイルをアップロードする必要がある場合、publicディレクトリをwebman.pharが存在するディレクトリから切り離して配置する必要があります。その場合、`config/app.php`を設定してください。
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
`public_path($相対パス)`ヘルパー関数を使用して、実際のpublicディレクトリの場所を取得できます。

* webman.pharはWindowsでカスタムプロセスをサポートしていません。
