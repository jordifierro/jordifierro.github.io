---
layout: post
title:  "Rails api modules"
date:   2016-03-30 11:00:00 +0100
categories: projects
---

This post is about the implementation details of the 4 small modules
of my [rails-api-base](https://github.com/jordifierro/rails-api-base)
project, implemented as Concerns
and included in the ApiController (they impact to the whole api).
You can read more about this on my previous post:
[ApiController and Concerns](/rails-apicontroller-and-concerns).
If you're already familiar with this structure, let's start!

{% gist jordifierro/3d0d4c9ee4eed1008c26 %}

This is the api parent controller,
where we have an index of the concern modules I would like to explain:

### Authenticator

{% gist jordifierro/b97c16d4acb6461fac89 %}

The modules are very simple.

This one is in charge of authenticating the user on each request,
including the `before_action :auth_with_token!`.
`auth_with_token!` checks if the user is signed in, searching it by
the auth_token param (sent by the `Authorization` header).
If the user is found, it remains available
through the `current_user` method, accessible from all controllers.
Otherwise, it returns an unauthorized void response.

(To get a better understanding of the authentication system, devise, model,
controllers... you can take a look at the
[source code](https://github.com/jordifierro/rails-api-base)
of the project)

### Error Handler

{% gist jordifierro/481a061fc911536726c4c7c2a7130520 %}

This one is the responsible to format the output in case of error.

It includes a `rescue_from`
(in this case from record not found, just as an example)
and calls the specific method for each error.
That method simply calls `render_error`,
specifying a message and status.
`render_error` formats the output json with an `error` object,
composed by a `message:string` and `status:integer`.
It also sets the response code.

### Version Expiration Handler

{% gist jordifierro/1a16e9ca39eb2f502417afc648b20e09 %}

Version expiration handler checks the validity of the requested api version
and its expiration date.

With this concern, the expiration date of an api version can be set
defining an environment variable, e.g: `ENV[V1_EXPIRATION_DATE] = "30/03/2016"`
(realize that we can define one for each version: V1, V2...).
The concern includes `check_expiration!`, that renders an error
if that expiration date exists and it's older than the current server date.
`expiration_date` remains available, in my project I use it
to report to the client when the
[versions controller](https://github.com/jordifierro/rails-api-base/blob/master/app/controllers/api/v1/versions_controller.rb)
 is called.

### Internationalizator

{% gist jordifierro/bad03cc0e0f525542bd625d7fc3b79f9 %}

This concern configures the I18n.locale language.

It includes the `before_action :set_locale` and this method
tries to find a language on the `HTTP_ACCEPT_LANGUAGE`
request environment variable
and checks if it is listed as an available one.
You can set the language from the client using the `Accept-Language`
header on your http request.

Comment if you have any questions or suggestions!
