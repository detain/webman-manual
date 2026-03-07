# Прокси nginx

Когда webman нужно предоставить прямой доступ из внешней сети, рекомендуется добавить прокси nginx перед webman. Это даёт следующие преимущества:

- Статические ресурсы обрабатываются nginx, позволяя webman сосредоточиться на бизнес-логике
- Несколько экземпляров webman могут совместно использовать порты 80 и 443, различая сайты по доменному имени и размещая несколько сайтов на одном сервере
- Позволяет совмещать архитектуры php-fpm и webman
- Прокси nginx с SSL для https проще и эффективнее
- Может строго фильтровать незаконные запросы из внешней сети

## Пример прокси nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name домен_сайта;
  listen 80;
  access_log off;
  # Важно: root должен указывать на каталог public в webman, а не на корневой каталог webman
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

  # Запретить доступ ко всем файлам с расширением .php
  location ~ \.php$ {
      return 404;
  }

  # Разрешить доступ к каталогу .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Запретить доступ ко всем файлам и каталогам, начинающимся с .
  location ~ /\. {
      return 404;
  }

}
```

Как правило, разработчикам достаточно настроить server_name и root фактическими значениями; остальные поля настраивать не нужно.

> **Внимание**
> Особенно важно: опция root должна указывать на каталог public в webman. Никогда не указывайте корневой каталог webman, иначе все ваши файлы могут стать доступными для скачивания из интернета, включая конфиденциальные файлы, такие как конфигурация базы данных.
