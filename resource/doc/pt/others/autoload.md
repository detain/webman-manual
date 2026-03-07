# Carregamento automático

## Carregar arquivos PSR-0 via Composer
O webman segue a especificação de carregamento automático `PSR-4`. Se o teu projeto precisa carregar bibliotecas compatíveis com PSR-0, segue estes passos:

- Cria o diretório `extend` para armazenar bibliotecas PSR-0
- Edita o `composer.json` e adiciona o seguinte em `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
O resultado final será semelhante a:
![](../../assets/img/psr0.png)

- Executa `composer dumpautoload`
- Executa `php start.php restart` para reiniciar o webman (nota: é necessário reiniciar para as alterações entrarem em vigor)

## Carregar certos arquivos via Composer

- Edita o `composer.json` e adiciona em `autoload.files` os arquivos a carregar:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Executa `composer dumpautoload`
- Executa `php start.php restart` para reiniciar o webman (nota: é necessário reiniciar para as alterações entrarem em vigor)

> **Nota**
> Os arquivos configurados em `autoload.files` do composer.json são carregados antes do arranque do webman. Os arquivos carregados via `config/autoload.php` do framework são carregados após o arranque do webman.
> As alterações em arquivos de `autoload.files` do composer.json exigem restart para entrarem em vigor; reload não funciona. Os arquivos carregados via `config/autoload.php` do framework suportam hot-reload; as alterações entram em vigor após reload.

## Carregar certos arquivos via o framework
Alguns arquivos podem não cumprir a especificação PSR e não ser carregados automaticamente. Podes carregá-los configurando `config/autoload.php`, por exemplo:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Nota**
 > Em `autoload.php` estão configurados `support/Request.php` e `support/Response.php` porque existem ficheiros com o mesmo nome em `vendor/workerman/webman-framework/src/support/`. Via `autoload.php` dás prioridade às versões na raiz do projeto, permitindo personalizar estes arquivos sem alterar os de `vendor`. Se não precisas personalizá-los, podes omitir estas duas entradas.
