# Estrutura de diretório

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

Um plugin de aplicação possui a mesma estrutura de diretórios e arquivos de configuração do webman. Na prática, a experiência de desenvolvimento é praticamente idêntica à de uma aplicação webman comum.

Os diretórios e a nomenclatura dos plugins seguem a especificação PSR-4. Como os plugins ficam no diretório `plugin`, todos os namespaces começam com `plugin`, por exemplo `plugin\foo\app\controller\UserController`.

## Sobre o diretório api

Cada plugin tem um diretório `api`. Se sua aplicação oferece interfaces internas para serem chamadas por outras aplicações, coloque essas interfaces no diretório `api`.

Nota: as interfaces citadas aqui são de chamada de função, não de rede/HTTP.

Por exemplo, o plugin de e-mail oferece a interface `Email::send()` em `plugin/email/api/Email.php` para outras aplicações enviarem e-mails. Além disso, `plugin/email/api/Install.php` é gerado automaticamente para o mercado de plugins do webman-admin executar operações de instalação ou desinstalação.
