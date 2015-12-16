---
layout: post
title:  "Hello world!"
date:   2015-12-16 14:44:41 +0100
categories: jekyll update
---

I was thinking on introduce myself in this first post... But I thought: "There is no better way to start than writing a meta-post!". So this is about how this blog has been created (I've also written some [words about myself]({{site.url}}/about)).

This blog is powered by Jekyll and Github Pages. [Jekyll](http://jekyllrb.com) is an static site generator that allows you to create a  blog on a super easy way. [Github Pages](https://pages.github.com) is a hosting service for static content that allows you to deploy with a simple push to github. Furthermore, Github Pages has Jekyll integration, so you can push the Jekyll blog source code to github and it automatically generates and deploys it.

### Less

* Time: Really fast to setup and deploy.
* Money: Github gives a free page site for each user.
* Headaches: There's no dynamic content, code or database to take care of.
* Latency: Static sites are always faster than dynamic.

### More

* Knowledge: You will learn: program it as you want,  generate the content with markdown language and use git.
* Security: Neither database nor running code on server.
* Control: All content changes are on github.

I'm not claiming this approach is the best way to create a generic blog, as always it depends on you constraints. Although for my particular and others cases, I can say it's fantastic. Try it for yourself!

#### 1 - Install Jekyll and create the blog
If you have Ruby, RubyGems and NodeJs installed (if not check [this](http://jekyllrb.com/docs/installation/)):

{% highlight bash %}
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve
# => Now browse to http://localhost:4000
{% endhighlight %}

At this point, you must be able to see a default template at `port 4000` of your `localhost`. Time to play with it! (check the [documentation](http://jekyllrb.com/docs/structure/) about the structure too)

Once you have shaped it at your taste...

#### 2 - Deploy the code to github pages

First of all, we need a [Github account](https://github.com/) and create a repository called `yourusername.github.io` (must be called exactly like that). Then update your code to that remote repository:

{% highlight bash %}
~/myblog $ git init
~/myblog $ git commit -m "first commit"
~/myblog $ git remote add origin https://github.com/yourusername/yourusername.github.io.git
~/myblog $ git push -u origin master
{% endhighlight %}

That's it, your blog must be available at `yourusername.github.io`.
You can also check the source code of [this](https://github.com/jordifierro/jordifierro.github.io) blog and [others](https://github.com/jekyll/jekyll/wiki/Sites) like this.

Thanks for reading and enjoy!
