# Nginx-Proxy

Wenn webman direkten Zugriff von außen benötigt, wird empfohlen, einen Nginx-Proxy vor webman zu setzen. Dies bietet folgende Vorteile:

- Statische Ressourcen werden von Nginx verarbeitet, sodass webman sich auf die Geschäftslogik konzentrieren kann
- Mehrere webman-Instanzen können die Ports 80 und 443 gemeinsam nutzen und über verschiedene Domains unterschieden werden, um mehrere Websites auf einem Server zu betreiben
- Ermöglicht das Zusammenarbeiten von PHP-FPM- und webman-Architektur
- Nginx-Proxy mit SSL für HTTPS ist einfacher und effizienter
- Kann illegale Anfragen aus dem externen Netz strikt filtern

## Nginx-Proxy-Beispiel
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name domain_der_website;
  listen 80;
  access_log off;
  # Wichtig: root muss auf das public-Verzeichnis unter webman zeigen, nicht auf das webman-Stammverzeichnis
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

  # Zugriff auf alle Dateien mit Endung .php verweigern
  location ~ \.php$ {
      return 404;
  }

  # Zugriff auf .well-known-Verzeichnis erlauben
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Zugriff auf alle Dateien oder Verzeichnisse, die mit . beginnen, verweigern
  location ~ /\. {
      return 404;
  }

}
```

In der Regel müssen Entwickler nur server_name und root mit den tatsächlichen Werten konfigurieren; andere Felder müssen nicht konfiguriert werden.

> **Hinweis**
> Besonders wichtig: Die Option root muss auf das public-Verzeichnis unter webman zeigen. Setzen Sie sie niemals auf das webman-Stammverzeichnis, sonst könnten alle Ihre Dateien aus dem Internet herunterladbar und zugreifbar sein, einschließlich sensibler Dateien wie der Datenbankkonfiguration.
