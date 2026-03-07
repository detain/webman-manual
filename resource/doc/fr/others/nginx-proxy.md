# Proxy Nginx

Lorsque webman doit être accessible directement depuis Internet, il est recommandé d'ajouter un proxy Nginx devant webman. Cela présente les avantages suivants :

- Les ressources statiques sont gérées par Nginx, permettant à webman de se concentrer sur la logique métier
- Plusieurs instances webman peuvent partager les ports 80 et 443, en distinguant les sites par nom de domaine, permettant plusieurs sites sur un seul serveur
- Permet la coexistence de l'architecture php-fpm et webman
- Le proxy Nginx avec SSL pour https est plus simple et efficace
- Peut filtrer strictement les requêtes illégales provenant d'Internet

## Exemple de proxy Nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name domaine_du_site;
  listen 80;
  access_log off;
  # Important : root doit pointer vers le répertoire public sous webman, pas vers la racine de webman
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

  # Refuser l'accès à tous les fichiers se terminant par .php
  location ~ \.php$ {
      return 404;
  }

  # Autoriser l'accès au répertoire .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Refuser l'accès à tous les fichiers ou répertoires commençant par .
  location ~ /\. {
      return 404;
  }

}
```

En général, les développeurs n'ont besoin de configurer que server_name et root avec les valeurs réelles ; les autres champs ne nécessitent pas de configuration.

> **Note**
> Il est particulièrement important que l'option root pointe vers le répertoire public sous webman. Ne la définissez jamais sur la racine de webman, sinon tous vos fichiers pourraient être téléchargeables et accessibles depuis Internet, y compris les fichiers sensibles tels que la configuration de la base de données.
