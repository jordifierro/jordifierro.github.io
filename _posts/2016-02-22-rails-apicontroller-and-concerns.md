---
layout: post
title:  "Rails ApiController and Concerns"
date:   2016-02-22 17:30:00 +0100
categories: development
comments: true
---

Hi there! Today I want to share with you some code architecture decisions
I've made during the development of
[rails-api-base](https://github.com/jordifierro/rails-api-base) project.
This post is about how I've structured the code
that concerns about controllers common behavior
(I'll only speak about controllers,
 but the concept can be applied to models too).

### ApiController versioning

The DRY (don't repite yourself) basic principle
forces us to collect all spread common code of the controllers
and put it inside the ApplicationController.
Nevertheless, when developing an api we must care about code versioning too.
As is normal, almost all of this common code
is related with the input parameters
or the output format and affects directly to the api version contracts.
Because of that, I've decided that it needs to be versioned,
like the other controllers.
That's the reason to create ApiController,
which has the same purpose as ApplicationController,
but it's placed under `app/controllers/api/v1` folder.
Hence, we'll use one or the other
depending on if the generic code must be versioned or not.

So finally, the inheritance path of an ordinary controller will look like this:

{% highlight ruby %}
                                    < ActionController::API (rails code)
                        < ApplicationController (unversioned code)
            < Api::V1::ApiController (versioned code)
Api::V1::OrdinaryController (versioned code)
{% endhighlight %}

And this is the file tree of
[rails-api-base](https://github.com/jordifierro/rails-api-base) controllers:

{% highlight sh %}
app
└───controllers
    │   application_controller.rb
    └───api
        ├───v1
        │   │   api_controller.rb
        │   │   notes_controller.rb
        │   │   sessions_controller.rb
        │   │   users_controller.rb
        │   │   versions_controller.rb
        │   └───concerns
        │            authenticator.rb
        │            error_handler.rb
        │            internationalizator.rb
        │            version_expiration_handler.rb
        └───v2
                ...
{% endhighlight %}

### ActiveSupport::Concern

But what are those "concerns"?
In order to don't mess the ApiController
with too much and unrelated code,
it can be structured by independent modules using the
[Rails ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html)
module type.
Each file under `/concerns` implements one of these modules,
whom are included in the ApiController, thus, to all api controllers.
Concerns let you define variables, methods...
as usual, but also include behavior to the controller itself
(e.g.: define a `before_action` or `rescue_from`).
That is all we need to refactor the ApiController.

I've applied this pattern to the project
and the result is an ApiController as clean and simple as that:

{% gist jordifierro/3d0d4c9ee4eed1008c26 %}

and here you can see an example of one of the concern modules:

{% gist jordifierro/b97c16d4acb6461fac89 %}

The unique remarkable thing is that
the module extends from `ActiveSupport::Concern`
and places the controller specific code, such as an `before_action`,
under `included`
(the filters, rescues... code that only works from inside a controller class).

On [this post](/rails-api-modules)
you can read more about the four modules included:

* [Authenticator](/rails-api-modules#authenticator)
* [Error handler](/rails-api-modules#error-handler)
* [Internationalizator](/rails-api-modules#internationalizator)
* [Version Expiration Handler](/rails-api-modules#version-expiration-handler)

### Concern modules testing

Finally, last but not least: concern modules testing using
[rspec](http://rspec.info/).

- <b> How do we test a controller module that affects all controllers?</b>
- We must define a fake controller that inherits from ApiController
and check that the behavior included by the target module
works as expected.

Here is an example:

{% gist jordifierro/9d04b9a888553573c734 %}

Here we can see how the fake controller is defined adhoc inside the spec.
In this case, I've also taken the opportunity to test the `current_user`
method, apart from the most important concern purpose defined by the
`before_action :auth_with_token!`.

Few lines below the controller, appears the also fake and adhoc routing.
Usually, the drawn route would be the following one:

{% highlight ruby %}
routes.draw { get 'fake_method_name' => 'anonymous#fake_method_name' }
{% endhighlight %}

but from Rspec version 3 it raises the following error
when the inherited controller is not in the `app/controllers/` folder root:

{% highlight console %}
1) Api::V1::Concerns::Authenticator when not authenticated
     Failure/Error: before { get :fake_current_user, nil }
     ActionController::UrlGenerationError:
       No route matches {:action=>"fake_current_user", :controller=>"api/v1/api"}
     # /Users/jordifierro/.rvm/gems/ruby-2.3.0/gems/actionpack-5.0.0.beta2/lib/action_dispatch/journey/formatter.rb:45:in `generate'
     ...
     # ./spec/controllers/api/v1/concerns/authenticator_spec.rb:23:in `block (3 levels) in <module:V1>'
{% endhighlight %}

To fix that I've changed the route parameters to the controller path
and called it with its name instead of 'anonymous'
(i.e.: if `Api::V1::SomeController` the route is `'api/v1/some#method_name'`):

{% highlight ruby %}
routes.draw { get 'fake_method_name' => 'api/v1/api#fake_method_name' }
{% endhighlight %}

And that's it for now. All feedback is welcome!

Here you can take a look at the whole api code ->
[rails-api-base](https://github.com/jordifierro/rails-api-base)
