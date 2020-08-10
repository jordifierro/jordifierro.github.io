---
layout: post
title:  "Building my own infrastructure I"
date:   2020-08-10 10:00:00 +0100
categories: development
comments: true
---

![Mur's Castle](/assets/images/infrastructure_castle_1.png)

I used to have my projects spread in different systems:

* [Github pages](https://pages.github.com/)
for my [Jekyll](https://jekyllrb.com/) personal blog.
* [Heroku](https://heroku.com) for my [Django](https://www.djangoproject.com/)
apis and also a [React](https://reactjs.org/) web application.
* External services for databases
([Postgres](https://www.postgresql.org/), [Elasticsearch](https://www.elastic.co/)...).
* Other static hostings...

I decided to start the trip of building my own infrastructure
and migratinf all projects to a centralized system.

What do I gain with this?

* Costs reduction
* Easier customization
* Services are centralized & you gain control
* Lots of learning!

And what do I lose?

* Time and speed.

Is it worth? Maybe...
In my case I was willing to introduce my self in systems world
so I decided to put my hands on work to figure it out!

The first thing I did was improve my linux skills with two
[lpic-1](https://www.lpi.org/our-certifications/lpic-1-overview) courses.
It is always really useful to have a good operating system knowledge,
even if you are not working as devops nor system administrator.

Before starting the migration of any project I though about what I needed and
how to structure my server.
These are the projects I had:

* [jordifierro's](https://jordifierro.com) My personal blog.
It uses Jekyll framework and was hosted on Github pages
(which has Jekyll automatic building integrated).
* [Taddapp](https://taddapp.com) A simple landing page for an Android application.
I don't want to remeber where it was hosted...
* [Pachatary](https://pachatary.com) An Android & iOS application.
I had the api in Heroku and was using heroku plugins (external services)
for databases, mailer, etc. Images were (and are) stored on AWS S3.
* [Llaor](https://llaor.com) A dictionary web. Same as pachatary:
api and web where hosted on Heroku.

_(Heroku is a great service to quickly/easily deploy and scale anything,
but it is a bit expensive...)_

So, I must implement a multiple domain hosting server
that can deliver statics, respond RESTful api requests,
store databases and connect to external services...

To achieve that, projects have to be prepared:
* Dockerize them (for both testing and running).
* Make them configurable. Setup variables must be injectable (eg: `env.list` files).
* Write an strong README documentation.

Once I'd had the schema in my mind I defined the pieces.
I should pick a linux distro ([Ubuntu server](https://releases.ubuntu.com/20.04/)).
Use [Nginx](https://www.nginx.com/) as a server
(to deliver statics and also as reverse proxy for api's).
[Docker](https://www.docker.com/) plays a very important role:
to build statics, run applications and test, store databases, etc.
making dependencies management much more easy.
And [Jenkins](https://www.jenkins.io/) to handle tests, deploys, backups...

And... to complicate it a little bit I added a requirement:
zero-downtime deployments.
So [Haproxy](http://www.haproxy.org/) comes into the equation.
Putting Haproxy before Nginx allows you to have multiple instances of an app
and load balance between them.

I already had the domains (at [namecheap](https://namecheap.pxf.io/4bR69))
so I opened an account on [digital ocean](https://m.do.co/c/8edd7aed3fee)
(it is a cheap and easy ,I didn't want any complicated features).
I created a server instance there choosing Ubuntu 20.04 (LTS) x64 as image.

Here starts the journey!

Follow me to the [next post](https://jordifierro.com/building-my-own-infrastructure-2)!
