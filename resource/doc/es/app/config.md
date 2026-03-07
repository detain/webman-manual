# Archivo de configuración

La configuración de los plugins funciona igual que en un proyecto webman estándar, pero generalmente solo aplica al plugin actual y no afecta al proyecto principal.
Por ejemplo, el valor de `plugin.foo.app.controller_suffix` solo afecta al sufijo del controlador del plugin y no tiene efecto en el proyecto principal.
Por ejemplo, el valor de `plugin.foo.app.controller_reuse` solo afecta a si el plugin reutiliza controladores y no tiene efecto en el proyecto principal.
Por ejemplo, el valor de `plugin.foo.middleware` solo afecta al middleware del plugin y no tiene efecto en el proyecto principal.
Por ejemplo, el valor de `plugin.foo.view` solo afecta a las vistas utilizadas por el plugin y no tiene efecto en el proyecto principal.
Por ejemplo, el valor de `plugin.foo.container` solo afecta al contenedor utilizado por el plugin y no tiene efecto en el proyecto principal.
Por ejemplo, el valor de `plugin.foo.exception` solo afecta a la clase de manejo de excepciones del plugin y no tiene efecto en el proyecto principal.

Sin embargo, como el enrutamiento es global, las rutas configuradas por un plugin también afectan al enrutamiento global.

## Obtener la configuración
Para obtener la configuración de un plugin, use `config('plugin.{plugin}.{config_específica}');`. Por ejemplo, para obtener toda la configuración de `plugin/foo/config/app.php`, use `config('plugin.foo.app')`.
Del mismo modo, el proyecto principal u otros plugins pueden usar `config('plugin.foo.xxx')` para obtener la configuración del plugin foo.

## Configuración no admitida
Los plugins de aplicación no admiten las configuraciones `server.php` y `session.php`, ni las configuraciones `app.request_class`, `app.public_path` y `app.runtime_path`.
