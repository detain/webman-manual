# Processo básico de criação e publicação de plugins

## Princípio
1. Tomando um plugin cross-origin como exemplo: o plugin é composto por três partes – o arquivo do middleware cross-origin, o arquivo de configuração `middleware.php` e um `Install.php` gerado automaticamente por comando.
2. Usamos um comando para empacotar esses três arquivos e publicá-los no Composer.
3. Quando o usuário instala o plugin cross-origin pelo Composer, `Install.php` copia o arquivo do middleware e a configuração para `{projeto principal}/config/plugin` para o webman carregar, ativando assim a configuração cross-origin.
4. Quando o usuário desinstala o plugin pelo Composer, `Install.php` remove os arquivos do middleware e a configuração correspondentes, realizando a desinstalação automática do plugin.

## Especificações
1. O nome do plugin é composto por `vendor` e `nome do plugin`, ex. `webman/push`, correspondente ao nome do pacote Composer.
2. Os arquivos de configuração do plugin ficam em `config/plugin/vendor/nome do plugin/` (o comando console cria o diretório de configuração automaticamente). Se o plugin não precisa de configuração, o diretório criado automaticamente deve ser removido.
3. O diretório de configuração do plugin aceita apenas: `app.php` (config principal), `bootstrap.php` (inicialização de processos), `route.php` (rotas), `middleware.php` (middleware), `process.php` (processos personalizados), `database.php` (banco de dados), `redis.php` (Redis), `thinkorm.php` (thinkorm). Estes são reconhecidos automaticamente pelo webman.
4. Acesso à configuração: `config('plugin.vendor.nome do plugin.arquivo de config.item específico');`, ex. `config('plugin.webman.push.app.app_key')`.
5. Se o plugin tem configuração própria de BD: para `illuminate/database` usar `Db::connection('plugin.vendor.nome do plugin.conexão')`, para `thinkorm` usar `Db::connect('plugin.vendor.nome do plugin.conexão')`.
6. Se um plugin coloca arquivos de negócio em `app/`, deve evitar conflitos com o projeto principal e outros plugins.
7. Plugins devem evitar copiar arquivos ou diretórios para o projeto principal. Ex.: no plugin cross-origin apenas a configuração é copiada; os arquivos do middleware ficam em `vendor/webman/cros/src`.
8. Recomenda-se PascalCase para os namespaces dos plugins, ex. `Webman/Console`.

## Exemplo

**Instalar a linha de comando `webman/console`**

`composer require webman/console`

### Criar um plugin

Suponha que o plugin a criar se chame `foo/admin` (nome do projeto Composer a publicar, deve estar em minúsculas). Executar:

`php webman plugin:create --name=foo/admin`

Isso cria `vendor/foo/admin` para os arquivos do plugin e `config/plugin/foo/admin` para a configuração.

> Nota
> `config/plugin/foo/admin` suporta: `app.php`, `bootstrap.php`, `route.php`, `middleware.php`, `process.php`, `database.php`, `redis.php`, `thinkorm.php`. Formato igual ao webman, merge automático.
> Acesso com o prefixo `plugin`, ex. `config('plugin.foo.admin.app')`.


### Exportar o plugin

Após concluir o desenvolvimento, executar:

`php webman plugin:export --name=foo/admin`

Exportar

> Explicação
> A exportação copia `config/plugin/foo/admin` para `vendor/foo/admin/src` e gera `Install.php`, executado na instalação e desinstalação.
> Instalação padrão: copia a configuração de `vendor/foo/admin/src` para `config/plugin` do projeto.
> Desinstalação padrão: remove os arquivos de configuração do plugin em `config/plugin` do projeto.
> Pode-se modificar `Install.php` para adicionar lógica personalizada na instalação e desinstalação.

### Enviar o plugin
* Suponha que você já tenha conta no [GitHub](https://github.com) e [Packagist](https://packagist.org).
* Crie um repositório `admin` no [GitHub](https://github.com) e envie o código, ex. `https://github.com/seu-usuario/admin`.
* Acesse `https://github.com/seu-usuario/admin/releases/new` e crie um release, ex. `v1.0.0`.
* No [Packagist](https://packagist.org) clique em `Submit` e envie `https://github.com/seu-usuario/admin` para publicar o plugin.

> **Dica**
> Se o Packagist reportar conflito de nome, use outro vendor, ex. alterar `foo/admin` para `myfoo/admin`.

Em atualizações: envie o código para o GitHub, crie um novo release em `https://github.com/seu-usuario/admin/releases/new` e clique em `Update` em `https://packagist.org/packages/foo/admin`.

## Adicionar comandos ao plugin
Alguns plugins precisam de comandos personalizados. Ex.: ao instalar `webman/redis-queue`, o projeto ganha o comando `redis-queue:consumer`. Executando `php webman redis-queue:consumer send-mail` gera rapidamente uma classe consumer `SendMail.php`, o que acelera o desenvolvimento.

Para adicionar o comando `foo-admin:add` ao plugin `foo/admin`:

### Criar um comando

**Criar o arquivo `vendor/foo/admin/src/FooAdminAddCommand.php`**

```php
<?php

namespace Foo\Admin;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

class FooAdminAddCommand extends Command
{
    protected static $defaultName = 'foo-admin:add';
    protected static $defaultDescription = 'Descrição do comando';

    /**
     * @return void
     */
    protected function configure()
    {
        $this->addArgument('name', InputArgument::REQUIRED, 'Add name');
    }

    /**
     * @param InputInterface $input
     * @param OutputInterface $output
     * @return int
     */
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('name');
        $output->writeln("Admin add $name");
        return self::SUCCESS;
    }

}
```

> **Nota**
> Para evitar conflitos de comandos entre plugins, usar o formato `vendor-plugin:comando`. Ex.: todos os comandos de `foo/admin` devem ter o prefixo `foo-admin:`, como `foo-admin:add`.

### Adicionar configuração
**Criar `config/plugin/foo/admin/command.php`**

```php
<?php

use Foo\Admin\FooAdminAddCommand;

return [
    FooAdminAddCommand::class,
    // Adicionar mais se necessário...
];
```

> **Dica**
> `command.php` registra os comandos personalizados do plugin. Cada entrada é uma classe de comando. `webman/console` carrega-os automaticamente. Ver [Comandos console](console.md).

### Executar a exportação
Executar `php webman plugin:export --name=foo/admin` para exportar e publicar no Packagist. Após instalar `foo/admin`, o comando `foo-admin:add` estará disponível. Ao executar `php webman foo-admin:add jerry` será impresso `Admin add jerry`.
