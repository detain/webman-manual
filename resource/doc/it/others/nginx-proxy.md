# Proxy Nginx

Quando webman deve essere accessibile direttamente dalla rete esterna, si consiglia di aggiungere un proxy nginx davanti a webman. Questo offre i seguenti vantaggi:

- Le risorse statiche sono gestite da nginx, consentendo a webman di concentrarsi sulla logica di business
- Più istanze webman possono condividere le porte 80 e 443, distinguendo i siti per nome di dominio, permettendo più siti su un singolo server
- Permette la coexistenza dell'architettura php-fpm e webman
- Il proxy nginx con SSL per https è più semplice ed efficiente
- Può filtrare rigorosamente le richieste illegali dalla rete esterna

## Esempio di proxy nginx
```
upstream webman {
    server 127.0.0.1:8787;
    keepalive 10240;
}

server {
  server_name dominio_del_sito;
  listen 80;
  access_log off;
  # Importante: root deve puntare alla directory public sotto webman, non alla directory radice di webman
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

  # Nega l'accesso a tutti i file che terminano con .php
  location ~ \.php$ {
      return 404;
  }

  # Consenti l'accesso alla directory .well-known
  location ~ ^/\.well-known/ {
    allow all;
  }

  # Nega l'accesso a tutti i file o directory che iniziano con .
  location ~ /\. {
      return 404;
  }

}
```

In genere, gli sviluppatori devono solo configurare server_name e root con i valori effettivi; gli altri campi non richiedono configurazione.

> **Nota**
> È particolarmente importante che l'opzione root punti alla directory public sotto webman. Non impostarla mai sulla directory radice di webman, altrimenti tutti i file potrebbero essere scaricabili e accessibili da Internet, inclusi file sensibili come la configurazione del database.
