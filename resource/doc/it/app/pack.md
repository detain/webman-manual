# Imballaggio

Ad esempio, per impacchettare il plugin dell'applicazione foo:

* Impostare il numero di versione in `plugin/foo/config/app.php` (**importante**)
* Eliminare i file non necessari in `plugin/foo`, in particolare i file temporanei per il test della funzionalità di caricamento sotto `plugin/foo/public`
* Se il progetto include la creazione di tabelle del database e operazioni simili, configurare correttamente `plugin/foo/install.sql`. Vedere la [sezione database](database.md)
* Se il progetto ha configurazioni indipendenti di database e Redis, rimuovere prima queste configurazioni. Esse devono essere richieste tramite una procedura guidata di installazione al primo accesso all'applicazione (è necessario implementarla autonomamente), consentendo all'amministratore di compilarle manualmente e generarle.
* Se il progetto include menu backend di webman admin, configurare `plugin/foo/config/menu.php` in modo che questi menu siano impostati automaticamente durante l'installazione del plugin. Vedere [webman-admin importare menu](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Ripristinare gli altri file che devono essere riportati allo stato originale
* Dopo aver completato i passaggi sopra, entrare nella cartella `{progetto principale}/plugin/`
* Utenti Linux: eseguire il comando `zip -r foo.zip foo` per generare foo.zip
* Utenti Windows: fare clic con il tasto destro sulla cartella foo e selezionare "Comprimi in file ZIP" per generare foo.zip

**foo.zip è il file impacchettato. Fare riferimento al capitolo successivo [Pubblicare plugin](publish.md)**
