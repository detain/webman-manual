# Plugins de base

Les plugins de base sont généralement des composants communs, typiquement installés via Composer, le code étant placé dans le répertoire vendor. Lors de l'installation, des configurations personnalisées (middleware, processus, routes, etc.) peuvent être automatiquement copiées dans le répertoire `{projet principal}config/plugin`. Webman reconnaîtra automatiquement la configuration de ce répertoire et la fusionnera avec la configuration principale, permettant ainsi aux plugins d'intervenir à n'importe quelle phase du cycle de vie de webman.

Pour en savoir plus, consultez [Création de plugins de base](create.md).
