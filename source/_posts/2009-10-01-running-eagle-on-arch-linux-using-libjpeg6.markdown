---
layout: post
title: running eagle on arch linux using libjpeg6
date: 2009-10-01 13:58:30 +03:00
tags: [ archlinux, eagle ]
---
Eagle 5.6 depends on libjpeg6 to run. I case you have upgraded to libjpeg7 there are a few tricks to help you run eagle.

One trick you should not use though is to make a libjpeg6 link, linking to libjpeg7.
This could possibly mess up other application on your system.

Install [libjpeg6](http://aur.archlinux.org/packages.php?ID=28427) from AUR.

If you don't want to install libjpeg6 sytem-wide there is a more elegand way to fix it:
1. grab a copy of libjpeg6
2. put it somewhere inside your home directory or for example inside /opt/eagle/lib
3. edit the launcher script (/usr/bin/eagle) and add before the exec ./eagle command:

{% highlight bash %}
export LD_LIBRARY_PATH=/opt/eagle/lib
{% endhighlight %}

Grab [eagle](http://aur.archlinux.org/packages.php?ID=15941) from AUR.

**Update**: as of 5.6.0-2 version in aur, you do not need to do the above steps as the PKGBUILD will take care everything for you.
