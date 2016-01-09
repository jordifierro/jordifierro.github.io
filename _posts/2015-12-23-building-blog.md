---
layout: post
title:  "Building the blog"
date:   2015-12-23 16:45:00 +0100
comments: true
---

Hi there! I keep developing my personal Jekyll blog,
and I would like to share with you these firsts steps.
If you are new to Jekyll blogging,
start reading [part I](/hello-blog/).

![Building a blog](/assets/images/building_blog.jpg)

Today I'm going to write about:

* Permalinks
* Domain
* Analytics
* Comments

### Permalinks

With this blog system, post link format is centrally managed.
Thus, you could deal with it later...
if it wasn't that when you change your linkage system you will broke the old links.
That's why they're called [permalinks](https://en.wikipedia.org/wiki/Permalink){:target="_blank"}
(permanent links).

I've decided to keep my permalinks format as simple as possible
adding this to my `_config.yml` file:

{% highlight ruby %}
permalink: /:title
{% endhighlight %}

Date and categories have been removed in order to
increase the readability of the urls
(I can keep using date and categories for other purposes).

You can use the format that pleases you most
(check the [main options](http://jekyllrb.com/docs/permalinks/){:target="_blank"}),
so take your time to decide which is better for your needs,
but remember to do it before start spreading links!

### Domain

Domain is also a part of the permalink (an important one, actually),
so it makes no sense to define the link format
and start blogging with a provisional domain.

* Buy it at your favorite domain website.
* Define it as an alias for `yourusername.github.io`
(usually `CNAME record`, but every domain seller has its own configuration).
* Create a file exactly named `CNAME` on your project root folder and
write `youdomain.com` inside it.
* Push the changes to github.

### Analytics

In order to track the usage information of my blog,
I've added the Google Analytics system.
It's very easy to setup too.

* Just sign up into Google Analytics,
create a new account (or a new project if you already have one)
and get a piece of code like this:

{% highlight html %}
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'XX-XXXXXXXX-X', 'auto');
  ga('send', 'pageview');

</script>
{% endhighlight %}

* Add it to `/_includes/google_analytics.html`.

* And finally, modify `/_layouts/default.html`:

{% highlight html %}
{% raw %}
    ...
    </body>

    {% include google_analytics.html %}

</html>
{% endraw %}
{% endhighlight %}

After a while, it should start appearing data on your analytics dashboard.

### Comments

And finally, the comments system.
I've chosen [Disqus](https://disqus.com){:target="_blank"}
because it's highly used and very easy to integrate and manage,
but you are free to use other services or even not to add comments at all.

If you choose this one, you can use this brief setup summary:

* Sign up at [Disqus](https://disqus.com){:target="_blank"} and create your site.
* Go to `Installation -> Universal Code`
and copy the code to `_includes/disqus.html`, replacing these lines:

{% highlight ruby %}
{% raw %}
this.page.url = PAGE_URL;
this.page.identifier = PAGE_IDENTIFIER;
{% endraw %}
{% endhighlight %}

with:

{% highlight ruby %}
{% raw %}
this.page.url = "{{site.url}}{{page.url}}";
this.page.identifier = "{{page.url}}";
{% endraw %}
{% endhighlight %}


* Include the following code where you want to add the comments
(I've added it at the end of `_layouts/post.html`):

{% highlight ruby %}
{% raw %}
...

{% if page.comments %}
{% include disqus.html %}
{% endif %}
{% endraw %}
{% endhighlight %}

* And finally, add

{% highlight ruby %}
---
...
comments: true
---
{% endhighlight %}

on the posts or pages you want to make the comment system available.
Set to false or don't add it at all elsewhere.

And that's it! More blog improvements coming soon...
