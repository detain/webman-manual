# ディレクトリ構造

```
plugin/
└── foo
    ├── app
    │   ├── controller
    │   │   └── IndexController.php
    │   ├── exception
    │   │   └── Handler.php
    │   ├── functions.php
    │   ├── middleware
    │   ├── model
    │   └── view
    │       └── index
    │           └── index.html
    ├── config
    │   ├── app.php
    │   ├── autoload.php
    │   ├── container.php
    │   ├── database.php
    │   ├── exception.php
    │   ├── log.php
    │   ├── middleware.php
    │   ├── process.php
    │   ├── redis.php
    │   ├── route.php
    │   ├── static.php
    │   ├── thinkorm.php
    │   ├── translation.php
    │   └── view.php
    ├── public
    └── api
```

アプリケーションプラグインは、webman と同じディレクトリ構造と設定ファイルを持ちます。実務上、開発体験は通常の webman アプリケーション開発とほぼ同じです。

プラグインのディレクトリと命名は PSR-4 規約に従います。プラグインはすべて `plugin` ディレクトリ以下に配置されるため、名前空間は `plugin` で始まります。例：`plugin\foo\app\controller\UserController`。

## api ディレクトリについて

各プラグインには `api` ディレクトリがあります。他アプリケーションから呼び出す内部インターフェースを提供する場合は、そのインターフェースを `api` ディレクトリに配置してください。

注意：ここでいうインターフェースは、関数呼び出しのインターフェースであり、ネットワーク／HTTP インターフェースではありません。

例えば、メールプラグインは `plugin/email/api/Email.php` に `Email::send()` インターフェースを提供し、他アプリケーションからメール送信時に呼び出されます。また、`plugin/email/api/Install.php` は自動生成され、webman-admin プラグインマーケットがインストール／アンインストール処理を実行する際に使われます。
