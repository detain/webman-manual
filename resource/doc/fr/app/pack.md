# Empaquetage

Par exemple, pour empaqueter le plugin de l'application foo :

* Définissez le numéro de version dans `plugin/foo/config/app.php` (**important**)
* Supprimez les fichiers inutiles dans `plugin/foo`, en particulier les fichiers temporaires de test de la fonctionnalité d'upload sous `plugin/foo/public`
* Si votre projet inclut la création de tables de base de données et des opérations similaires, configurez correctement `plugin/foo/install.sql`. Voir la [section base de données](database.md)
* Si votre projet a sa propre configuration de base de données et Redis, supprimez d'abord ces configurations. Elles doivent être proposées via un assistant d'installation lors du premier accès à l'application (vous devez l'implémenter vous-même), pour que l'administrateur puisse les remplir manuellement et les générer.
* Si votre projet inclut des menus backend webman admin, configurez `plugin/foo/config/menu.php` afin que ces menus soient définis automatiquement lors de l'installation du plugin. Voir [webman-admin importer les menus](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Restaurez les autres fichiers qui doivent être rétablis dans leur état d'origine
* Une fois ces opérations effectuées, accédez au répertoire `{projet principal}/plugin/`
* Utilisateurs Linux : exécutez la commande `zip -r foo.zip foo` pour générer foo.zip
* Utilisateurs Windows : cliquez droit sur le dossier foo et sélectionnez « Compresser en fichier ZIP » pour générer foo.zip

**foo.zip est le fichier empaqueté. Reportez-vous au chapitre suivant [Publier le plugin](publish.md)**
