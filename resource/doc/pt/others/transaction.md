# Uso correto de transações

O uso de transações de banco de dados no webman é igual ao dos outros frameworks. Aqui estão os pontos a notar.

## Estrutura do código

A estrutura do código é a mesma dos outros frameworks (por exemplo Laravel, think-orm similar):

```php
Db::beginTransaction();
try {
    // ..lógica de negócio omitida...
    
    Db::commit();
} catch (\Throwable $exception) {
    Db::rollBack();
}
```

**Importante:** deve usar **obrigatoriamente** `\Throwable` e **não** `\Exception`, pois a lógica de negócio pode disparar um `Error`, que não herda de `Exception`.

## Conexão com o banco de dados

Ao operar com modelos dentro de uma transação, preste atenção se o modelo tem conexão configurada. Se o modelo define uma conexão, é preciso especificá-la ao iniciar a transação; caso contrário a transação não terá efeito (think-orm similar). Exemplo:

```php
<?php

namespace app\model;
use support\Model;

class User extends Model
{

    // Conexão definida para o modelo
    protected $connection = 'mysql';

    protected $table = 'users';

    protected $primaryKey = 'id';

}
```

Quando o modelo define uma conexão, é preciso especificá-la para begin, commit e rollback:

```php
Db::connection('mysql')->beginTransaction();
try {
    // Lógica de negócio
    $user = new User;
    $user->name = 'webman';
    $user->save();
    Db::connection('mysql')->commit();
} catch (\Throwable $exception) {
    Db::connection('mysql')->rollBack();
}
```

## Encontrar pedidos com transações não confirmadas

Às vezes, um bug no código de negócio deixa uma transação sem confirmar. Para localizar rapidamente qual método do controller está em causa, pode instalar o componente `webman/log`. O componente verifica automaticamente após cada pedido se há transações não confirmadas e regista-as nos logs. A palavra-chave no log é `Uncommitted transactions`.

**Instalação do webman/log**

`composer require webman/log`

> **Nota**
> Após a instalação é necessário fazer **restart**; reload não surte efeito.
