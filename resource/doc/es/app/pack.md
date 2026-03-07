# Empaquetar

Por ejemplo, empaquetar el complemento de la aplicación foo:

* Establece el número de versión en `plugin/foo/config/app.php` (**importante**)
* Elimina los archivos innecesarios en `plugin/foo`, especialmente los archivos temporales de prueba de la funcionalidad de subida en `plugin/foo/public`
* Si tu proyecto incluye creación de tablas de base de datos u operaciones similares, configura correctamente `plugin/foo/install.sql`. Consulta la [parte de base de datos](database.md)
* Si tu proyecto tiene configuración independiente de base de datos y Redis, elimina primero estas configuraciones. Deben activarse mediante un asistente de instalación al acceder a la aplicación por primera vez (debes implementarlo tú mismo), para que el administrador las complete manualmente y las genere.
* Si tu proyecto incluye menús del panel de administración de webman, configura `plugin/foo/config/menu.php` para que estos menús se establezcan automáticamente al instalar el complemento. Consulta [webman-admin importar menús](https://www.workerman.net/doc/webman-admin/app-development/menu.html)
* Restaura los demás archivos que deban volver a su estado original
* Tras completar los pasos anteriores, entra en el directorio `{proyecto principal}/plugin/`
* Usuarios de Linux: ejecuta `zip -r foo.zip foo` para generar foo.zip
* Usuarios de Windows: haz clic derecho en la carpeta foo y selecciona "Comprimir como archivo ZIP" para generar foo.zip

**foo.zip es el archivo empaquetado. Consulta el siguiente capítulo [Publicar complemento](publish.md)**
