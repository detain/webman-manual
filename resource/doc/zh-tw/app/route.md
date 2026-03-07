# 路由設定檔
插件的路由設定檔在 `plugin/插件名/config/route.php`

## 預設路由
應用插件 URL 位址路徑都以 `/app` 開頭，例如 `plugin\foo\app\controller\UserController` 的 URL 位址是 `http://127.0.0.1:8787/app/foo/user`

## 禁用預設路由
如果想要禁用某個應用插件的預設路由，在路由設定中設定類似
```php
Route::disableDefaultRoute('foo');
```

## 處理 404 回調
如果想要給某個應用插件設定 fallback，需要透過第二個參數傳遞插件名，例如
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
