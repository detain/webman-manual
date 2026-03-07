# Fichier de configuration

La configuration des plugins fonctionne comme celle d'un projet webman classique, mais elle s'applique en général uniquement au plugin concerné et n'a pas d'effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.app.controller_suffix` n'affecte que le suffixe du contrôleur du plugin, sans effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.app.controller_reuse` n'affecte que la réutilisation des contrôleurs par le plugin, sans effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.middleware` n'affecte que les middlewares du plugin, sans effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.view` n'affecte que les vues utilisées par le plugin, sans effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.container` n'affecte que le conteneur utilisé par le plugin, sans effet sur le projet principal.
Par exemple, la valeur de `plugin.foo.exception` n'affecte que la classe de gestion des exceptions du plugin, sans effet sur le projet principal.

En revanche, comme le routage est global, les routes configurées par un plugin s'appliquent également au routage global.

## Récupérer la configuration
Pour récupérer la configuration d'un plugin, utilisez `config('plugin.{plugin}.{config_spécifique}');`. Par exemple, pour récupérer toute la configuration de `plugin/foo/config/app.php`, utilisez `config('plugin.foo.app')`.
De même, le projet principal ou d'autres plugins peuvent utiliser `config('plugin.foo.xxx')` pour récupérer la configuration du plugin foo.

## Configurations non prises en charge
Les plugins d'application ne prennent pas en charge les configurations `server.php` et `session.php`, ni les configurations `app.request_class`, `app.public_path` et `app.runtime_path`.
