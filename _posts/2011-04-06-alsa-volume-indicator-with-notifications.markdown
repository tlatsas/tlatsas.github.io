--- 
layout: post
title: alsa volume indicator with notifications
date: 2011-04-06 11:03:05 +03:00
---
I recently <a href="http://ubuntuforums.org/showthread.php?p=7241817">found a thread</a> in ubuntu forums that explained how to get a gnome-like volume indicator in Xfce.

I took this script and polished it a bit. It now features:

<ul>
	<li>command line parameter for alsa channel</li>


	<li>command line parameter for volume change step</li>


	<li>proper help screen</li>


	<li>proper stock icon names based on the <a href="http://standards.freedesktop.org/icon-naming-spec/icon-naming-spec-latest.html">freedesktop icon naming specification</a></li>
</ul>

You can get the script from here <a href="https://github.com/tlatsas/scripts/blob/master/amixer-osd.sh">amixer-osd.sh</a>

