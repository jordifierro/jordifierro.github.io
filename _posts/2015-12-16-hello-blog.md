---
layout: post
title:  "Hello blog!"
date:   2015-12-16 14:44:41 +0100
categories: blog
comments: true
---

I was thinking on introduce myself in this first post... But I thought: "There is no better way to start than writing a meta-post!". Therefore, this is about how this blog has been created (I've also written some words about myself [here]({{site.url}}/about)).

This blog is powered by Jekyll and GitHub Pages.
[Jekyll](http://jekyllrb.com)
is an static site generator that allows you to create a blog easily.
[GitHub Pages](https://pages.github.com)
is a hosting service for static content that permits
 the deployment with a simple git push.
 Furthermore, Jekyll comes integrated with GitHub Pages
 so you can push the source code to GitHub and
 it automatically generates and deploys it.

![Jekyll + GitHub Pages](/assets/images/jekyll_github.png)

#### Less

* Time: Really fast to setup and deploy.
* Money: GitHub gives a free page site to every user.
* Headaches: There's no dynamic content, code or database to take care of.
* Latency: Static sites are always faster than dynamic ones.

#### More

* Knowledge: You will learn because you program it (as you wish), generate the content with markdown language and use git.
* Security: Neither database nor running code on server.
* Control: You manage your content changes with git.

I'm not claiming this approach is the best way to create a generic blog, as always it depends on your constraints. However, for my particular case I can say it's perfect. Try it for yourself!

## How to create your own blog

### 1 - Install Jekyll and create the blog
If you have Ruby, RubyGems and NodeJs installed (otherwise, check [this](http://jekyllrb.com/docs/installation/) first):

{% highlight bash %}
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve
{% endhighlight %}

At this point, you must be able to see a default template at `http://localhost:4000`. Time to play with it! (check the [documentation](http://jekyllrb.com/docs/structure/)
about the structure too)

Once you have shaped it at your taste...

### 2 - Deploy the code to github pages

First of all, we need a [GitHub account](https://github.com/)
and create a repository called `yourusername.github.io`
(must be called exactly like that).
Then upload your blog source code to that remote repository:

{% highlight bash %}
~/myblog $ git init
~/myblog $ git commit -m "first commit"
~/myblog $ git remote add origin https://github.com/yourusername/yourusername.github.io.git
~/myblog $ git push -u origin master
{% endhighlight %}

That's it, your blog must be available at `yourusername.github.io`.
You can also check the source code of
[this blog](https://github.com/jordifierro/jordifierro.github.io)
and
[other blogs](https://github.com/jekyll/jekyll/wiki/Sites)
like this.

If you want to keep improving your blog, read
[part II](/building-blog/).

Happy blogging!
