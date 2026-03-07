# Was ist webman

Webman ist ein leistungsstarkes Service-Framework, das auf Workerman basiert und HTTP, WebSocket, TCP, UDP sowie andere Module integriert. Durch Technologien wie residenten Speicher, Coroutines und Connection Pools durchbricht Webman nicht nur die Leistungsgrenzen traditioneller PHP-Anwendungen, sondern erweitert deren Anwendungsmöglichkeiten erheblich.

Außerdem bietet Webman einen leistungsstarken Plugin-Mechanismus, mit dem Entwickler Funktionsmodule anderer Entwickler schnell integrieren und wiederverwenden können. Ob Websites, HTTP-APIs, Instant Messaging, IoT-Systeme, Spiele, TCP/UDP-Dienste, Unix-Socket-Dienste oder ähnliches—Webman bewältigt all dies mühelos mit herausragender Leistung und Flexibilität.

# Die Philosophie von webman
**Maximale Erweiterbarkeit und beste Leistung mit einem minimalen Kern.**

Webman bietet nur die wesentlichsten Funktionen (Routing, Middleware, Session, Schnittstelle für eigene Prozesse). Alle weiteren Funktionen stammen aus dem Composer-Ökosystem. Das heißt, Sie können in webman mit Ihren gewohnten Komponenten arbeiten—bei Datenbanken z.B. Laravels [illuminate/database](./db/tutorial.md), ThinkPHP’s [ThinkORM](./db/thinkorm.md) oder andere wie `Medoo`. Die Integration in webman ist sehr einfach.

# Merkmale von webman

1. Hohe Stabilität. Webman baut auf Workerman auf, einem seit langem bewährten Socket-Framework mit sehr wenigen Fehlern.

2. Sehr hohe Leistung. Webman ist 10–100-mal schneller als klassische PHP-FPM-Frameworks und etwa doppelt so schnell wie Go-Frameworks wie Gin oder Echo.

3. Hohe Wiederverwendbarkeit. Das bestehende Composer-Ökosystem kann nahtlos weiterverwendet werden.

4. Hohe Erweiterbarkeit. Eigene Prozesse werden unterstützt, sodass alles möglich ist, was auch mit Workerman möglich ist.

5. Sehr einfach und intuitiv, geringer Lernaufwand, der Code unterscheidet sich kaum von klassischen Frameworks.

6. Unterstützt [Binär-Packaging](./others/bin.md) und kann ohne PHP-Umgebung direkt ausgeführt werden.

7. Nutzt die sehr permissive und entwicklerfreundliche MIT Open-Source-Lizenz.

# Projektadressen
GitHub: https://github.com/walkor/webman **Gerne einen Stern vergeben!**

Gitee: https://gitee.com/walkor/webman **Gerne einen Stern vergeben!**

# Benchmarks von unabhängiger Seite

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Bei Datenbankabfragen erreicht Webman auf einer einzelnen Maschine bis zu 390.000 QPS—etwa 80-mal mehr als Laravel mit klassischer PHP-FPM-Architektur.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Bei Datenbankabfragen übertrifft Webman vergleichbare Go-Web-Frameworks etwa um den Faktor zwei.

Die Angaben stammen von [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
