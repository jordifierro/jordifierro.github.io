---
layout: post
title:  "Building my own infrastructure II"
date:   2020-08-10 12:00:00 +0100
categories: development
comments: true
---

![Door lock](/assets/images/infrastructure_door.png)

At the [first post](https://jordifierro.com/building-my-own-infrastructure-1)
of this series I have explained the background and planned the system.

Once I had a running instance of ubuntu server,
first thing I did was secure it a little bit.
What I wanted was to avoid intrusions and decrease risks in case of them.
To do that, I created a sudo user, closed root and password ssh logins,
changed ssh port to prevent sniffer scripts to find it easily
and setup a firewall.

Here are the steps:

Login as root on your server.

Create your user, add a password to it and make it sudoer:
```bash
useradd -m -s /bin/bash myuser
passwd myuser
usermod -aG sudo myuser
exit
```

Generate your ssh key (if not already done), add it to your server user trusted keys
and ssh into server:
```bash
ssh-keygen
ssh-copy-id myuser@serverip
ssh myuser@serverip
```

Configure and activate firewall:
```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 322/tcp          # for ssh later use
sudo ufw --force enable
```

Edit ssh config to make it more secure
(close password and root login and change ssh port):
```bash
sudo vim /etc/ssh/sshd_config

----------------------------------
Port 322
PasswordAuthentication no
ChallengeResponseAuthentication no
PermitRootLogin no
----------------------------------

sudo systemctl restart ssh
exit
```

Now you can log in again and delete ssh ufw rule:
```bash
ssh -p 322 myuser@serverip
sudo ufw delete allow ssh
```

Once that was done, I installed all the needed software:
* Docker
* HAProxy
* Nginx
* Jenkins
* AWS cli _(to store db backups)_
* Letsencrypt certbot _(to generate ssl certificates)_

Commands to install all these packages are explained in detail
on [project README](https://github.com/jordifierro/server-setup)
but almost all of them were installed using `apt install`.

Keep it reading how I
[built my own infrastructure](https://jordifierro.com/building-my-own-infrastructure-3)!
