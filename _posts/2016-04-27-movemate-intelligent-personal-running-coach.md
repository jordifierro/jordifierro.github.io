---
layout: post
title:  "Movemate intelligent personal running coach"
date:   2016-04-27 10:00:00 +0100
categories: projects
comments: true
---

[Movemate](http://www.movemateapp.com/)
is an intelligent personal running coach application.
Presented in the form of a mobile application, it aims to help runners
training in a more professional way.
It plans and adapts the workouts to the user's schedule, goals and capabilities.
The experience is meant to be as human and personalized as possible,
with the main purpose of helping people to live a healthy live.

![Victor Mier](/assets/images/movemate.jpg)

That is the brief summary of the project,
which made me involve quickly once
[Victor Mier](https://twitter.com/victormier) described it to me.
Victor, in addition to being an excellent engineer, an incredible mountain
runner and a great friend, is the Movemate founder who gave me the opportunity
of joining this project. The team was completed by an awesome graphic designer:
[Eva Blanes](https://twitter.com/eva_blanes).

We spent a few weeks working on the product and then started developing the
first minimum viable product that aimed to start analyzing real workouts and
lay the foundations to begin the iterative development of the final product.
That MVP was an Android application,
which let the user register to the system and start tracking workouts.
After each training, the user could view its statistics and also the route
made over a map.

I was in charge of the Android development, which was interesting and complete.
It was a RESTful client application that consumes data from an api.
The api was developed by Victor but I learnt a lot from it too.
Apart from sending and consuming data, the app also has other interesting parts.

![Movemate screenshots](/assets/images/movemate_screenshots_1.png)

The development of a service responsible to track a gps route
is the most important piece of this application. That involves using a service
that runs in a different process, showing foreground notifications,
storing the geopoints with a certain format internally in the device
and the posterior synchronization with the server.
The application was rounded by the integration with
[Mapbox](https://www.mapbox.com/)
(a map render api service) and an accurate implementation of the design.

![Movemate screenshots](/assets/images/movemate_screenshots_2.png)

The project is still in progress, so if you are interested on it
don't hesitate to ask [Victor](https://twitter.com/victormier)
or [apply for the beta](http://www.movemateapp.com/).
Unfortunately, I left the project when it required full involvement
for a long time period, which I could not devote.

Just wanted to share with you this little great experience and
give information about this brilliant project.
I hope you liked it!
