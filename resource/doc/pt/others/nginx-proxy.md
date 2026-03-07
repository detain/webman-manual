# Proxy nginx

Quando o webman precisa fornecer acesso direto à rede externa, recomenda-se adicionar um proxy nginx na frente do webman. Isso oferece as seguintes vantagens:

- Os recursos estáticos são tratados pelo nginx, permitindo que o webman se concentre na lógica de negócios
- Múltiplas instâncias do webman podem compartilhar as portas 80 e 443, diferenciando sites pelo nome de domínio, permitindo vários sites em um único servidor
- Permite a coexistência da arquitetura php-fpm e webman
- O proxy nginx com SSL para https é mais simples e eficiente
- Pode filtrar rigorosamente requisições ilegais da rede externa

## Exemplo de proxy nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name dominio_do_site;
  listen 80;
  access_log off;
  # Importante: root deve apontar para o diretório public dentro do webman, não para o diretório raiz do webman
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

  # Negar acesso a todos os arquivos que terminam em .php
  location ~ \.php$ {
      return 404;
  }

  # Permitir acesso ao diretório .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Negar acesso a todos os arquivos ou diretórios que começam com .
  location ~ /\. {
      return 404;
  }

}
```

Geralmente, os desenvolvedores só precisam configurar server_name e root com os valores reais; os outros campos não precisam ser configurados.

> **Nota**
> É especialmente importante que a opção root aponte para o diretório public dentro do webman. Nunca configure para o diretório raiz do webman, caso contrário todos os seus arquivos podem ficar disponíveis para download e acesso pela internet, incluindo arquivos sensíveis como a configuração do banco de dados.
