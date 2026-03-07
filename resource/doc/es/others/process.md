# Flujo de ejecución

## Flujo de inicio del proceso

Tras ejecutar `php start.php start`, el flujo de ejecución es el siguiente:

1. Cargar las configuraciones bajo config/
2. Configurar el Worker con opciones como `pid_file`, `stdout_file`, `log_file`, `max_package_size`, etc.
3. Crear el proceso webman y escuchar en el puerto (por defecto: 8787)
4. Crear procesos personalizados según la configuración
5. Tras iniciar el proceso webman y los procesos personalizados, se ejecuta la siguiente lógica (todo dentro de `onWorkerStart`):
   ① Cargar los archivos definidos en `config/autoload.php`, como `app/functions.php`
   ② Cargar los middlewares definidos en `config/middleware.php` (incluyendo `config/plugin/*/*/middleware.php`)
   ③ Ejecutar el método `start` de las clases definidas en `config/bootstrap.php` (incluyendo `config/plugin/*/*/bootstrap.php`) para inicializar módulos, como la conexión de base de datos de Laravel
   ④ Cargar las rutas definidas en `config/route.php` (incluyendo `config/plugin/*/*/route.php`)

## Flujo de manejo de solicitudes

1. Comprobar si la URL de la solicitud corresponde a un archivo estático en public. Si es así, devolver el archivo (fin de la solicitud). Si no, pasar al paso 2.
2. Determinar si la URL coincide con alguna ruta. Si no coincide, pasar al paso 3; si coincide, pasar al paso 4.
3. Comprobar si la ruta por defecto está deshabilitada. Si es así, devolver 404 (fin de la solicitud). Si no, pasar al paso 4.
4. Encontrar los middlewares del controlador correspondiente a la solicitud, ejecutar las operaciones previas de los middlewares en orden (fase de solicitud del modelo cebolla), ejecutar la lógica del controlador, ejecutar las operaciones posteriores de los middlewares (fase de respuesta del modelo cebolla) y finalizar la solicitud. (Consulte el [modelo cebolla de middleware](https://www.workerman.net/doc/webman/middleware.html#%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%B4%8B%E8%91%B1%E6%A8%A1%E5%9E%8B))
