# 自動読み込み

## Composer で PSR-0 規格のファイルを読み込む
webman は `PSR-4` 自動読み込み規格に従います。プロジェクトで PSR-0 規格のライブラリを読み込む必要がある場合は、以下の手順に従ってください。

- `extend` ディレクトリを作成し、PSR-0 規格のライブラリを格納します
- `composer.json` を編集し、`autoload` に以下を追加します：

```json
"psr-0" : {
    "": "extend/"
}
```
最終的な結果は以下のようになります：
![](../../assets/img/psr0.png)

- `composer dumpautoload` を実行します
- `php start.php restart` を実行して webman を再起動します（注意：変更を反映するには再起動が必要です）

## Composer で特定のファイルを読み込む

- `composer.json` を編集し、`autoload.files` に読み込むファイルを追加します：
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- `composer dumpautoload` を実行します
- `php start.php restart` を実行して webman を再起動します（注意：変更を反映するには再起動が必要です）

> **ヒント**
> composer.json の `autoload.files` で設定したファイルは webman 起動前に読み込まれます。一方、フレームワークの `config/autoload.php` で読み込むファイルは webman 起動後に読み込まれます。
> composer.json の `autoload.files` で読み込むファイルの変更を反映するには restart が必要で、reload では反映されません。`config/autoload.php` で読み込むファイルはホットリロードに対応しており、変更は reload で反映されます。

## フレームワークで特定のファイルを読み込む
PSR 規格に準拠しないファイルがあり自動読み込みできない場合は、`config/autoload.php` を設定して読み込むことができます。例：
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **ヒント**
 > `autoload.php` で `support/Request.php` と `support/Response.php` の読み込みが設定されています。これは `vendor/workerman/webman-framework/src/support/` にも同名のファイルがあるためです。`autoload.php` によりプロジェクトルートのファイルを優先的に読み込むことで、`vendor` 内を変更せずにこれらをカスタマイズできます。カスタマイズが不要な場合は、この 2 つの設定を省略して構いません。
