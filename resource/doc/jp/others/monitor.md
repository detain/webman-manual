# プロセス監視
webmanには標準でmonitorプロセスが組み込まれており、次の2つの機能をサポートします。
1. ファイルの更新を監視し、新しいビジネスコードを自動でリロード（主に開発時に使用）
2. 全プロセスのメモリ使用量を監視し、プロセスが `php.ini` の `memory_limit` を超えそうになると自動で安全に再起動（業務への影響なし）

## 監視設定
`config/process.php` の `monitor` 設定：
```php

global $argv;

return [
    // ファイル更新の検出と自動リロード
    'monitor' => [
        'handler' => process\Monitor::class,
        'reloadable' => false,
        'constructor' => [
            // これらのディレクトリを監視
            'monitorDir' => array_merge([    // 監視対象のディレクトリ
                app_path(),
                config_path(),
                base_path() . '/process',
                base_path() . '/support',
                base_path() . '/resource',
                base_path() . '/.env',
            ], glob(base_path() . '/plugin/*/app'), glob(base_path() . '/plugin/*/config'), glob(base_path() . '/plugin/*/api')),
            // これらの拡張子のファイルを監視
            'monitorExtensions' => [
                'php', 'html', 'htm', 'env'
            ],
            'options' => [
                'enable_file_monitor' => !in_array('-d', $argv) && DIRECTORY_SEPARATOR === '/', // ファイル監視を有効にするか
                'enable_memory_monitor' => DIRECTORY_SEPARATOR === '/',                      // メモリ監視を有効にするか
            ]
        ]
    ]
];
```
`monitorDir` は更新を監視するディレクトリを指定します（監視対象のファイル数は多すぎないようにすること）。
`monitorExtensions` は `monitorDir` 内で監視するファイルの拡張子を指定します。
`options.enable_file_monitor` が `true` の場合、ファイル更新監視が有効になります（Linuxではデバッグモード実行時、デフォルトで有効）。
`options.enable_memory_monitor` が `true` の場合、メモリ監視が有効になります（Windowsではサポートされていません）。

> **ヒント**
> Windowsでは、ファイル監視を有効にするには `windows.bat` または `php windows.php` を実行する必要があります。
