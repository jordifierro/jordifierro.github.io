---
layout: post
title:  "Android Base"
date:   2016-03-31 09:00:00 +0100
categories: projects
---

I'm glad to present you my new project:
[android-base](https://github.com/jordifierro/android-base)

It's almost finished (there are still some tasks to be done)
and I haven't posted anything about it yet.
This post is just a brief summary,
I'll link here the specific posts once I publish them.

Android Base is a mobile Android application template
where you can manage notes (like pencil annotations)
and users with authentication.
The content is just to have something to code
and the goal isn't make a useful app
but to create a structure for future projects,
and learn during the process too.

This application is a RESTful mobile client
and consumes its data from its partner project,
[rails-api-base](https://github.com/jordifierro/rails-api-base).
While using different technologies, platforms and RESTful roles
(one serves and the other consumes),
the projects share origins, goals and philosophy.
That aspects are already described [here](/rails-api-base)
so I'll ignore them on this post and focus on the particularities.

![Neuschwanstein Castle](/assets/images/neuschwanstein_castle.jpg)

Unlike with the Rails app,
I was experienced with Android when I started this project.
Still, I decided to take this opportunity
to push my Android skills to the next level.
So I've decided do it as well as possible.

I did a previous research about the new available methodologies and libraries
and I decided to apply the
[Clean Architecture](https://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)
with the
[model-view-presenter](http://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/)
 pattern.
I also learnt how to implement
[Dependency Injection](https://guides.codepath.com/android/Dependency-Injection-with-Dagger-2)
on Android
and apply the
[Reactive Programming](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
paradigm.
Last but not least, I've reached my main goal: make unit testing painless
and reach full testing coverage on an Android project.

For more information go to the
[github repository](https://github.com/jordifierro/android-base)
of the project.
Here I leave you a list the most outstanding points
you'll find there explained in detail:

* Notes app example.
* MVP Clean Architecture.
* Dependency Inversion (Dagger 2 and Butterknife).
* Reactive Programming (RxJava and RxAndroid).
* RESTful client with version, language and authentication (Retrofit).
* Whole app unit tested (Espresso, Mockito, Dagger 2 and MockWebServer).
* Other patterns and Android good practices.
* Continuous Integration system.

If you have any doubts or suggestions don't hesitate to leave a comment.
I'm looking for contributors too ;)
Feel free to use the code for your own purposes!

[(Source code)](https://github.com/jordifierro/android-base)
