# Caricamento automatico

## Caricare file PSR-0 tramite Composer
webman segue le specifiche di caricamento automatico `PSR-4`. Se il tuo progetto deve caricare librerie conformi a PSR-0, segui questi passaggi:

- Crea una directory `extend` per memorizzare le librerie PSR-0
- Modifica `composer.json` e aggiungi il seguente contenuto in `autoload`:

```json
"psr-0" : {
    "": "extend/"
}
```
Il risultato finale sarà simile a:
![](../../assets/img/psr0.png)

- Esegui `composer dumpautoload`
- Esegui `php start.php restart` per riavviare webman (nota: è necessario riavviare per applicare le modifiche)

## Caricare alcuni file tramite Composer

- Modifica `composer.json` e aggiungi in `autoload.files` i file da caricare:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Esegui `composer dumpautoload`
- Esegui `php start.php restart` per riavviare webman (nota: è necessario riavviare per applicare le modifiche)

> **Nota**
> I file configurati in `autoload.files` di composer.json vengono caricati prima dell'avvio di webman. I file caricati tramite `config/autoload.php` del framework vengono caricati dopo l'avvio di webman.
> Per i file in `autoload.files` di composer.json le modifiche richiedono restart; reload non basta. I file caricati tramite `config/autoload.php` del framework supportano l’hot-reload; le modifiche si applicano con un reload.

## Caricare alcuni file tramite il framework
Alcuni file potrebbero non essere conformi alle specifiche PSR e non caricarsi automaticamente. Puoi caricarli configurando `config/autoload.php`, ad esempio:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Nota**
 > In `autoload.php` sono configurati `support/Request.php` e `support/Response.php` perché esistono file omonimi in `vendor/workerman/webman-framework/src/support/`. Tramite `autoload.php` si dà priorità alle versioni nella root del progetto, permettendo di personalizzare questi file senza modificare quelli in `vendor`. Se non serve personalizzarli, puoi omettere queste due voci.
