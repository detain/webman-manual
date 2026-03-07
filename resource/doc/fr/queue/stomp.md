# File d'attente Stomp

Stomp est un protocole simple de messagerie orienté texte (streaming) qui fournit un format de connexion interopérable, permettant aux clients STOMP d'interagir avec n'importe quel courtier de messages STOMP. [workerman/stomp](https://github.com/walkor/stomp) implémente un client Stomp, principalement utilisé dans des scénarios de files d'attente de messages tels que RabbitMQ, Apollo, ActiveMQ, etc.

## Installation
`composer require webman/stomp`

## Configuration
Le fichier de configuration se trouve dans `config/plugin/webman/stomp`

## Envoi de messages
```php
<?php
namespace app\controller;

use support\Request;
use Webman\Stomp\Client;

class Index
{
    public function queue(Request $request)
    {
        // File d'attente
        $queue = 'examples';
        // Données (lors de la transmission d'un tableau, une sérialisation manuelle est requise, par ex. avec json_encode, serialize, etc.)
        $data = json_encode(['to' => 'tom@gmail.com', 'content' => 'hello']);
        // Effectuer l'envoi
        Client::send($queue, $data);

        return response('redis queue test');
    }

}
```
> Pour des raisons de compatibilité avec d'autres projets, le composant Stomp ne fournit pas de sérialisation ni de désérialisation automatiques. Si les données envoyées sont un tableau, il faut les sérialiser manuellement et les désérialiser lors de la consommation.

## Consommation de messages
Créez le fichier `app/queue/stomp/MyMailSend.php` (le nom de la classe est libre, tant qu'il respecte la norme PSR-4).
```php
<?php
namespace app\queue\stomp;

use Workerman\Stomp\AckResolver;
use Webman\Stomp\Consumer;

class MyMailSend implements Consumer
{
    // Nom de la file d'attente
    public $queue = 'examples';

    // Nom de la connexion, correspondant à la connexion dans stomp.php
    public $connection = 'default';

    // Si la valeur est 'client', il faut appeler $ack_resolver->ack() pour indiquer au serveur que la consommation a réussi
    // Si la valeur est 'auto', il n'est pas nécessaire d'appeler $ack_resolver->ack()
    public $ack = 'auto';

    // Consommer
    public function consume($data, AckResolver $ack_resolver = null)
    {
        // Si les données sont un tableau, il faut les désérialiser manuellement
        var_export(json_decode($data, true)); // affiche ['to' => 'tom@gmail.com', 'content' => 'hello']
        // Indiquer au serveur que la consommation a réussi
        $ack_resolver->ack(); // cet appel peut être omis lorsque ack est 'auto'
    }
}
```

# Activer le protocole Stomp dans RabbitMQ
Par défaut, RabbitMQ n'active pas le protocole Stomp. Il faut exécuter la commande suivante pour l'activer :
```
rabbitmq-plugins enable rabbitmq_stomp
```
Une fois activé, le port par défaut pour Stomp est 61613.
