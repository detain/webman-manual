# Plugin di base

I plugin di base sono generalmente componenti comuni che vengono tipicamente installati tramite Composer, con il codice collocato nella directory vendor. Durante l'installazione, le configurazioni personalizzate (middleware, processi, route, ecc.) possono essere copiate automaticamente nella directory `{progetto principale}config/plugin`. Webman riconoscerà automaticamente la configurazione di questa directory e la unirà alla configurazione principale, consentendo ai plugin di intervenire in qualsiasi fase del ciclo di vita di webman.

Per ulteriori informazioni, consultare [Creazione di plugin di base](create.md).
