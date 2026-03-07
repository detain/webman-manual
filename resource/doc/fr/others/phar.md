# Emballer en Phar

Phar est un type de fichier d'emballage similaire à JAR dans PHP. Vous pouvez utiliser Phar pour empaqueter votre projet webman dans un seul fichier Phar pour une distribution pratique.

**Un grand merci à [fuzqing](https://github.com/fuzqing) pour sa PR.**

> **Remarque**
> Il est nécessaire de désactiver l'option de configuration Phar dans `php.ini`, c'est-à-dire de définir `phar.readonly = 0`.

## Installation de l'outil en ligne de commande
`composer require webman/console`

## Empaquetage
Exécutez la commande `php webman build:phar` dans le répertoire racine du projet webman. Cela générera un fichier `webman.phar` dans le répertoire `build`.

> La configuration relative à l'empaquetage se trouve dans `config/plugin/webman/console/app.php`.

## Commandes de démarrage et d'arrêt
**Démarrer**
`php webman.phar start` ou `php webman.phar start -d`

**Arrêter**
`php webman.phar stop`

**Vérifier l'état**
`php webman.phar status`

**Vérifier l'état de connexion**
`php webman.phar connections`

**Redémarrer**
`php webman.phar restart` ou `php webman.phar restart -d`

## Remarques
* Les projets empaquetés ne prennent pas en charge le reload ; il faut redémarrer pour mettre à jour le code.

* Pour éviter une taille de paquet excessive et une consommation mémoire trop élevée, vous pouvez configurer les options `exclude_pattern` et `exclude_files` dans `config/plugin/webman/console/app.php` pour exclure les fichiers inutiles.

* L'exécution de webman.phar générera un répertoire `runtime` dans le répertoire où se trouve webman.phar, utilisé pour stocker des fichiers temporaires tels que les journaux.

* Si votre projet utilise un fichier .env, vous devez placer le fichier .env dans le même répertoire que webman.phar.

* Ne stockez jamais les fichiers téléversés par les utilisateurs dans le paquet Phar, car opérer sur ces fichiers via le protocole `phar://` est très dangereux (vulnérabilité de désérialisation Phar). Les fichiers téléversés par les utilisateurs doivent être stockés séparément sur le disque, en dehors du paquet Phar. Voir ci-dessous.

* Si votre application a besoin de téléverser des fichiers dans le répertoire public, ce répertoire doit être séparé et placé dans le même répertoire que webman.phar. Dans ce cas, vous devez configurer `config/app.php`.
```php
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
L'application peut utiliser la fonction d'aide `public_path($chemin_relatif)` pour trouver l'emplacement réel du répertoire public.

* webman.phar ne prend pas en charge les processus personnalisés sous Windows.
