# Installazione

Esistono due modi per installare i plugin dell'applicazione:

## Installazione dal mercato dei plugin
Accedi al [pannello di amministrazione ufficiale webman-admin](https://www.workerman.net/plugin/82), apri la pagina dei plugin dell'applicazione e clicca sul pulsante di installazione per installare il plugin corrispondente.

## Installazione dal pacchetto sorgente
Scarica l'archivio del plugin dall'app market, decomprimilo e carica la cartella estratta in `{progetto principale}/plugin/` (crea manualmente la cartella plugin se non esiste). Esegui `php webman app-plugin:install nome_plugin` per completare l'installazione.

Ad esempio, se l'archivio scaricato si chiama ai.zip, decomprimilo in `{progetto principale}/plugin/ai` e esegui `php webman app-plugin:install ai` per completare l'installazione.


# Disinstallazione

Anche la disinstallazione dei plugin dell'applicazione avviene in due modi:

## Disinstallazione dal mercato dei plugin
Accedi al [pannello di amministrazione ufficiale webman-admin](https://www.workerman.net/plugin/82), apri la pagina dei plugin dell'applicazione e clicca sul pulsante di disinstallazione per disinstallare il plugin corrispondente.

## Disinstallazione dal pacchetto sorgente
Esegui `php webman app-plugin:uninstall nome_plugin` per completare la disinstallazione. Successivamente elimina manualmente la cartella del plugin corrispondente in `{progetto principale}/plugin/`.
