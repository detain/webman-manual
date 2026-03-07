# Installation

Il existe deux façons d'installer des plugins d'application :

## Installation depuis le marché des plugins
Accédez au [panneau d'administration officiel webman-admin](https://www.workerman.net/plugin/82), ouvrez la page des plugins d'application et cliquez sur le bouton d'installation pour installer le plugin correspondant.

## Installation depuis le paquet source
Téléchargez l'archive du plugin d'application depuis le marché d'applications, décompressez-la et uploadez le répertoire extrait dans `{projet principal}/plugin/` (créez manuellement le répertoire plugin s'il n'existe pas). Exécutez `php webman app-plugin:install nom_plugin` pour terminer l'installation.

Par exemple, si l'archive téléchargée s'appelle ai.zip, décompressez-la dans `{projet principal}/plugin/ai` puis exécutez `php webman app-plugin:install ai` pour terminer l'installation.


# Désinstallation

Il existe également deux façons de désinstaller des plugins d'application :

## Désinstallation depuis le marché des plugins
Accédez au [panneau d'administration officiel webman-admin](https://www.workerman.net/plugin/82), ouvrez la page des plugins d'application et cliquez sur le bouton de désinstallation pour désinstaller le plugin correspondant.

## Désinstallation depuis le paquet source
Exécutez `php webman app-plugin:uninstall nom_plugin` pour terminer la désinstallation. Ensuite, supprimez manuellement le répertoire du plugin correspondant dans `{projet principal}/plugin/`.
