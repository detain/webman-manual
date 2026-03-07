# Como instalar o webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: Instalar PHP + Composer (pular se já configurado)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Nota**
> O comando acima aplica-se a Linux e macOS. Usuários do Windows precisam instalar PHP separadamente.

Você também pode baixar manualmente a [versão estática do PHP](https://www.workerman.net/download) fornecida pelo webman e extrair para usar.

## 1. Criar projeto

```php
composer create-project workerman/webman:~2.0
```

> **Dica**
> Se houver erros, talvez você esteja usando um espelho do Composer com problemas. Execute `composer config -g --unset repos.packagist` para remover o proxy.

## 2. Executar

Navegue até o diretório webman

#### Usuários Windows
Clique duas vezes em `windows.bat` ou execute `php windows.php` para iniciar

> **Dica**
> Se houver erros, algumas funções podem estar desativadas. Consulte [Verificação de funções desativadas](others/disable-function-check.md) para reativá-las.

#### Usuários Linux
**Modo debug** (para desenvolvimento: a saída aparece no terminal; o serviço para ao fechar o terminal)

```php
php start.php start
```

**Modo daemon** (para produção: sem saída no terminal; o serviço continua após fechar o terminal)

```php
php start.php start -d
```

#### Usuários Docker

Iniciar todos os serviços e anexar ao console
```php
docker-compose up
```

Executar serviços em segundo plano
```php
docker-compose up -d
```

> **Dica**
> Se houver erros, algumas funções podem estar desativadas. Consulte [Verificação de funções desativadas](others/disable-function-check.md) para reativá-las.

## 3. Acesso

Acesse `http://endereço-ip:8787` no navegador.
