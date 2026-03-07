# ログ
ログクラスの使用法はデータベースの使用法と同様です
```php
use support\Log;
Log::channel('plugin.admin.default')->info('test');
```

メインプロジェクトのログ設定を再利用したい場合は、そのまま以下のように使用します
```php
use support\Log;
Log::info('ログ内容');
// メインプロジェクトに test というログ設定があると仮定
Log::channel('test')->info('ログ内容');
```
