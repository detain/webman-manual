# Come installare webman

* PHP >= 8.1
* [Composer](https://getcomposer.org/) >= 2.0


## Linux: Installare PHP + Composer (saltare se già configurato)
```
curl -sO https://www.workerman.net/install-php-and-composer && sudo bash install-php-and-composer
```
> **Nota**
> Il comando sopra si applica a Linux e macOS. Gli utenti Windows devono installare PHP separatamente.

È possibile anche scaricare manualmente la [versione statica di PHP](https://www.workerman.net/download) fornita da webman e estrarla per l'uso.

## 1. Creare un progetto

```php
composer create-project workerman/webman:~2.0
```

> **Suggerimento**
> In caso di errori, potrebbe essere in uso uno specchio Composer difettoso. Eseguire `composer config -g --unset repos.packagist` per rimuovere il proxy.

## 2. Eseguire

Andare alla directory webman

#### Utenti Windows
Fare doppio clic su `windows.bat` o eseguire `php windows.php` per avviare

> **Suggerimento**
> In caso di errori, alcune funzioni potrebbero essere disabilitate. Consultare il [controllo delle funzioni disabilitate](others/disable-function-check.md) per sbloccarle.

#### Utenti Linux
**Modalità debug** (per sviluppo: l'output appare nel terminale; il servizio si arresta alla chiusura del terminale)

```php
php start.php start
```

**Modalità daemon** (per produzione: nessun output nel terminale; il servizio continua dopo la chiusura del terminale)

```php
php start.php start -d
```

#### Utenti Docker

Avviare tutti i servizi e collegarli alla console
```php
docker-compose up
```

Eseguire i servizi in background
```php
docker-compose up -d
```

> **Suggerimento**
> In caso di errori, alcune funzioni potrebbero essere disabilitate. Consultare il [controllo delle funzioni disabilitate](others/disable-function-check.md) per sbloccarle.

## 3. Accesso

Aprire `http://indirizzo-ip:8787` nel browser.
