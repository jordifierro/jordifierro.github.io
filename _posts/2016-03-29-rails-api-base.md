---
layout: post
title:  "Rails Api Base"
date:   2016-03-29 17:00:00 +0100
categories: projects
comments: true
---

Hi there! In this post I'm going to summarize the
[rails-api-base](https://github.com/jordifierro/rails-api-base)
project. I've written about this other times:

* [ApiController and Concerns](/rails-apicontroller-and-concerns)
* [Api Modules](/rails-api-modules)
* [Rails api base development](/rails-api-base-development)

so I'll just explain it briefly today.

This project is a simple Rails RESTful api that manages notes
(like pencil annotations) and users with authentication.
The notes are just an example of model
and the main goal of it is to be used as a base to start real projects.

The idea was born of the don't repeat yourself principle,
this time not applied to code components but on a project level.
The boilerplate amount needed to develop an api with users management
made me think about implementing my own reusable piece of code.

Apart from saving precious time on the near future,
the development of this base project is incredibly useful to learn.
The fact of coding things you will certainly reuse,
thinking about them on abstract situations
and making it open and public
forces you to invest more time on each decision,
and that makes you research deeply and learn a lot.

![Bodiam Castle](/assets/images/bodiam_castle.jpg)

Here is a summary of the most important characteristics:

* RESTful api.
* Api versioning.
* Notes app example.
* Patterns and good practices.
* Users management.
* Version expiration.
* Internationalization.
* Secret api key.
* Rspec testing.
* Setup scripts.
* Postgres database.
* Versions up-to-date.
* Ruby Style Guide.
* Json serialization

There is more detailed information about each one of these points at the
[rails-api-base](https://github.com/jordifierro/rails-api-base)
github repository of the project.

You can also check its Android client counterpart project called
[android-base](/android-base),
which shares philosophy with this project
but playing the client role as a mobile application.

If you have any questions about project details
don't hesitate to ask. Contributions are welcome too!
And of course you are free to use the project :)

[(Source code)](https://github.com/jordifierro/rails-api-base)
