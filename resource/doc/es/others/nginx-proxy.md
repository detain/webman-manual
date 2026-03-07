# Proxy nginx

Cuando webman necesita proporcionar acceso directo desde la red externa, se recomienda añadir un proxy nginx delante de webman. Esto ofrece las siguientes ventajas:

- Los recursos estáticos son manejados por nginx, permitiendo que webman se centre en la lógica de negocio
- Múltiples instancias de webman pueden compartir los puertos 80 y 443, distinguiendo diferentes sitios por nombre de dominio, permitiendo varios sitios en un solo servidor
- Permite la coexistencia entre la arquitectura php-fpm y webman
- El proxy nginx con SSL para https es más sencillo y eficiente
- Puede filtrar estrictamente peticiones ilegales desde la red externa

## Ejemplo de proxy nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name dominio_del_sitio;
  listen 80;
  access_log off;
  # Importante: root debe apuntar al directorio public bajo webman, no al directorio raíz de webman
  root /your/webman/public;

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_pass http://webman;
  }

  # Denegar acceso a todos los archivos que terminen en .php
  location ~ \.php$ {
      return 404;
  }

  # Permitir acceso al directorio .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Denegar acceso a todos los archivos o directorios que comiencen con .
  location ~ /\. {
      return 404;
  }

}
```

Por lo general, los desarrolladores solo necesitan configurar server_name y root con los valores reales; el resto de campos no requieren configuración.

> **Nota**
> Es especialmente importante que la opción root apunte al directorio public bajo webman. Nunca la configure con el directorio raíz de webman, ya que en ese caso todos sus archivos podrían ser descargables y accesibles desde internet, incluidos archivos sensibles como la configuración de la base de datos.
