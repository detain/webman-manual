# File d'attente Redis

File d'attente de messages basée sur Redis, prenant en charge le traitement différé des messages.

## Installation
`composer require webman/redis-queue`

## Fichier de configuration
Le fichier de configuration Redis est généré automatiquement dans `{projet-principal}/config/plugin/webman/redis-queue/redis.php`, avec un contenu similaire à :
```php
<?php
return [
    'default' => [
        'host' => 'redis://127.0.0.1:6379',
        'options' => [
            'auth' => '',         // Mot de passe, optionnel
            'db' => 0,            // Base de données
            'max_attempts'  => 5, // Nombre de tentatives après échec de consommation
            'retry_seconds' => 5, // Intervalle de réessai en secondes
        ]
    ],
];
```

### Réessai en cas d'échec de consommation
Si la consommation échoue (une exception se produit), le message est placé dans la file d'attente différée et attend le prochain réessai. Le nombre de réessais est contrôlé par `max_attempts`, l'intervalle l'est conjointement par `retry_seconds` et `max_attempts`. Par ex. si `max_attempts` vaut 5 et `retry_seconds` 10, l'intervalle du 1er réessai est `1*10` s, du 2e `2*10` s, du 3e `3*10` s, etc. jusqu'à 5 réessais. Si le nombre de réessais dépasse le réglage `max_attempts`, le message est placé dans la file d'échecs avec la clé `{redis-queue}-failed`.

## Envoi de messages (synchrone)

```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Redis;

class Index
{
    public function queue(Request $request)
    {
        // Nom de la file d'attente
        $queue = 'send-mail';
        // Données, peuvent être passées en tableau directement, pas de sérialisation
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Envoyer le message
        Redis::send($queue, $data);
        // Envoyer un message différé, traité après 60 secondes
        Redis::send($queue, $data, 60);

        return response('test file redis queue');
    }

}
```
En cas d'envoi réussi, `Redis::send()` retourne true, sinon false ou lance une exception.

> **Astuce**
> Le temps de consommation de la file différée peut varier. Par ex. si la vitesse de consommation est inférieure à celle de production, la file peut s'accumuler et retarder la consommation. Solution : lancer plus de processus consommateurs.

## Envoi de messages (asynchrone)
```php
<?php
namespace app\controller;

use support\Request;
use Webman\RedisQueue\Client;

class Index
{
    public function queue(Request $request)
    {
        // Nom de la file d'attente
        $queue = 'send-mail';
        // Données, peuvent être passées en tableau directement, pas de sérialisation
        $data = ['to' => 'tom@gmail.com', 'content' => 'hello'];
        // Envoyer le message
        Client::send($queue, $data);
        // Envoyer un message différé, traité après 60 secondes
        Client::send($queue, $data, 60);

        return response('test file redis queue');
    }

}
```
`Client::send()` ne retourne rien. C'est un envoi asynchrone et ne garantit pas une livraison à 100 % vers Redis.

> **Astuce**
> Le principe de `Client::send()` est de créer une file en mémoire locale et de synchroniser les messages vers Redis de façon asynchrone (synchronisation rapide, ~10 000 messages/s). Si le processus redémarre avant que les données de la file en mémoire soient entièrement synchronisées, des messages peuvent être perdus. L'envoi asynchrone via `Client::send()` convient aux messages non critiques.

> **Astuce**
> `Client::send()` est asynchrone et utilisable uniquement dans l'environnement d'exécution Workerman. Pour les scripts en ligne de commande, utiliser l'interface synchrone `Redis::send()`.

## Envoyer des messages depuis d'autres projets
Parfois vous devez envoyer des messages depuis d'autres projets sans pouvoir utiliser `webman\redis-queue`. Dans ce cas, vous pouvez utiliser la fonction suivante pour envoyer des messages dans la file.

```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';
    $queue_delay = '{redis-queue}-delayed';
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```

Ici, le paramètre `$redis` est l'instance Redis. Par ex. l'utilisation de l'extension redis ressemble à :

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```

## Consommation
Le fichier de configuration du processus consommateur est dans `{projet-principal}/config/plugin/webman/redis-queue/process.php`. Le répertoire des consommateurs est sous `{projet-principal}/app/queue/redis/`.

La commande `php webman redis-queue:consumer my-send-mail` génère le fichier `{projet-principal}/app/queue/redis/MyMailSend.php`.

> **Astuce**
> Cette commande nécessite l'installation du plugin [Console](../plugin/console.md). Si vous préférez ne pas l'installer, vous pouvez créer manuellement un fichier similaire au suivant :

```php
<?php

namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class MyMailSend implements Consumer
{
    // Nom de la file à consommer
    public $queue = 'send-mail';

    // Nom de connexion, correspond à la connexion dans plugin/webman/redis-queue/redis.php
    public $connection = 'default';

    // Consommation
    public function consume($data)
    {
        // Pas de désérialisation nécessaire
        var_export($data); // Affiche ['to' => 'tom@gmail.com', 'content' => 'hello']
    }
    // Callback en cas d'échec de consommation
    /* 
    $package = [
        'id' => 1357277951, // ID du message
        'time' => 1709170510, // Heure du message
        'delay' => 0, // Délai
        'attempts' => 2, // Nombre de consommations
        'queue' => 'send-mail', // Nom de la file
        'data' => ['to' => 'tom@gmail.com', 'content' => 'hello'], // Contenu du message
        'max_attempts' => 5, // Nombre max de réessais
        'error' => 'Message d\'erreur' // Message d'erreur
    ]
    */
    public function onConsumeFailure(\Throwable $e, $package)
    {
        echo "consume failure\n";
        echo $e->getMessage() . "\n";
        // Pas de désérialisation nécessaire
        var_export($package); 
    }
}
```

> **Remarque**
> La consommation est considérée réussie si aucune exception ni Error n'est lancée pendant la consommation ; sinon échec et le message entre dans la file de réessai. redis-queue n'a pas de mécanisme ack ; on peut le considérer comme un ack automatique (sans exception ni Error). Pour marquer le message actuel comme non consommé avec succès, lancez manuellement une exception pour envoyer le message dans la file de réessai. En pratique, c'est équivalent à un mécanisme ack.

> **Astuce**
> Les consommateurs supportent plusieurs serveurs et processus, et le même message ne sera pas consommé deux fois. Les messages consommés sont automatiquement retirés de la file ; pas de suppression manuelle nécessaire.

> **Astuce**
> Les processus consommateurs peuvent consommer plusieurs files différentes en parallèle. Ajouter une nouvelle file ne nécessite pas de modifier la configuration dans `process.php`. Pour ajouter un consommateur de file, il suffit d'ajouter la classe `Consumer` correspondante sous `app/queue/redis` et d'utiliser la propriété `$queue` pour indiquer le nom de la file à consommer.

> **Astuce**
> Les utilisateurs Windows doivent exécuter `php windows.php` pour démarrer webman, sinon le processus consommateur ne démarrera pas.

> **Astuce**
> Le callback onConsumeFailure est déclenché à chaque échec de consommation. Vous pouvez gérer la logique post-échec ici. (Cette fonction nécessite `webman/redis-queue>=1.3.2` et `workerman/redis-queue>=1.2.1`)

## Configurer des processus consommateurs différents pour des files différentes
Par défaut, tous les consommateurs partagent le même processus. Parfois il faut séparer la consommation de certaines files—par ex. activité à consommation lente dans un groupe de processus, activité à consommation rapide dans un autre. Pour cela, on peut diviser les consommateurs en deux répertoires, par ex. `app_path() . '/queue/redis/fast'` et `app_path() . '/queue/redis/slow'` (le namespace de la classe consommateur doit être adapté). La configuration est la suivante :
```php
return [
    ...autres configurations omises...
    
    'redis_consumer_fast'  => [ // La clé est personnalisée, pas de contrainte de format, ici redis_consumer_fast
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Répertoire des classes consommateurs
            'consumer_dir' => app_path() . '/queue/redis/fast'
        ]
    ],
    'redis_consumer_slow'  => [  // La clé est personnalisée, pas de contrainte de format, ici redis_consumer_slow
        'handler'     => Webman\RedisQueue\Process\Consumer::class,
        'count'       => 8,
        'constructor' => [
            // Répertoire des classes consommateurs
            'consumer_dir' => app_path() . '/queue/redis/slow'
        ]
    ]
];
```

Ainsi, les consommateurs rapides vont dans le répertoire `queue/redis/fast` et les lents dans `queue/redis/slow`, atteignant l'objectif d'assigner des processus consommateurs aux files.

## Configuration Redis multiple
#### Configuration
`config/plugin/webman/redis-queue/redis.php`
```php
<?php
return [
    'default' => [
        'host' => 'redis://192.168.0.1:6379',
        'options' => [
            'auth' => null,       // Mot de passe, type chaîne, optionnel
            'db' => 0,            // Base de données
            'max_attempts'  => 5, // Nombre de tentatives après échec de consommation
            'retry_seconds' => 5, // Intervalle de réessai en secondes
        ]
    ],
    'other' => [
        'host' => 'redis://192.168.0.2:6379',
        'options' => [
            'auth' => null,       // Mot de passe, type chaîne, optionnel
            'db' => 0,            // Base de données
            'max_attempts'  => 5, // Nombre de tentatives après échec de consommation
            'retry_seconds' => 5, // Intervalle de réessai en secondes
        ]
    ],
];
```

Notez qu'une configuration Redis supplémentaire avec la clé `other` a été ajoutée.

#### Envoyer des messages vers plusieurs Redis

```php
// Envoyer le message vers la file avec clé `default`
Client::connection('default')->send($queue, $data);
Redis::connection('default')->send($queue, $data);
// Équivalent à
Client::send($queue, $data);
Redis::send($queue, $data);

// Envoyer le message vers la file avec clé `other`
Client::connection('other')->send($queue, $data);
Redis::connection('other')->send($queue, $data);
```

#### Consommer depuis plusieurs Redis
Consommer les messages de la file avec clé `other` dans la configuration :
```php
namespace app\queue\redis;

use Webman\RedisQueue\Consumer;

class SendMail implements Consumer
{
    // Nom de la file à consommer
    public $queue = 'send-mail';

    // === Mettre 'other' ici pour consommer depuis la file avec clé 'other' dans la configuration ===
    public $connection = 'other';

    // Consommation
    public function consume($data)
    {
        // Pas de désérialisation nécessaire
        var_export($data);
    }
}
```

## FAQ

**Pourquoi l'erreur `Workerman\Redis\Exception: Workerman Redis Wait Timeout (600 seconds)` se produit-elle ?**

Cette erreur n'apparaît qu'avec l'interface d'envoi asynchrone `Client::send()`. L'envoi asynchrone enregistre d'abord les messages en mémoire locale puis les envoie à Redis quand le processus est inactif. Si Redis reçoit les messages plus lentement qu'ils ne sont produits, ou si le processus est trop occupé pour synchroniser les messages de la mémoire vers Redis, un engorgement peut se produire. Si des messages restent en attente plus de 600 secondes, cette erreur est déclenchée.

Solution : utilisez l'interface d'envoi synchrone `Redis::send()` pour envoyer les messages.
