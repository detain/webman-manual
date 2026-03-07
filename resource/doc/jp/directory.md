# ディレクトリ構造
```
.
├── app                           アプリケーションディレクトリ
│   ├── controller                コントローラーディレクトリ
│   ├── model                     モデルディレクトリ
│   ├── view                      ビューディレクトリ
│   ├── middleware                ミドルウェアディレクトリ
│   │   └── StaticFile.php        組み込み静的ファイルミドルウェア
│   ├── process                   カスタムプロセスディレクトリ
│   │   ├── Http.php              Httpプロセス
│   │   └── Monitor.php           モニタープロセス
│   └── functions.php             ビジネスカスタム関数をこのファイルに記述する
├── config                        設定ディレクトリ
│   ├── app.php                   アプリケーション設定
│   ├── autoload.php              ここで設定したファイルは自動的に読み込まれる
│   ├── bootstrap.php             プロセス起動時に onWorkerStart で実行されるコールバック設定
│   ├── container.php             コンテナ設定
│   ├── dependence.php            コンテナ依存設定
│   ├── database.php              データベース設定
│   ├── exception.php             例外設定
│   ├── log.php                   ログ設定
│   ├── middleware.php            ミドルウェア設定
│   ├── process.php               カスタムプロセス設定
│   ├── redis.php                 Redis設定
│   ├── route.php                 ルート設定
│   ├── server.php                ポート、プロセス数などのサーバー設定
│   ├── view.php                  ビュー設定
│   ├── static.php                静的ファイルのオン/オフおよび静的ファイルミドルウェアの設定
│   ├── translation.php           多言語設定
│   └── session.php               セッション設定
├── public                        静的リソースディレクトリ
├── runtime                       アプリケーションの実行時ディレクトリ、書き込み権限が必要
├── start.php                     サービス起動ファイル
├── vendor                        Composerでインストールされたサードパーティライブラリディレクトリ
└── support                       ライブラリアダプタ（サードパーティライブラリを含む）
    ├── Request.php               リクエストクラス
    ├── Response.php              レスポンスクラス
    ├── Setup.php                 インストールウィザードスクリプト
    └── bootstrap.php             プロセス起動後の初期化スクリプト
```
