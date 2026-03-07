# Structure du répertoire
```
.
├── app                           Répertoire des applications
│   ├── controller                Répertoire des contrôleurs
│   ├── model                     Répertoire des modèles
│   ├── view                      Répertoire des vues
│   ├── middleware                Répertoire des middlewares
│   │   └── StaticFile.php        Middleware de fichier statique intégré
│   ├── process                   Répertoire des processus personnalisés
│   │   ├── Http.php              Processus Http
│   │   └── Monitor.php           Processus de surveillance
│   └── functions.php             Les fonctions commerciales personnalisées sont écrites dans ce fichier
├── config                        Répertoire de configuration
│   ├── app.php                   Configuration de l'application
│   ├── autoload.php              Les fichiers configurés ici seront chargés automatiquement
│   ├── bootstrap.php             Configuration du callback exécuté lors du démarrage du processus (onWorkerStart)
│   ├── container.php             Configuration du conteneur
│   ├── dependence.php            Configuration des dépendances du conteneur
│   ├── database.php              Configuration de la base de données
│   ├── exception.php             Configuration des exceptions
│   ├── log.php                   Configuration des journaux
│   ├── middleware.php            Configuration des middlewares
│   ├── process.php               Configuration des processus personnalisés
│   ├── redis.php                 Configuration de Redis
│   ├── route.php                 Configuration des routes
│   ├── server.php                Configuration du serveur (ports, nombre de processus, etc.)
│   ├── view.php                  Configuration des vues
│   ├── static.php                Activation des fichiers statiques et configuration du middleware
│   ├── translation.php           Configuration multilingue
│   └── session.php               Configuration de la session
├── public                        Répertoire des ressources statiques
├── runtime                       Répertoire d'exécution de l'application, nécessite des permissions d'écriture
├── start.php                     Fichier de démarrage du service
├── vendor                        Répertoire des bibliothèques tierces installées par Composer
└── support                       Adaptation des bibliothèques (y compris les tierces)
    ├── Request.php               Classe de requête
    ├── Response.php              Classe de réponse
    ├── Setup.php                 Script de l'assistant d'installation
    └── bootstrap.php             Script d'initialisation après le démarrage du processus
```
