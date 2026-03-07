# Arquivo de configuração de rotas
O arquivo de configuração de rotas dos plugins está em `plugin/nome_do_plugin/config/route.php`.

## Rota padrão
Todos os caminhos de URL dos plugins de aplicação começam com `/app`, por exemplo a URL de `plugin\foo\app\controller\UserController` é `http://127.0.0.1:8787/app/foo/user`.

## Desativar rota padrão
Para desativar a rota padrão de um plugin de aplicação, adicione o seguinte na configuração de rotas:
```php
Route::disableDefaultRoute('foo');
```

## Tratar callback 404
Para definir um fallback para um plugin de aplicação, é necessário passar o nome do plugin como segundo parâmetro. Exemplo:
```php
Route::fallback(function(){
    return redirect('/');
}, 'foo');
```
