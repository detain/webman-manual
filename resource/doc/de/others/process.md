# Ausführungsablauf

## Prozessstartablauf

Nach dem Ausführen von `php start.php start` erfolgt der folgende Ablauf:

1. Laden der Konfigurationen unter config/
2. Setzen der Worker-bezogenen Konfigurationen wie `pid_file`, `stdout_file`, `log_file`, `max_package_size` usw.
3. Erstellen des webman-Prozesses und Abhören des Ports (Standard: 8787)
4. Erstellen benutzerdefinierter Prozesse basierend auf der Konfiguration
5. Nach dem Start von webman-Prozess und benutzerdefinierten Prozessen wird folgende Logik ausgeführt (alles innerhalb von `onWorkerStart`):
   ① Laden der in `config/autoload.php` definierten Dateien, z.B. `app/functions.php`
   ② Laden der in `config/middleware.php` (inkl. `config/plugin/*/*/middleware.php`) definierten Middlewares
   ③ Ausführen der `start`-Methode der in `config/bootstrap.php` (inkl. `config/plugin/*/*/bootstrap.php`) definierten Klassen zur Modulinitialisierung, z.B. Laravel-Datenbankverbindung
   ④ Laden der in `config/route.php` (inkl. `config/plugin/*/*/route.php`) definierten Routen

## Anfragebehandlungsablauf

1. Prüfen, ob die Anfrage-URL einer statischen Datei unter public entspricht. Wenn ja, Datei zurückgeben (Anfrage beenden). Andernfalls zu Schritt 2.
2. Prüfen, ob die URL eine Route trifft. Wenn nicht, zu Schritt 3; wenn ja, zu Schritt 4.
3. Prüfen, ob die Standardroute deaktiviert ist. Wenn ja, 404 zurückgeben (Anfrage beenden). Wenn nicht, zu Schritt 4.
4. Middlewares des angefragten Controllers ermitteln, Middleware-Voroperationen der Reihe nach ausführen (Onion-Modell Anfragephase), Controller-Geschäftslogik ausführen, Middleware-Nachoperationen ausführen (Onion-Modell Antwortphase) und Anfrage abschließen. (Siehe [Middleware-Onion-Modell](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
