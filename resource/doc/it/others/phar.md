# Pacchetto phar

Phar è un tipo di file di impacchettamento simile a JAR in PHP. Puoi usare Phar per impacchettare il progetto webman in un singolo file phar per facilitarne la distribuzione.

**Un ringraziamento speciale a [fuzqing](https://github.com/fuzqing) per il suo PR.**

> **Nota**
> È necessario disabilitare le opzioni di configurazione phar nel file `php.ini`, impostando `phar.readonly = 0`.

## Installazione del tool da riga di comando
`composer require webman/console`

## Impacchettamento
Esegui il comando `php webman build:phar` nella cartella radice del progetto webman. Questo genererà un file `webman.phar` nella cartella `build`.

> Le configurazioni relative all'impacchettamento sono presenti in `config/plugin/webman/console/app.php`.

## Comandi per avviare e interrompere
**Avviare**
`php webman.phar start` o `php webman.phar start -d`

**Interrompere**
`php webman.phar stop`

**Verificare lo stato**
`php webman.phar status`

**Verificare lo stato della connessione**
`php webman.phar connections`

**Riavviare**
`php webman.phar restart` o `php webman.phar restart -d`

## Note
* I progetti impacchettati non supportano il reload; per aggiornare il codice è necessario riavviare.

* Per evitare un pacchetto troppo grande e un eccessivo utilizzo di memoria, puoi impostare le opzioni `exclude_pattern` e `exclude_files` in `config/plugin/webman/console/app.php` per escludere file non necessari.

* Dopo aver eseguito webman.phar, verrà generata una cartella `runtime` nella stessa directory, utilizzata per salvare i file di log e altri file temporanei.

* Se nel progetto viene utilizzato un file .env, è necessario posizionarlo nella stessa directory di webman.phar.

* Non memorizzare mai i file caricati dagli utenti nel pacchetto Phar, poiché operare su questi file tramite il protocollo `phar://` è molto pericoloso (vulnerabilità di deserializzazione Phar). I file caricati dagli utenti devono essere memorizzati separatamente su disco, fuori dal pacchetto Phar. Vedi sotto.

* Se il tuo progetto richiede di caricare file nella cartella public, è necessario isolare la cartella public e posizionarla nella stessa directory di webman.phar. In questo caso, è necessario configurare `config/app.php`.
```php
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
Puoi utilizzare la funzione di assistenza `public_path($percorso_relativo)` per trovare la posizione effettiva della cartella public.

* webman.phar non supporta l'avvio di processi personalizzati in Windows.
