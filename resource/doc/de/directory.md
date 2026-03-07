# Verzeichnisstruktur
```
.
├── app                           Anwendungsverzeichnis
│   ├── controller                Controller-Verzeichnis
│   ├── model                     Model-Verzeichnis
│   ├── view                      Ansichtsverzeichnis
│   ├── middleware                Middleware-Verzeichnis
│   │   └── StaticFile.php        Integrierte Static-Datei-Middleware
│   ├── process                   Verzeichnis für benutzerdefinierte Prozesse
│   │   ├── Http.php              Http-Prozess
│   │   └── Monitor.php           Monitor-Prozess
│   └── functions.php             Geschäftsspezifische Funktionen werden in dieser Datei geschrieben
├── config                        Konfigurationsverzeichnis
│   ├── app.php                   Anwendungs-Konfiguration
│   ├── autoload.php              Hier konfigurierte Dateien werden automatisch geladen
│   ├── bootstrap.php             Callback-Konfiguration, die beim Prozessstart (onWorkerStart) ausgeführt wird
│   ├── container.php             Container-Konfiguration
│   ├── dependence.php            Konfiguration der Container-Abhängigkeiten
│   ├── database.php              Datenbank-Konfiguration
│   ├── exception.php             Ausnahme-Konfiguration
│   ├── log.php                   Protokoll-Konfiguration
│   ├── middleware.php            Middleware-Konfiguration
│   ├── process.php               Konfiguration für benutzerdefinierte Prozesse
│   ├── redis.php                 Redis-Konfiguration
│   ├── route.php                 Routenkonfiguration
│   ├── server.php                Konfiguration für Server-Ports, Prozessanzahl usw.
│   ├── view.php                  Ansichtskonfiguration
│   ├── static.php                Konfiguration für statische Dateien und Static-File-Middleware
│   ├── translation.php           Mehrsprachen-Konfiguration
│   └── session.php               Sitzungs-Konfiguration
├── public                        Verzeichnis für statische Ressourcen
├── runtime                       Laufzeitverzeichnis der Anwendung, Schreibrechte erforderlich
├── start.php                     Startdatei des Dienstes
├── vendor                        Verzeichnis für mit Composer installierte Drittanbieter-Bibliotheken
└── support                       Bibliotheksanpassung (einschließlich Drittanbieter-Bibliotheken)
    ├── Request.php               Anfrageklasse
    ├── Response.php              Antwortklasse
    ├── Setup.php                 Installations-Assistenten-Skript
    └── bootstrap.php             Initialisierungsskript nach Prozessstart
```
