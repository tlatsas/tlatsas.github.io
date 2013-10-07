---
layout: post
title: weechat horizontal buffers bar
date: 2011-08-19 10:26:28
tags: [ weechat ]
---
To get a nice clean buffer bar at the bottom of the screen you need the *buffers.pl* script.

First install using weeget:
{% highlight text %}
/weeget install buffers.pl
{% endhighlight %}

Weeget will also load the script automatically for you.

Finaly, set its position at the bottom:
{% highlight text %}
/set weechat.bar.buffers.position bottom
{% endhighlight %}
