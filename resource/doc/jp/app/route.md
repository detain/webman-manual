# ルート設定ファイル
プラグインのルート設定ファイルは `plugin/プラグイン名/config/route.php` にあります。

## デフォルトルート
アプリケーションプラグインの URL パスはすべて `/app` で始まります。例えば `plugin\foo\app\controller\UserController` の URL は `http://127.0.0.1:8787/app/foo/user` です。

## デフォルトルートの無効化
特定のアプリケーションプラグインのデフォルトルートを無効化するには、ルート設定に以下を追加します：
```php
Route::disableDefaultRoute('foo');
```

## 404 コールバックの処理
アプリケーションプラグインに fallback を設定するには、第二引数でプラグイン名を渡す必要があります。例：
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
