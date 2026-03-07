# Flux d'exécution

## Flux de démarrage du processus

Après l'exécution de `php start.php start`, le flux d'exécution est le suivant :

1. Charger les configurations sous config/
2. Configurer le Worker avec des options comme `pid_file`, `stdout_file`, `log_file`, `max_package_size`, etc.
3. Créer le processus webman et écouter sur le port (par défaut : 8787)
4. Créer les processus personnalisés selon la configuration
5. Après le démarrage du processus webman et des processus personnalisés, la logique suivante est exécutée (tout dans `onWorkerStart`) :
   ① Charger les fichiers définis dans `config/autoload.php`, comme `app/functions.php`
   ② Charger les middlewares définis dans `config/middleware.php` (y compris `config/plugin/*/*/middleware.php`)
   ③ Exécuter la méthode `start` des classes définies dans `config/bootstrap.php` (y compris `config/plugin/*/*/bootstrap.php`) pour initialiser des modules, comme la connexion à la base de données Laravel
   ④ Charger les routes définies dans `config/route.php` (y compris `config/plugin/*/*/route.php`)

## Flux de traitement des requêtes

1. Vérifier si l'URL de la requête correspond à un fichier statique sous public. Si oui, renvoyer le fichier (fin de la requête). Sinon, passer à l'étape 2.
2. Déterminer si l'URL correspond à une route. Si non, passer à l'étape 3 ; si oui, passer à l'étape 4.
3. Vérifier si la route par défaut est désactivée. Si oui, renvoyer 404 (fin de la requête). Sinon, passer à l'étape 4.
4. Trouver les middlewares du contrôleur correspondant à la requête, exécuter les opérations préalables des middlewares dans l'ordre (phase requête du modèle oignon), exécuter la logique métier du contrôleur, exécuter les opérations ultérieures des middlewares (phase réponse du modèle oignon) et terminer la requête. (Voir le [modèle oignon des middlewares](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
