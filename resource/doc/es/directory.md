# Estructura de directorios
```
.
├── app                           Directorio de la aplicación
│   ├── controller                Directorio de controladores
│   ├── model                     Directorio de modelos
│   ├── view                      Directorio de vistas
│   ├── middleware                Directorio de middlewares
│   │   └── StaticFile.php        Middleware de archivos estáticos incorporado
│   ├── process                   Directorio de procesos personalizados
│   │   ├── Http.php              Proceso Http
│   │   └── Monitor.php           Proceso de supervisión
│   └── functions.php             Las funciones personalizadas del negocio se escriben en este archivo
├── config                        Directorio de configuración
│   ├── app.php                   Configuración de la aplicación
│   ├── autoload.php              Los archivos configurados aquí se cargarán automáticamente
│   ├── bootstrap.php             Configuración del callback ejecutado en onWorkerStart al iniciar el proceso
│   ├── container.php             Configuración del contenedor
│   ├── dependence.php            Configuración de dependencias del contenedor
│   ├── database.php              Configuración de la base de datos
│   ├── exception.php             Configuración de excepciones
│   ├── log.php                   Configuración de registro
│   ├── middleware.php            Configuración de middlewares
│   ├── process.php               Configuración de procesos personalizados
│   ├── redis.php                 Configuración de Redis
│   ├── route.php                 Configuración de rutas
│   ├── server.php                Configuración del servidor (puertos, número de procesos, etc.)
│   ├── view.php                  Configuración de vistas
│   ├── static.php                Configuración de archivos estáticos y middleware
│   ├── translation.php           Configuración multilingüe
│   └── session.php              Configuración de sesión
├── public                        Directorio de recursos estáticos
├── runtime                       Directorio de tiempo de ejecución, requiere permisos de escritura
├── start.php                     Archivo de inicio del servicio
├── vendor                        Directorio de bibliotecas de terceros instaladas por Composer
└── support                       Adaptación de bibliotecas (incluidas las de terceros)
    ├── Request.php               Clase de solicitud
    ├── Response.php              Clase de respuesta
    ├── Setup.php                 Script del asistente de instalación
    └── bootstrap.php             Script de inicialización tras el inicio del proceso
```
