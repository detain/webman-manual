# Comment installer webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux : Installer PHP + Composer (passer si déjà configuré)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Note**
> La commande ci-dessus s'applique à Linux et macOS. Les utilisateurs Windows doivent installer PHP séparément.

Vous pouvez aussi télécharger manuellement la [version PHP statique](https://www.workerman.net/download) fournie par webman et l'extraire pour l'utiliser.

## 1. Créer un projet

```php
composer create-project workerman/webman:~2.0
```

> **Conseil**
> En cas d'erreur, vous utilisez peut-être un miroir Composer défectueux. Exécutez `composer config -g --unset repos.packagist` pour supprimer le proxy.

## 2. Exécution

Accédez au répertoire webman

#### Utilisateurs Windows
Double-cliquez sur `windows.bat` ou exécutez `php windows.php` pour démarrer

> **Conseil**
> En cas d'erreur, des fonctions peuvent être désactivées. Consultez [Vérification des fonctions désactivées](others/disable-function-check.md) pour les réactiver.

#### Utilisateurs Linux
**Mode debug** (pour le développement : la sortie s'affiche dans le terminal ; le service s'arrête à la fermeture du terminal)

```php
php start.php start
```

**Mode daemon** (pour la production : pas de sortie dans le terminal ; le service continue après fermeture du terminal)

```php
php start.php start -d
```

#### Utilisateurs Docker

Démarrer tous les services et les attacher à la console
```php
docker-compose up
```

Exécuter les services en arrière-plan
```php
docker-compose up -d
```

> **Conseil**
> En cas d'erreur, des fonctions peuvent être désactivées. Consultez [Vérification des fonctions désactivées](others/disable-function-check.md) pour les réactiver.

## 3. Accès

Accédez à `http://adresse-ip:8787` dans le navigateur.
