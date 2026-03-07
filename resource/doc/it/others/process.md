# Flusso di esecuzione

## Flusso di avvio del processo

Dopo l'esecuzione di `php start.php start`, il flusso di esecuzione è il seguente:

1. Caricare le configurazioni sotto config/
2. Impostare le configurazioni relative al Worker come `pid_file`, `stdout_file`, `log_file`, `max_package_size`, ecc.
3. Creare il processo webman e ascoltare sulla porta (predefinita: 8787)
4. Creare processi personalizzati in base alla configurazione
5. Dopo l'avvio del processo webman e dei processi personalizzati, viene eseguita la seguente logica (tutto all'interno di `onWorkerStart`):
   ① Caricare i file impostati in `config/autoload.php`, come `app/functions.php`
   ② Caricare i middleware impostati in `config/middleware.php` (inclusi `config/plugin/*/*/middleware.php`)
   ③ Eseguire il metodo `start` delle classi impostate in `config/bootstrap.php` (inclusi `config/plugin/*/*/bootstrap.php`) per inizializzare i moduli, come la connessione al database Laravel
   ④ Caricare le rotte definite in `config/route.php` (inclusi `config/plugin/*/*/route.php`)

## Flusso di gestione delle richieste

1. Verificare se l'URL della richiesta corrisponde a un file statico sotto public. Se sì, restituire il file (fine della richiesta). Altrimenti, procedere al passo 2.
2. Determinare se l'URL corrisponde a una rotta. Se non corrisponde, procedere al passo 3; se corrisponde, procedere al passo 4.
3. Verificare se la rotta predefinita è disabilitata. Se sì, restituire 404 (fine della richiesta). Altrimenti, procedere al passo 4.
4. Trovare i middleware del controller corrispondente alla richiesta, eseguire le operazioni preliminari dei middleware in ordine (fase di richiesta del modello a cipolla), eseguire la logica di business del controller, eseguire le operazioni successive dei middleware (fase di risposta del modello a cipolla) e terminare la richiesta. (Vedere il [modello a cipolla dei middleware](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
