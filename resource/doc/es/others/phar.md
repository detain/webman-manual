# Empaquetado phar

Phar es un tipo de archivo de empaquetado en PHP similar a JAR. Puedes usar Phar para empaquetar tu proyecto webman en un único archivo phar para facilitar su implementación.

**Un agradecimiento especial a [fuzqing](https://github.com/fuzqing) por su PR.**

> **Nota**
> Es necesario desactivar la opción de configuración phar en `php.ini`, configurando `phar.readonly = 0`.

## Instalación de la herramienta de línea de comandos
`composer require webman/console`

## Empaquetado
Ejecuta el comando `php webman build:phar` en el directorio raíz del proyecto webman. Esto generará un archivo `webman.phar` en el directorio `build`.

> La configuración relacionada con el empaquetado se encuentra en `config/plugin/webman/console/app.php`.

## Comandos de inicio y detención
**Inicio**
`php webman.phar start` o `php webman.phar start -d`

**Detención**
`php webman.phar stop`

**Ver estado**
`php webman.phar status`

**Ver estado de las conexiones**
`php webman.phar connections`

**Reinicio**
`php webman.phar restart` o `php webman.phar restart -d`

## Notas
* Los proyectos empaquetados no admiten reload; para actualizar el código hace falta reiniciar.

* Para evitar un tamaño de paquete excesivo y uso de memoria, puedes configurar las opciones `exclude_pattern` y `exclude_files` en `config/plugin/webman/console/app.php` para excluir archivos innecesarios.

* Al ejecutar webman.phar, se generará un directorio `runtime` en el mismo directorio que webman.phar, utilizado para almacenar archivos temporales como registros.

* Si tu proyecto utiliza un archivo .env, deberás colocar el archivo .env en el mismo directorio que webman.phar.

* Nunca almacenes archivos subidos por usuarios dentro del paquete Phar, ya que operar con archivos de usuarios mediante el protocolo `phar://` es muy peligroso (vulnerabilidad de deserialización Phar). Los archivos subidos por usuarios deben almacenarse por separado en disco fuera del paquete Phar. Véase más abajo.

* Si tu aplicación necesita subir archivos al directorio público, debes separar el directorio `public` y colocarlo en el mismo directorio que webman.phar. En este caso deberás configurar `config/app.php`.
```
'public_path' => base_path(false) . DIRECTORY_SEPARATOR . 'public',
```
Tu aplicación puede usar la función de ayuda `public_path($ruta_relativa)` para encontrar la ubicación real del directorio público.

* webman.phar no es compatible con procesos personalizados en Windows.
