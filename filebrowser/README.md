### Filebrowser NOTE

#### Filebrowser with nginx reverse proxy and SSL Certification and Google reCAPTCHA

Ref:<a href="https://filebrowser.org/installation">Site</a>

#### 1. Preparation

```/media/data/filebrowser/``` is root dir

```sh 
cd /media/data/filebrowser/
mkdir database
mkdir config
touch database/filebrowser.db
touch config/settings.json
sudo apt install nginx
```

```sh
vim config/settings.json
```

append it

```
{
  "port": 80,
  "baseURL": "/fileb",
  "address": "",
  "log": "stdout",
  "database": "/database/filebrowser.db",
  "root": "/srv"
}
```
baseURL is use for nginx reverse proxy e.g. ```192.168.0.2/filebrowser``` you need fill ```filebrowser``` to baseURL<br>

port is internal port in docker

#### 2. Docker installation

```sh
sudo apt install docker.io 
```


```sh
sudo docker run -it \
    -v /media/data/filebrowser:/srv \
    -v /media/data/filebrowser/database/filebrowser.db:/database/filebrowser.db \
    -v /media/data/filebrowser/config/settings.json:/config/settings.json \
    -e PUID=$(id -u) \
    -e PGID=$(id -g) \
    -p 8880:80 \
    filebrowser/filebrowser:s6
```

8880 is host port, 80 is docker internal port


##### automatically start when boot
```
sudo docker update --restart unless-stopped container_ID
```

```sudo docker ps -a```  can find your container_ID



#### 3. nginx configuration

```sh
sudo vim /etc/nginx/sites-enabled/default
```

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name chuchu.ddns.net;
        return 301 https://$host$request_uri;
}
server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        ssl_certificate  '/etc/ssl/nginx/nginx1-trustcor.pem';
        ssl_certificate_key '/etc/ssl/private/nginx1-trustcor.key';

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name chuchu.ddns.net;

        location /fileb/ {
                proxy_pass http://localhost:8880;

                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        }
}


```

ssl_certificate seems like ```-----BEGIN RSA PRIVATE KEY-----``` ```-----END RSA PRIVATE KEY-----```   (download from your SSL CA)
 
ssl_certificate_key seems like ```-----BEGIN PRIVATE KEY-----```  ```-----END PRIVATE KEY-----```

server_name is your domain name

```
        location /fileb/ {     //fileb is your ngnix reverse proxy keyword
                proxy_pass http://localhost:8880;   //proxy to which host

                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $http_x_forwarded_proto;
        }
```

restart it
```
sudo service nginx restart
```

#### 4. Google reCAPTCHA

go to <a href="www.google.com/recaptcha/admin"> Google reCAPTCHA </a>

click create and select reCAPTCHA v2 invisible, enter domain then you can get site key and private key

```
filebrowser config set --auth.method=json \
    --recaptcha.key site-key \
    --recaptcha.secret private-key
```

#### NOTE1 
reCAPTCHA v3 does not support

#### NOTE2
config only can edit while service is down, you can copy present ```filebrowser.db``` to other dir, edit it, and copy back (in my instance, filebrowser.db save in docker /database, I create a dir named old, and copy filebrowser.db to old, and edit in old, then copy back to ..) 



