# Struttura della directory
```
.
├── app                           Cartella dell'applicazione
│   ├── controller                Cartella dei controller
│   ├── model                     Cartella dei modelli
│   ├── view                      Cartella delle viste
│   ├── middleware                Cartella dei middleware
│   │   └── StaticFile.php        Middleware per file statici predefinito
│   ├── process                   Cartella dei processi personalizzati
│   │   ├── Http.php              Processo Http
│   │   └── Monitor.php           Processo di monitoraggio
│   └── functions.php             Le funzioni personalizzate dell'applicazione vanno in questo file
├── config                        Cartella di configurazione
│   ├── app.php                   Configurazione dell'applicazione
│   ├── autoload.php              I file configurati qui verranno caricati automaticamente
│   ├── bootstrap.php             Configurazioni dei callback eseguiti all'avvio del processo (onWorkerStart)
│   ├── container.php             Configurazione del contenitore
│   ├── dependence.php            Configurazione delle dipendenze del contenitore
│   ├── database.php              Configurazione del database
│   ├── exception.php             Configurazione delle eccezioni
│   ├── log.php                   Configurazione dei log
│   ├── middleware.php            Configurazione dei middleware
│   ├── process.php               Configurazione dei processi personalizzati
│   ├── redis.php                 Configurazione di Redis
│   ├── route.php                 Configurazione del routing
│   ├── server.php                Configurazione del server (porte, numero di processi, ecc.)
│   ├── view.php                  Configurazione delle viste
│   ├── static.php                Configurazione dei file statici e middleware
│   ├── translation.php           Configurazione multilingue
│   └── session.php               Configurazione della sessione
├── public                        Cartella delle risorse statiche
├── runtime                       Cartella di runtime dell'applicazione, richiede permessi di scrittura
├── start.php                     File di avvio del servizio
├── vendor                        Cartella delle librerie di terze parti installate con Composer
└── support                       Adattatori di librerie (incluse quelle di terze parti)
    ├── Request.php               Classe di richiesta
    ├── Response.php              Classe di risposta
    ├── Setup.php                 Script della procedura guidata di installazione
    └── bootstrap.php             Script di inizializzazione dopo l'avvio del processo
```
