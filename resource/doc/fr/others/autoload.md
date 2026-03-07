# Chargement automatique

## Charger des fichiers PSR-0 via Composer
webman suit la spécification de chargement automatique `PSR-4`. Si ton projet doit charger des bibliothèques conformes PSR-0, suivi les étapes ci-dessous :

- Crée un répertoire `extend` pour stocker les bibliothèques PSR-0
- Modifie `composer.json` et ajoute le contenu suivant dans `autoload` :

```json
"psr-0" : {
    "": "extend/"
}
```
Le résultat final ressemble à ceci :
![](../../assets/img/psr0.png)

- Exécute `composer dumpautoload`
- Exécute `php start.php restart` pour redémarrer webman (remarque : le redémarrage est indispensable pour appliquer les changements)

## Charger certains fichiers via Composer

- Modifie `composer.json` et ajoute sous `autoload.files` les fichiers à charger :
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Exécute `composer dumpautoload`
- Exécute `php start.php restart` pour redémarrer webman (remarque : le redémarrage est indispensable pour appliquer les changements)

> **Remarque**
> Les fichiers configurés dans `autoload.files` de composer.json sont chargés avant le démarrage de webman. Les fichiers chargés via `config/autoload.php` du framework sont chargés après le démarrage de webman.
> Pour les fichiers dans `autoload.files` de composer.json, tout changement exige un redémarrage (restart) pour prendre effet ; reload ne suffit pas. Les fichiers chargés via `config/autoload.php` du framework supportent le hot-reload ; un simple reload suffit pour appliquer les changements.

## Charger certains fichiers via le framework
Certains fichiers peuvent ne pas respecter la spécification PSR et ne peuvent pas être chargés automatiquement. Tu peux les charger en configurant `config/autoload.php`, par exemple :
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Remarque**
 > Dans `autoload.php`, le chargement de `support/Request.php` et `support/Response.php` est configuré car des fichiers du même nom existent dans `vendor/workerman/webman-framework/src/support/`. En passant par `autoload.php`, les versions à la racine du projet sont chargées en priorité, ce qui permet de personnaliser ces fichiers sans modifier ceux du dossier `vendor`. Si tu n’as pas besoin de les personnaliser, tu peux ignorer ces deux entrées.
