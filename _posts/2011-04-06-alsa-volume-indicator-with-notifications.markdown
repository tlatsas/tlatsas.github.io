---
layout: post
title: alsa volume indicator with notifications
date: 2011-04-06 11:03:05 +03:00
tags: alsa
---
<img class="right" src="http://farm7.static.flickr.com/6134/5958779113_a15ef21a15_m.jpg" alt="osd-screenshot">
I recently [found a thread](http://ubuntuforums.org/showthread.php?p=7241817) in ubuntu forums that explained how to get a gnome-like volume indicator in Xfce.

I took this script and polished it a bit. It now features:

* command line parameter for alsa channel
* command line parameter for volume change step
* proper help screen
* proper stock icon names based on the [freedesktop icon naming specification](http://standards.freedesktop.org/icon-naming-spec/icon-naming-spec-latest.html)

You can get the script from here: [alsavol](https://github.com/tlatsas/utils-scripts/blob/master/alsavol).
