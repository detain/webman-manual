# Rota yapılandırma dosyası
Eklentilerin rota yapılandırma dosyası `plugin/eklenti_adı/config/route.php` konumundadır.

## Varsayılan rota
Uygulama eklentilerinin tüm URL yolları `/app` ile başlar, örneğin `plugin\foo\app\controller\UserController` için URL `http://127.0.0.1:8787/app/foo/user` şeklindedir.

## Varsayılan rotayı devre dışı bırakma
Bir uygulama eklentisinin varsayılan rotasını devre dışı bırakmak için rota yapılandırmasına şunu ekleyin:
```php
Route::disableDefaultRoute('foo');
```

## 404 geri çağrısını işleme
Bir uygulama eklentisi için fallback ayarlamak üzere eklenti adını ikinci parametre olarak geçirmeniz gerekir. Örnek:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
