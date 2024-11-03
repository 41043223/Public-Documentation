# Codimd 
## This document will tell you how to host your own codimd server

### Install requirement packages

```sh
sudo apt-get update
sudo apt-get docker-compose 
```

### Create compose file
```sh
vim docker-compose.yml
```

```sh   
NOTE: replace [DB_USER] to your user name
      replace [DB_PASSWORD] to your password
      All [] should be remove
```

```sh
version: "3"
services:
  database:
    image: postgres:11.6-alpine
    environment:
      - POSTGRES_USER=[DB_USER]
      - POSTGRES_PASSWORD=[DB_PASSWORD]
      - POSTGRES_DB=codimd
    volumes:
      - "database-data:/var/lib/postgresql/data"
    restart: always
  codimd:
    image: hackmdio/hackmd:2.4.2
    environment:
      - CMD_DB_URL=postgres://[DB_USER]:[DB_PASSWORD]@database/codimd
      - CMD_USECDN=false
    depends_on:
      - database
    ports:
      - "3300:3000"
    volumes:
      - upload-data:/home/hackmd/app/public/uploads
    restart: always
volumes:
  database-data: {}
  upload-data: {}
```

###  Start compose

```sh
sudo docker-compose up -d
```

### Nginx reverse proxy

```sh
This is just a exaple, modify in your server youself
```

```sh
server {
        listen 80 ;
        listen [::]:80;
        server_name YOURDOMAIN;
        return 301 https://$server_name$request_uri;
}


upstream @codimd {
        server 127.0.0.1:3300;
        keepalive 300;
}
# for socket.io (http upgrade)
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;

        ssl_certificate  '/etc/ssl/nginx/nginx1-trustcor.pem'; # Your certificate path
        ssl_certificate_key '/etc/ssl/private/nginx1-trustcor.key'; # Your key path

        server_name YOURDOMAIN;

        location /{
                proxy_http_version 1.1;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;

                # setup for image upload
                client_max_body_size 8192m;

                # adjust proxy buffer setting
                proxy_buffers 8 32k;
                proxy_buffer_size 32k;
                proxy_busy_buffers_size 64k;

                proxy_max_temp_file_size 8192m;

                proxy_read_timeout 300;
                proxy_connect_timeout 300;
                proxy_pass http://@codimd;

        }
}
```
