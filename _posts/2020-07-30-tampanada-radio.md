---
layout: post
title:  "Tampanada Radio"
date:   2020-07-30 10:00:00 +0100
categories: project
comments: true
---

![Tampanada Radio](/assets/images/tampanadaradio_icon.png)

[Tampanada](https://llaor.com/llengua/diccionari/mots/tampanada)
means hit in pallarese and gives the name to this little radio station
created by a group of people from Pallars to spread news and culture of that place,
always with a touch of humor.

The [website](http://tampanadaradio.com) offers the visitors
information about the project, as well as the possibility
to listen live streaming and podcasts.

We also decided to develop native mobile applications
to make the radio easier to listen.

# Android

![Main Activity](/assets/images/tampanadaradio_main_activity.png)
![Foreground Service](/assets/images/tampanadaradio_foreground_service.png)

First release is very simple: just a play/pause button and
a foreground service to play online streaming.
It uses google's [ExoPlayer](https://github.com/google/ExoPlayer)
as player engine.
That player is managed by a service,
which communicates its state to the activity broadcasting messages
and also shows a foreground notification with a little player menu.
ExoPlayer's retry policy has been overriden to improve network errors recovery.

Project code is available on [Github](https://github.com/jordifierro/tampanada-android)
and can be easily tunned to create your own radio station (feel free to do it!).
Application is already available on
[Google Play Store](https://play.google.com/store/apps/details?id=com.tampanada.radio).

# iOS

![Main ViewController](/assets/images/tampanadaradio_main_viewcontroller.png)

iOS app was easier to develop in this case because
AVPlayer already handles background player.
I used [ModernAVPlayer](https://github.com/noreasonprojects/ModernAVPlayer)
which is an AVPlayer wrapper that handles retries when network errors occur.

Project code is also available on [Github](https://github.com/jordifierro/tampanada-ios)
and can be easily tunned to create your own radio station too.
Application is already available on
[App Store](https://apps.apple.com/us/app/id1513016413).
