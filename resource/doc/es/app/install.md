# Instalación

Hay dos formas de instalar complementos de aplicación:

## Instalación desde el mercado de complementos
Acceda al [panel de administración oficial webman-admin](https://www.workerman.net/plugin/82), abra la página de complementos de aplicación y haga clic en el botón de instalación para instalar el complemento correspondiente.

## Instalación desde el paquete de código fuente
Descargue el paquete comprimido del complemento desde el mercado de aplicaciones, descomprímalo y suba el directorio descomprimido a `{proyecto principal}/plugin/` (cree manualmente el directorio plugin si no existe). Ejecute `php webman app-plugin:install nombre_complemento` para completar la instalación.

Por ejemplo, si el archivo descargado se llama ai.zip, descomprímalo en `{proyecto principal}/plugin/ai` y ejecute `php webman app-plugin:install ai` para completar la instalación.


# Desinstalación

También hay dos formas de desinstalar complementos de aplicación:

## Desinstalación desde el mercado de complementos
Acceda al [panel de administración oficial webman-admin](https://www.workerman.net/plugin/82), abra la página de complementos de aplicación y haga clic en el botón de desinstalación para desinstalar el complemento correspondiente.

## Desinstalación desde el paquete de código fuente
Ejecute `php webman app-plugin:uninstall nombre_complemento` para completar la desinstalación. Después, elimine manualmente el directorio del complemento correspondiente en `{proyecto principal}/plugin/`.
