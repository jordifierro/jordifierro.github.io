---
layout: post
title:  "Rails api base development"
date:   2016-01-27 16:00:00 +0100
categories: projects
comments: true
---

Hi there!
I have in mind a new project (I'll write about it on future posts)
and I've decided to build a lean MVP
(minimum viable product)
to start testing and iterating over it as soon as possible.
That MVP will be composed by an Android native application as client
and a rails api as backend server.
I've started some other projects with that architecture in the past
(and surely I'll do it again in the future)
so I've decided to build first a base template
that could be reused to achieve releasable products faster.
This little project is called Rails api base.

![Stacked Stones](/assets/images/stacked_stones.jpg)

Rails api base is a basic RESTful api template build in Ruby on Rails.
As I've said, it aims to be a minimum reusable example of a basic app
to start a new project but also a place to discuss about rails development.
Decisions about patterns, technologies, gems, best practices, etc.
are made on this "abstract" code, so it can be a good place to talk about them.

In order to let people discuss,
but also freely use and collaborate with the project,
I've uploaded the code as open source to Github.
Here you can find the
[Rails api base repository](https://github.com/jordifierro/rails-api-base).
I hope it can be useful for other rails developers.
I'm far of being a rails expert developer so
I'll also be glad if some of them help me
improving the project with suggestions or collaborations.
Let me know if you are interested on it!

During the development,
I've tried to deeply understand how rails works and its philosophy and
every decision I've taken has a reason behind.
I'm learning a lot and having fun with that process
(I love doing things that I don't know how to do!),
that's important too.

Once the project will be "done"
(actually it will never be totally done, I've in mind to keep improving it)
I'll start with its counterpart for Android,
a client native mobile base app that will consume the data from this api
(I'll keep you updated of that project too).

Finally, let me list some more detailed specifications of the project
(already implemented):

* [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer)
good practices: verbs, endpoints, status codes...
* ...api code custom versioning (by header).
* Example ('notes' objects with 'title' and 'content')
of a basic object model, controller, routes and tests.
* [Devise](https://github.com/plataformatec/devise) to handle users.
* Custom token authentication (by header) and basic session management.
* Usage of
[Concerns](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)
and other rails good practices.
* All code tested using [rspec](https://github.com/rspec/rspec) following
latest [betterspecs.org](http://betterspecs.org/) guidelines.
* [Postgres](http://www.postgresql.org/) as database.
* Rails 5.0.0.beta1 (API module) and Ruby 2.3.0.

(To check the latest specifications, just take a look at the
[repository](https://github.com/jordifierro/rails-api-base))

Thanks for reading!
