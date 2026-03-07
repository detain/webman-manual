# Installation

Es gibt zwei Möglichkeiten, Anwendungs-Plugins zu installieren:

## Installation über den Plugin-Markt
Öffnen Sie die [offizielle Verwaltungsoberfläche webman-admin](https://www.workerman.net/plugin/82), gehen Sie zur Anwendungs-Plugin-Seite und klicken Sie auf die Installationsschaltfläche, um das entsprechende Anwendungs-Plugin zu installieren.

## Installation über das Quellpaket
Laden Sie das Anwendungs-Plugin-Archiv vom Anwendungsmarkt herunter, entpacken Sie es und laden Sie das entpackte Verzeichnis in `{Hauptprojekt}/plugin/` hoch (erstellen Sie das plugin-Verzeichnis bei Bedarf manuell). Führen Sie `php webman app-plugin:install Pluginname` aus, um die Installation abzuschließen.

Beispiel: Wenn das heruntergeladene Archiv ai.zip heißt, entpacken Sie es nach `{Hauptprojekt}/plugin/ai` und führen Sie `php webman app-plugin:install ai` aus, um die Installation abzuschließen.


# Deinstallation

Es gibt auch zwei Möglichkeiten, Anwendungs-Plugins zu deinstallieren:

## Deinstallation über den Plugin-Markt
Öffnen Sie die [offizielle Verwaltungsoberfläche webman-admin](https://www.workerman.net/plugin/82), gehen Sie zur Anwendungs-Plugin-Seite und klicken Sie auf die Deinstallationsschaltfläche, um das entsprechende Anwendungs-Plugin zu deinstallieren.

## Deinstallation über das Quellpaket
Führen Sie `php webman app-plugin:uninstall Pluginname` aus, um die Deinstallation abzuschließen. Löschen Sie anschließend manuell das entsprechende Plugin-Verzeichnis in `{Hauptprojekt}/plugin/`.
