# Plugins básicos

Los plugins básicos suelen ser componentes comunes que se instalan típicamente mediante Composer, con el código ubicado en el directorio vendor. Durante la instalación, las configuraciones personalizadas (middleware, procesos, rutas, etc.) pueden copiarse automáticamente al directorio `{proyecto principal}config/plugin`. Webman reconocerá automáticamente la configuración de este directorio y la fusionará con la configuración principal, permitiendo que los plugins intervengan en cualquier fase del ciclo de vida de webman.

Para más información, consulte [Creación de plugins básicos](create.md).
