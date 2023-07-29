---
title: "Misskey Install"
date: 2022-05-31T21:47:56+10:00
draft: true
toc: false
images:
tags:
  - tech
---

node.js 安裝

```
curl -sL https://deb.nodesource.com/setup_16.x | bash -
apt-get install -y nodejs
```



postgres 13 安裝

```
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
sudo apt update
sudo apt install postgresql-13 postgresql-client-13
```



postgres 創建數據庫和用戶

```
sudo -u postgres psql
create database misskey with encoding = 'UTF8';
create user misskey with encrypted password 'Hannie09088825';
grant all privileges on database misskey to misskey;
\q
```



安裝 redis

```
apt install redis
```

 

安裝 Yarn

```
npm install -g yarn
```



安全 ffmpeg

```
apt install ffmpeg
```



安裝 python 和 build-essential

```
apt install python build-essential
```



create misskey user

```
adduser --disabled-password --disabled-login misskey
```



安裝 misskey

```
su - misskey
git clone --recursive https://github.com/kococomi/misskey.git
cd misskey
git checkout main
yarn install
```



configure misskey

```
cp .config/example.yml .config/default.yml
nano .config/default.yml
```



build misskey 

```
NODE_ENV=production yarn build
```



Init DB 

```
yarn run init
```

 

Launch 

```
NODE_ENV=production npm start
```



Launch with systemd

```
nano /etc/systemd/system/misskey.service
```

paste and save

```
[Unit]
Description=Misskey daemon

[Service]
Type=simple
User=misskey
ExecStart=/usr/bin/npm start
WorkingDirectory=/home/misskey/misskey
Environment="NODE_ENV=production"
TimeoutSec=60
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=misskey
Restart=always

[Install]
WantedBy=multi-user.target
```

start misskey service

```
systemctl daemon-reload ; systemctl enable misskey
systemctl start misskey
```



安裝 nginx 和 SSL 證書

```
apt install nginx certbot python-certbot-nginx
```



編輯 nginx

```
nano /etc/nginx/sites-available/misskey
```

 

paste the code below and edit it

```
# Sample Nginx configuration for Misskey
#
# 1. Replace example.tld to your domain
# 2. Copy to /etc/nginx/sites-enabled
#    or copy to /etc/nginx/sites-available and symlink from /etc/nginx/sites-ebabled

# For WebSockets
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=cache1:16m max_size=1g inactive=720m use_temp_path=off;

server {
    listen 80;
    listen [::]:80;
    server_name never.bid;

    # For SSL domain validation
    root /var/www/html;
    location /.well-known/acme-challenge/ { allow all; }
    location /.well-known/pki-validation/ { allow all; }
    location / { return 301 https://$server_name$request_uri; }
}

server {
    listen 443 http2;
    listen [::]:443 http2;
    server_name never.bid;
    ssl on;
    ssl_session_timeout 5m;

    # To use letsencrypt certificate
    #ssl_certificate           /etc/letsencrypt/live/example.tld/fullchain.pem;
    #ssl_certificate_key       /etc/letsencrypt/live/example.tld/privkey.pem;

    # To use Debian/Ubuntu's self-signed certificate (For testing or before issuing a certificate)
    ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;

    # SSL protocol settings
    ssl_protocols TLSv1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:AES128-SHA;
    ssl_prefer_server_ciphers on;

    # Change to your upload limit
    client_max_body_size 80m;

    # Proxy to Node
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_http_version 1.1;
        proxy_redirect off;

        # For WebSockets
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        # Cache settings
        proxy_cache cache1;
        proxy_cache_lock on;
        proxy_cache_use_stale updating;
        add_header X-Cache $upstream_cache_status;
    }
}
```

create symlink 

```
ln -s /etc/nginx/sites-available/misskey /etc/nginx/sites-enabled/misskey
```





 test nginx configuration and restart 

```
nginx -t
systemctl restart nginx
```



generate an SSL certificate using certbbot

```certbot --nginx -d example.com
certbot --nginx -d example.com
```

