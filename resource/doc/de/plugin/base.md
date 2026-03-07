# Grundlegende Plugins

Grundlegende Plugins sind in der Regel allgemeine Komponenten, die typischerweise per Composer installiert werden, wobei der Code im vendor-Verzeichnis liegt. Bei der Installation können benutzerdefinierte Konfigurationen (Middleware, Prozesse, Routen usw.) automatisch in das Verzeichnis `{Hauptprojekt}config/plugin` kopiert werden. Webman erkennt die Konfiguration in diesem Verzeichnis automatisch und fügt sie in die Hauptkonfiguration ein, sodass Plugins in jede Phase des webman-Lebenszyklus eingreifen können.

Weitere Informationen: [Erstellen von Grundlegenden Plugins](create.md)
