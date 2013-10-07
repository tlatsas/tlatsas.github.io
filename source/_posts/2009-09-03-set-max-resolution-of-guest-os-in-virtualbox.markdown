---
layout: post
title: set max resolution of guest os in virtualbox
date: 2009-09-03 10:24:39 +03:00
tags: virtualbox
---
There is a possibility that even if you have installed the guest additions in virtualbox, the resolution of the guest OS won't exceed a certain value.
If that's the case you can use the VBoxManage command to set the max resolution like this :

{% highlight bash %}
VBoxManage setextradata global GUI/MaxGuestResolution X,Y
{% endhighlight %}
where X,Y the desired resolution. e.g. 1680,1050
