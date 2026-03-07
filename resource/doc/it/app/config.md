# File di configurazione

La configurazione dei plugin funziona come in un progetto webman standard, ma generalmente si applica solo al plugin corrente e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.app.controller_suffix` influisce solo sul suffisso del controller del plugin e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.app.controller_reuse` influisce solo sulla riutilizzazione dei controller da parte del plugin e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.middleware` influisce solo sui middleware del plugin e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.view` influisce solo sulle viste utilizzate dal plugin e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.container` influisce solo sul contenitore utilizzato dal plugin e non ha effetti sul progetto principale.
Ad esempio, il valore di `plugin.foo.exception` influisce solo sulla classe di gestione delle eccezioni del plugin e non ha effetti sul progetto principale.

Tuttavia, poiché il routing è globale, le rotte configurate da un plugin influiscono anch'esse sul routing globale.

## Recuperare la configurazione
Per recuperare la configurazione di un plugin, utilizzare `config('plugin.{plugin}.{config_specifica}');`. Ad esempio, per recuperare tutta la configurazione di `plugin/foo/config/app.php`, utilizzare `config('plugin.foo.app')`.
Allo stesso modo, il progetto principale o altri plugin possono utilizzare `config('plugin.foo.xxx')` per recuperare la configurazione del plugin foo.

## Configurazioni non supportate
I plugin applicativi non supportano le configurazioni `server.php` e `session.php`, né le configurazioni `app.request_class`, `app.public_path` e `app.runtime_path`.
