# Automatisches Laden

## PSR-0-konforme Dateien über Composer laden
webman folgt der `PSR-4`-Autoload-Spezifikation. Wenn dein Projekt PSR-0-konforme Bibliotheken laden muss, gehe wie folgt vor:

- Erstelle ein `extend`-Verzeichnis zur Aufnahme von PSR-0-konformen Bibliotheken
- Bearbeite `composer.json` und ergänze unter `autoload` folgende Einträge:

```json
"psr-0" : {
    "": "extend/"
}
```
Das Ergebnis sieht in etwa so aus:
![](../../assets/img/psr0.png)

- Führe `composer dumpautoload` aus
- Führe `php start.php restart` aus, um webman neu zu starten (Hinweis: Nur ein Neustart macht die Änderungen wirksam)

## Bestimmte Dateien über Composer laden

- Bearbeite `composer.json` und füge unter `autoload.files` die zu ladenden Dateien hinzu:
```
"files": [
    "./support/helpers.php",
    "./app/helpers.php"
]
```

- Führe `composer dumpautoload` aus
- Führe `php start.php restart` aus, um webman neu zu starten (Hinweis: Nur ein Neustart macht die Änderungen wirksam)

> **Hinweis**
> In `composer.json` unter `autoload.files` konfigurierte Dateien werden vor dem Start von webman geladen. Über die Framework-Datei `config/autoload.php` geladene Dateien werden erst nach dem Start von webman geladen.
> Änderungen an Dateien in `composer.json` unter `autoload.files` erfordern einen Neustart, Reload reicht nicht. Über `config/autoload.php` geladene Dateien unterstützen Hot-Reload; Änderungen wirken nach einem Reload.

## Bestimmte Dateien über das Framework laden
Manche Dateien entsprechen möglicherweise nicht der PSR-Spezifikation und können nicht automatisch geladen werden. Du kannst sie über `config/autoload.php` laden, z. B.:
```php
return [
    'files' => [
        base_path() . '/app/functions.php',
        base_path() . '/support/Request.php', 
        base_path() . '/support/Response.php',
    ]
];
```
 > **Hinweis**
 > In `autoload.php` sind `support/Request.php` und `support/Response.php` hinterlegt, da dieselben Dateien auch unter `vendor/workerman/webman-framework/src/support/` existieren. Über `autoload.php` werden die Versionen im Projektstammverzeichnis bevorzugt geladen, sodass du diese Dateien anpassen kannst, ohne die in `vendor` zu ändern. Wenn du sie nicht anpassen möchtest, kannst du diese beiden Einträge weglassen.
