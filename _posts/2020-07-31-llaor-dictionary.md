---
layout: post
title:  "Llaor dictionary"
date:   2020-07-31 10:00:00 +0100
categories: project
comments: true
---

[Llaor](https://llaor.com/llengua/diccionari/mots/llaor)
means seed in Pallars and this project aims to collect and spred pallarese language and culture.
First step to achieve this has been to dump two dictionary books to a database
and present that info (more than 2000 word definitions) on a web application.
Your can visit it at [llaor.com](https://llaor.com). Here you have a preview:

![Home](/assets/images/llaor_web_home.png)

![Search](/assets/images/llaor_web_search.png)

Dictionary will be enhanced and other sections are on their way...
but for the moment, let's talk about the development process of this little dictionary.

## Database

![Books](/assets/images/llaor_books.jpg)

Data modeling is usually done having in mind the purpose of the information 
and how we will work with and present it...
But this case is different. We have a few dictionary books (there are no more than 10 of them)
and new information is difficult to collect nowadays (need philology proficiency and a methodical process)
so we can asume that we have a finite amount of information.

Once I realized that, I decided to dump the definition trying to lose as less information as possible.
Instead of starting with the user interface and then modeling the data according that UI,
here the process is the inverse: first try to define the best model to fit all the information
and then see the possibilities to work and show it.
Moreover, the better model definition, the higher the UI possibilities!

So the first think I did was create a model to fill the db with definitions...
but how is a dictionary word modeled?

![Book A](/assets/images/llaor_book_1.jpg)

![Book B](/assets/images/llaor_book_2.jpg)

As you can see, the two dictionaries I picked to start with are very different.
I tried to detect all the features of different words
to define a model to fit all the information... until I got crazy!
So, again, I decided to do it backwards: start with a simple model
and modify it while introducing each definition manually.

The final model after all entries were created was this:

```python
class Definition(models.Model):

    # data
    word = models.CharField(max_length=30)
    phonetic = models.CharField(max_length=60, blank=True)
    scientific = models.CharField(max_length=60, blank=True)
    type = models.CharField(max_length=30, blank=True)
    meaning = models.TextField()
    extra_info = models.TextField(blank=True)
    private_notes = models.TextField(blank=True)
    synonyms = models.CharField(max_length=60, blank=True)
    related = models.CharField(max_length=60, blank=True)
    origin = models.CharField(max_length=60, blank=True)
    semantic_field = models.CharField(max_length=60, blank=True)

    # metadata
    semantic_group = models.PositiveSmallIntegerField(default=1)
    source = models.CharField(max_length=60)
    reviewed = models.BooleanField(default=True)
    public = models.BooleanField(default=True)
``` 

A word can have multiple definitions or meanings, so model is a definition
(we will see later how to deal with that).

Phonetic contains how the word is pronounced.

Scientific its scientific name.

Type's about if it's a noun, verb...

Meaning contains the word meaning itself.

Extra info and private notes were created to fit some extra data.

Synonims and related link to other words.
I introduce them as comma separated strings to ease the creation form
but as we will see later we return them as a list of words.

Origin points the place where the word was collected.

Semantic field indicates the section of the word (agriculture, animals, people...).

Semantic group is an important field that allows multiple definitions for the same word.

Source links to the source book reference.

Reviewed is used to mark when a word is ready to be published
and public flag define if a definition will be visible from outside
(some definitions appear on both books so we need a mechanism
to choose the best while keeping the other on the database).

## Api

Once the database was created, next step was expose it to the world.
I chose Django framework to create a RESTful api.
You can view its source code on its [github repo](https://github.com/jordifierro/llaor-api).

I had a database full with definitions. But is a definition a useful entity?
Maybe yes, but I decided to tweak it a little bit.
Instead of working with definitions directly,
I found that a word with its meanings could be a better entity package.
Thus, definition/word turns to be the identifier.
For example, if we have this definitions database (simplified version):
```
definition 1:
    word: left
    meaning: past of leave
definition 2:
    word: left
    meaning: opposite of right
```
then api should return for a `/words/left` GET call:
```json
{
    "word": "left",
    "meanings": [
        "past of leave",
        "opposite of right"
    ]
}
```

That fit in my mind so I got to work.
I used [clean architecture in django](https://jordifierro.dev/django-clean-architecture)
to work database definitions and transform them to word entities.
Once that was done, I was able to easily introduce Elasticsearch as search engine
to enable search by full text and also by first letter (to create a cool ABC list).

## Web

There is no mistery in the web.
It's been done using react and simply try to display api words in a beautiful way,
both in ABC list and a text search pages.
The rest of the pages are informative sections.
You can visit it on [llaor.com](https://llaor.com)
and view its source code on its [github repo](https://github.com/jordifierro/llaor-web).

![Home](/assets/images/llaor_mobile_home.png)
![Letters](/assets/images/llaor_mobile_letters.png)

## Infrastructure

Database, api and web run on a dedicated server
(not elasticsearch, that runs on a third party service because of its memory requirements)
handled by docker, postgres, nginx and haproxy.
Api containers are duplicated and load balanced to allow zero downtime deployments.
Jenkins is used as ci/cd (hooked with github), to backup and restore database dumps
and also to trigger massive search engine indexations.


I hope you enjoyed this article. Any comments are welcome!
