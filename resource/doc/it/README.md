# Cos'è webman

Webman è un framework di servizi ad alte prestazioni basato su Workerman, che integra HTTP, WebSocket, TCP, UDP e altri moduli. Grazie a tecnologie avanzate come memoria residente, coroutine e pool di connessioni, Webman non solo supera i colli di bottiglia delle prestazioni del PHP tradizionale, ma ne amplia notevolmente gli scenari di applicazione.

Inoltre, Webman offre un potente meccanismo di plugin che consente agli sviluppatori di integrare e riutilizzare rapidamente moduli funzionali sviluppati da altri. Che si tratti di costruire siti web, sviluppare API HTTP, messaggistica istantanea, sistemi IoT, giochi, servizi TCP/UDP o Unix Socket, Webman affronta tutto con facilità, offrendo prestazioni e flessibilità eccezionali.

# Filosofia di webman
**Massima estensibilità e massime prestazioni con un nucleo minimo.**

Webman fornisce solo le funzionalità essenziali (routing, middleware, sessione, interfaccia processi personalizzati). Tutto il resto viene riutilizzato dall'ecosistema Composer. Puoi quindi usare i componenti che conosci già: ad esempio, per i database puoi scegliere [illuminate/database](./db/tutorial.md) di Laravel, [ThinkORM](./db/thinkorm.md) di ThinkPHP o altri come `Medoo`. Integrarli in webman è molto semplice.

# Caratteristiche di webman

1. Alta stabilità. Webman è basato su Workerman, un framework socket molto stabile e con pochissimi bug nell'industria.

2. Prestazioni eccellenti. Webman supera i framework PHP-FPM tradizionali di 10–100 volte ed è circa il doppio più performante di framework Go come Gin ed Echo.

3. Alta riutilizzabilità. L'ecosistema Composer esistente può essere riutilizzato senza modifiche.

4. Grande estensibilità. Supporta processi personalizzati e può fare tutto ciò che Workerman può fare.

5. Molto semplice e facile da usare, con bassi costi di apprendimento e uno stile di codice simile ai framework tradizionali.

6. Supporta l'[impacchettamento binario](./others/bin.md) e può essere eseguito direttamente senza ambiente PHP.

7. Utilizza la licenza MIT open source, molto permissiva e user-friendly per gli sviluppatori.

# Link del progetto
GitHub: https://github.com/walkor/webman **Non esitare a dare una stella!**

Gitee: https://gitee.com/walkor/webman **Non esitare a dare una stella!**

# Dati di benchmark di terze parti

[![](../assets/img/benchmark1.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Con operazioni di query al database, Webman raggiunge fino a 390.000 QPS su una singola macchina, circa 80 volte in più rispetto al framework Laravel con architettura PHP-FPM tradizionale.

[![](../assets/img/benchmarks-go.png)](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf)

Con operazioni di query al database, Webman offre circa il doppio delle prestazioni rispetto a framework web simili in Go.

I dati provengono da [techempower.com](https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf).
