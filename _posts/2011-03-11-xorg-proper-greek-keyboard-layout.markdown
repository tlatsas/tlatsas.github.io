---
layout: post
title: xorg - proper greek keyboard layout
date: 2011-03-11 09:53:39 +02:00
tags: Xorg
---
I was getting recently a lot random crashes with xfce4-xkb-plugin.
Also when I opened the plugin properties the greek layout variant text was all scrambled and ineligible.
Having searched through the internet and bug trackers I couldn't find something similar so I started suspecting that my setup was broken.
A look in _**/usr/share/X11/xkb/rules/xorg.lst**_ showed me that the greek layout was 'gr' and not 'el' as I had specified it.

You can get the greek layout code with:
{% highlight bash %}grep -i 'greece' /usr/share/X11/xkb/rules/xorg.lst{% endhighlight %}

and the keyboard variants with:
{% highlight bash %}grep 'gr:' /usr/share/X11/xkb/rules/xorg.lst{% endhighlight %}

so this is how my (correct) keyboard settings look:

{% highlight text %}
Section "InputClass"
  Identifier       "Keyboard Defaults"
  MatchIsKeyboard  "yes"
  Option           "XkbLayout" "us, gr"
  Option           "XkbVariant" ",extended"
  Option           "XkbOptions" "grp:alt_shift_toggle, terminate:ctrl_alt_bksp,
lv3:ralt_switch_multikey, eurosign:e"
EndSection
{% endhighlight %}

(The lv3 multikey option is used to type ligatures see: <a href="http://shtrom.ssji.net/skb/xorg-ligatures.html">http://shtrom.ssji.net/skb/xorg-ligatures.html</a>)
