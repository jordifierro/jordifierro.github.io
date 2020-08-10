---
layout: post
title:  "Building my own infrastructure IV"
date:   2020-08-10 18:00:00 +0100
categories: development
comments: true
---

![Mur's castle inside](/assets/images/infrastructure_castle_inside.png)

In this last post
(see the [previous one](https://jordifierro.com/building-my-own-infrastructure-3))
I'm going to talk about [Jenkins](https://www.jenkins.io),
how to configure it and what I use it for.

# Setup

To install Jenkins on Ubuntu:

```bash
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install openjdk-11-jdk-headless
sudo apt install jenkins
sudo systemctl enable --now jenkins
```

I have this file `server-setup/jenkins/nginx.conf` already created:
```bash
upstream jenkins {
  server 127.0.0.1:8080 fail_timeout=0;
}

server {
  listen 127.0.0.1:80;
  server_name jenkins.jordifierro.com;

  ssl_certificate /etc/letsencrypt/live/jordifierro.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/jordifierro.com/privkey.pem;

  location / {
    proxy_set_header        Host $host:$server_port;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;
    proxy_redirect http:// https://;
    proxy_pass              http://jenkins;
    # Required for new HTTP-based CLI
    proxy_http_version 1.1;
    proxy_request_buffering off;
    proxy_buffering off; # Required for HTTP-based CLI to work over SSL
    # workaround for https://issues.jenkins-ci.org/browse/JENKINS-45651
    add_header 'X-SSH-Endpoint' 'jenkins.jordifierro.com:50022' always;
  }
}
```

Copy that file to nginx conf and restart nginx:
```bash
sudo cp server-setup/jenkins/nginx.conf /etc/nginx/sites-available/jenkins.jordifierro.com
sudo rm /etc/nginx/sites-enabled/jenkins.jordifierro.com
sudo ln -s /etc/nginx/sites-available/jenkins.jordifierro.com /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

I have exposed Jenkins on `jenkins.jordifierro.com`.
Now, I can enter and configure my user.

Once installed, open it and create your user.

After that, install Github hook and credentials plugins. Then:

* Add ssh credentials to github:

```bash
sudo su - jenkins
ssh-keygen

# copy /var/lib/jenkins/.ssh/id_rsa.pub to github ssh new credential
```

* Add pachatary env.list file as secret file named `pachatary_env_list`
and db.env.list file as secret file named `pachatary_db_env_list`
(files with env and secrets vars that will be used by the jobs).

# Jobs

Here I'm going to explain the jobs I've created for my blog and Pachatary.

## Blog

Jobs:

* Deploy

### Deploy

This job is almost the same as the script to deploy my blog
(see the [previous post](https://jordifierro.com/building-my-own-infrastructure-3)).
The difference here is that I configured it
to run on git push on master branch of the Github repository of the project.
To make that enter to `Configure` page of the job and fill the Github forms.

## Pachatary

Jobs:

* Deploy
* Test
* Test and deploy
* Restore db
* Backup db
* Reindex all experiences

### Deploy

This job is not hooked with github because we will trigger it
from test_and_deploy pipeline.
The only configuration that has to be added here is the `env.list` secret file.
I added it under the name `pachatary_env_list`.
Here is the job script:

```bash
cat $pachatary_env_list > env.list

sudo docker build -t pachatary/api .

sudo docker update --restart=no pachatary-nginx-01
sudo docker stop pachatary-nginx-01
sudo docker rm pachatary-nginx-01
sudo docker update --restart=no pachatary-api-01
sudo docker stop pachatary-api-01
sudo docker rm pachatary-api-01
sudo docker run -d --restart=always --env-file env.list --net pachatary-net -v pachatary-statics-01:/code/pachatary/staticfiles --name pachatary-api-01 -e INTERNAL_IP=127.0.1.1 -t pachatary/api
sudo docker run --name pachatary-nginx-01 -v pachatary-statics-01:/usr/share/nginx/html/static:ro -v /etc/nginx/sites-available/api.pachatary.com/nginx-01.conf:/etc/nginx/nginx.conf:ro -p 127.0.1.1:80:80 --net pachatary-net --restart=always -d nginx

response=000
while [ $response -gt 499 -o "${response}" = 000 ]
do
    sleep 1
    response=$(curl --write-out %{http_code} --silent --output /dev/null 127.0.1.1)
    echo $response
done

sudo docker update --restart=no pachatary-nginx-02
sudo docker stop pachatary-nginx-02
sudo docker rm pachatary-nginx-02
sudo docker update --restart=no pachatary-api-02
sudo docker stop pachatary-api-02
sudo docker rm pachatary-api-02
sudo docker run -d --restart=always --env-file env.list --net pachatary-net -v pachatary-statics-02:/code/pachatary/staticfiles --name pachatary-api-02 -e INTERNAL_IP=127.0.1.2 -t pachatary/api
sudo docker run --name pachatary-nginx-02 -v pachatary-statics-02:/usr/share/nginx/html/static:ro -v /etc/nginx/sites-available/api.pachatary.com/nginx-02.conf:/etc/nginx/nginx.conf:ro -p 127.0.1.2:80:80 --net pachatary-net --restart=always -d nginx
```

It is similar to the one used on the
[previous post](https://jordifierro.com/bulding-my-own-infrastructure-3),
but in this case we `cat` the `env.list` file from a secret file.
I've also added an sleep between the update of the container
01 and 02 to avoid downtime (HAProxy does the rest).

### Test

I run the tests using docker-compose.
I've also used Django testing tags to exclude elasticsearch ones
(elasticsearch docker container weights 2GB and generates out of memory exceptions).

```bash
sudo docker-compose down
sudo docker-compose build
sudo docker-compose run api bash -c "pytest && python manage.py test --exclude-tag=elasticsearch"
sudo docker-compose down
```

### Test and deploy (pipeline)

This pipeline launches test and deploy jobs:

```bash
stage('test') {
    build 'test';
}
stage('deploy') {
    build 'deploy'
}
```

The intention is to run it when a git push occurs on master branch,
but I couldn't get Github hooks working with a Jenkins pipeline
so I added another simple job that triggers this pipeline to achieve that
(ugly but it does the trick):

### Test and deploy trigger

Hook it to github and make it trigger test_and_deploy (previous one) job.

### Backup db

Here I also needed `db.env.list` secret file to hide some variables.
I added it under the name `pachatary_db_env_list`.
What this job does is to make a `pg_dump` and upload it to aws s3
(previously installed and configured profiles)
to files `latest.dump`(which always keeps the last version) and `2020-08-10.dump` (for example):

```bash
eval "$(cat $pachatary_db_env_list)" 
date=$(date +%F)
sudo docker exec pachatary-postgres pg_dump -U postgres --verbose $PACHATARY_DB > $date.dump
sudo aws s3 cp $date.dump s3://pachatary-db/$date.dump --profile pachatary
sudo aws s3 cp s3://pachatary-db/$date.dump s3://pachatary-db/latest.dump --profile pachatary
```

### Restore db

This is the inverse job of the previous one: it takes the `latest.dump` from aws s3
and recreates the postgres database with it:

```bash
eval "$(cat $pachatary_db_env_list)" 
sudo aws s3 cp s3://pachatary-db/latest.dump latest.dump --profile pachatary
sudo docker run --rm -v $PWD:/src -v pachatary-pgdata:/dest -w /src alpine cp latest.dump /dest
sudo docker exec -t pachatary-postgres psql -U postgres -c "SELECT pid, pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = '$PACHATARY_DB' AND pid <> pg_backend_pid();"
sudo docker exec -t pachatary-postgres psql -U postgres -c "drop database $PACHATARY_DB"
sudo docker exec -t pachatary-postgres psql -U postgres -c "create database $PACHATARY_DB with owner $PACHATARY_DB_ROLE"
sudo docker exec -t pachatary-postgres bash -c "psql -U $PACHATARY_DB_ROLE -d $PACHATARY_DB < /var/lib/postgresql/data/latest.dump"
```

It uses docker to copy the file to a volume shared with postgres docker container.
Then, it drops and creates database again from that file using docker too.

### Reindex all experiences

And the last job is an example of how to launch a routinary Django command,
in this case the one that reindex all experiences entities from database
to elasticsearch:

```bash
sudo docker exec -t pachatary-api python manage.py reindex_all_experiences
```


_Extra trick:_

I've versioned and backuped Jenkins config files
so I can better work with them and restore if needed.
Take a look at 
[server-jenkins](https://github.com/jordifierro/server-jenkins) repository.
Be careful to not version control any sensible file using
[`.gitignore`](https://github.com/jordifierro/server-jenkins/blob/master/.gitignore)!


I hope you enjoyed this journey, where I've explained how I migrated
4 of my projects to a server.
I'm a noob on the systems world so I know there are a lot of things that can be improved.
Feel free to comment anything. Thanks for reading me!
