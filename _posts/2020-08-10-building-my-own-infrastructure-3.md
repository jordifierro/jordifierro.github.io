---
layout: post
title:  "Building my own infrastructure III"
date:   2020-08-10 16:00:00 +0100
categories: development
comments: true
---

![Llordà's castle](/assets/images/infrastructure_castle_2.png)

After some security setup I talked about on the
[previous post](https://jordifierro.com/building-my-own-infrastructure-2)...
let's start with the applications!
I will only talk about two of them here (others follow similar pattern):
My blog web and Pachatary mobile application.

## My blog

[jordifierro.com](https://jordifierro.com)

This is a website created with Jekyll so its statics must be builded from source code.

Here I paste the instructions to mount it on the server:

First, add an `A Record` from your domain to the server ip.
Then, generate an ssl certificate with certbot tool:
```bash 
sudo certbot certonly --manual --preferred-challenges=dns --email=jordifierromulero@gmail.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d "*.jordifierro.com" -d jordifierro.com
```

This is the `nginx.conf` for this project:

```bash
server {
    listen 127.0.0.1:80;

    root /var/www/jordifierro.com/html;
    index index.html index.htm index.nginx-debian.html;

    server_name jordifierro.com www.jordifierro.com;

    location / {
            try_files $uri $uri.html $uri/index.html /index.html;
    }
}
```

Setup it:
```bash
sudo cp server-setup/blog/nginx.conf /etc/nginx/sites-available/jordifierro.com
sudo rm /etc/nginx/sites-enabled/jordifierro.com
sudo ln -s /etc/nginx/sites-available/jordifierro.com /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

And finally build and deploy the static files under `/var/www/jordifierro.com`
using a docker jekyll images and volumes:
```bash
git clone https://github.com/jordifierro/jordifierro.github.io.git
cd jordifierro.github.io
sudo mkdir -p .jekyll-cache _site
sudo docker run --rm -t --volume="$PWD:/srv/jekyll" --env JEKYLL_ENV=production jekyll/jekyll:3.8 bash -c "jekyll build --trace"
cd ..
sudo mkdir -p /var/www/jordifierro.com/html
sudo chown -R $USER:$USER /var/www/jordifierro.com/html
sudo mv jordifierro.github.io/_site/* /var/www/jordifierro.com/html/
```

## Pachatary

[pachatary.com](https://pachatary.com)

Here we have to setup an api for pachatary android and ios applications.
It's a Django application with a database, so requires a postgres,
two running instances of the app
and an nginx to serve static content.

Diagram looks like this:
```
| 80             --Docker nginx 01-----Docker api 01-----
|443            |                                        |
>----HAProxy----|                                        ----Docker postgres
|               |                                        |
|                --Docker nginx 02-----Docker api 02-----
```

All docker containers should share a network in order to communicate between them.
Also, nginx and api (01 with 01 and 02 with 02) should share volumes
because api docker is who generates the statics that nginx docker will serve.


Let's start with the database.
Get `pachatary/api/db.env.list` from secrets repo and execute the following commands:
```bash
source pachatary/api/db.env.list
sudo docker volume create pachatary-pgdata
sudo docker network create pachatary-net
sudo docker run --name pachatary-postgres -e POSTGRES_PASSWORD=$PACHATARY_POSTGRES_PASSWORD -v pachatary-pgdata:/var/lib/postgresql/data --restart=always --net pachatary-net -d postgres
sudo docker exec -t pachatary-postgres psql -U postgres -c  "CREATE ROLE $PACHATARY_DB_ROLE WITH LOGIN ENCRYPTED PASSWORD '$PACHATARY_DB_ROLE_PASSWORD'"
sudo docker exec -t pachatary-postgres psql -U postgres -c  "ALTER ROLE $PACHATARY_DB_ROLE createdb"
aws s3 cp s3://pachatary-db/latest.dump latest.dump --profile pachatary
sudo docker exec -t pachatary-postgres psql -U postgres -c "drop database $PACHATARY_DB"
sudo docker exec -t pachatary-postgres psql -U postgres -c "create database $PACHATARY_DB with owner $PACHATARY_DB_ROLE"
sudo docker run --rm -v $PWD:/src -v pachatary-pgdata:/dest -w /src alpine cp latest.dump /dest
sudo docker exec -t pachatary-postgres bash -c "psql -u $PACHATARY_DB_ROLE -d $PACHATARY_DB < /var/lib/postgresql/data/latest.dump"
```

We must create a volume for data persistance and
a network for communicate between nginx, api and database containers.
This commands also restore the latests dump saved on aws s3.

Once we have the database up and running we must setup apis and statics servers.
For this we'll also use docker.
As I said before, this script gets 2 instances of each up
(that will be load balanced with HAProxy).

First of all, generate the ssl certificate:
```bash
sudo certbot certonly --manual --preferred-challenges=dns --email=jordifierromulero@gmail.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d "*.pachatary.com" -d pachatary.com
```

Then, we'll copy the Nginx configurations to Nginx folder.

```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
	worker_connections 768;
}

http {
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;
	gzip on;

	upstream django {
	  server pachatary-api.pachatary-net:8000;
	}

	server {
	  listen 80;
	  server_name api.pachatary.com;

	  location /static/ {
	    autoindex on;
	    root /usr/share/nginx/html;
	  }

	  location / {
	    try_files $uri @proxy_to_app;
	  }

	  location @proxy_to_app {
	    proxy_pass http://django;

	    proxy_http_version 1.1;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection "upgrade";

	    proxy_redirect off;
	    proxy_set_header Host $host;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header X-Forwarded-Host $server_name;
	  }
	}
}
```

I used sed to replace docker host names for the appropiate ones
(`pachatary-api-01` and `pachatary-api-02`):
```bash
sudo mdkir -p /etc/nginx/sites-available/api.pachatary.com
sudo cp server-setup/pachatary/api/nginx.conf /etc/nginx/sites-available/api.pachatary.com/nginx.conf
sudo sed 's/pachatary-api/pachatary-api-01/g' /etc/nginx/sites-available/api.pachatary.com/nginx.conf > /etc/nginx/sites-available/api.pachatary.com/nginx-01.conf
sudo sed 's/pachatary-api/pachatary-api-02/g' /etc/nginx/sites-available/api.pachatary.com/nginx.conf > /etc/nginx/sites-available/api.pachatary.com/nginx-02.conf
```

This nginx config files will be used later.

Finally, deploy:

```bash
# sudo rm -rf pachatary-api
git clone https://github.com/jordifierro/pachatary-api.git
cp server-setup/pachatary/api/env.list pachatary-api/
cd pachatary-api
sudo docker build -t pachatary/api .

sudo docker volume create pachatary-statics-01
sudo docker run -d --restart=always --env-file env.list --net pachatary-net -v pachatary-statics-01:/code/pachatary/staticfiles --name pachatary-api-01 -e INTERNAL_IP=127.0.1.1 -t pachatary/api
sudo docker run --name pachatary-nginx-01 -v pachatary-statics-01:/usr/share/nginx/html/static:ro -v /etc/nginx/sites-available/api.pachatary.com/nginx-01.conf:/etc/nginx/nginx.conf:ro -p 127.0.1.1:80:80 --net pachatary-net --restart=always -d nginx

sudo docker volume create pachatary-statics-02
sudo docker run -d --restart=always --env-file env.list --net pachatary-net -v pachatary-statics-02:/code/pachatary/staticfiles --name pachatary-api-02  -e INTERNAL_IP=127.0.1.2 -t pachatary/api
sudo docker run --name pachatary-nginx-02 -v pachatary-statics-02:/usr/share/nginx/html/static:ro -v /etc/nginx/sites-available/api.pachatary.com/nginx-02.conf:/etc/nginx/nginx.conf:ro -p 127.0.1.2:80:80 --net pachatary-net --restart=always -d nginx
```

We get the code from github and build its docker image.
Then, create volumes to share statics from api container to nginx container.
After that, we can run api and Nginx containers.
Api container will build statics on start up and Nginx will get them through the volume.
Repeat the process with the second container.
Be carefull with the ips, they must match with HAProxy's ones!

Finally, we must configure HAProxy to work as a proxy for the applications.
Copy haproxy.cfg file from this repo to `etc/haproxy/haproxy.cfg`:

```bash
sudo cp server-setup/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg
```

How does it works?

```bash
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

frontend jordifierro
        bind SERVER_IP:80
        bind SERVER_IP:443 ssl crt /etc/haproxy/certs/jordifierro.com.pem crt /etc/haproxy/certs/pachatary.com.pem
        redirect scheme https code 301 if !{ ssl_fc }
        mode http
        option forwardfor header X-Real-IP
        acl is_pachatary_api hdr_dom(host) -i api.pachatary.com
        acl is_pachatary_api hdr_dom(host) -i pachatary.com
        use_backend pachatary_api if is_pachatary_api
        default_backend nginx

backend nginx
        mode http
        server nginx 127.0.0.1:80

backend pachatary_api
        mode http
        option httpchk
        http-check expect ! rstatus ^5
        server pachatary_server_01 127.0.1.1:80 check
        server pachatary_server_02 127.0.1.2:80 check
```

This HAProxy config has 2 paths.
The default one redirects to system Nginx (jordifierro.com),
where each server config will serve appropriate files.
The other is for pachatary.com domain.
It load balances the requests to the two different docker containers
with the intention of achieve zero-downtime deployments.

The last step is to copy the domain certifications to `haproxy/certs` folder
with the desired format:

```bash
sudo mkdir -p /etc/haproxy/certs
sudo cat /etc/letsencrypt/live/pachatary.com/fullchain.pem /etc/letsencrypt/live/pachatary.com/privkey.pem > /etc/haproxy/certs/pachatary.com.pem
sudo cat /etc/letsencrypt/live/jordifierro.com/fullchain.pem /etc/letsencrypt/live/jordifierro.com/privkey.pem > /etc/haproxy/certs/jordifierro.com.pem
```

Let's restart HAProxy to apply changes:
```bash
systemctl restart haproxy
```

To get more information about all the setup you can visit the
[server-setup](https://github.com/jordifierro/server-setup) Github repository.

On the [last post](https://jordifierro.com/building-my-own-infrastructure-4)
I will explain how I use Jenkins to handle server tasks!
