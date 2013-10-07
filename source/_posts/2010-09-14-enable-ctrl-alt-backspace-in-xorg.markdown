--- 
layout: post
title: Enable Ctrl-Alt-Backspace in Xorg
date: 2010-09-14 12:11:25 +03:00
tags: Xorg
---
Add
{% highlight text %}
Option             "XkbOptions" "terminate:ctrl_alt_bksp"
{% endhighlight %}
to your Keyboard InputClass to enable Ctrl+Alt+Backspace to kill your x server
