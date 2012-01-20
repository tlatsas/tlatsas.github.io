---
layout: post
title: weechat howto
date: 2010-12-07 19:49:52 +02:00
tags: weechat
---
<img class="right" alt="weechat" src="http://farm7.static.flickr.com/6125/5959339312_4ef9321ec2_m.jpg" />
[Weechat](http://www.weechat.org/ "weechat") (Wee Enhanced Environment for Chat) is a lightweight, extensible, console based irc client. It is written in C and licensed under GNU GPL3.

Launch weechat from console with the weechat-curses command.

One of the most important commands is the **/help** command. So, when in doubt use /help.
Also using the command **/set config.section.option** will print the value of **config.section.option** and if you want to set a value to an option you type **/set config.section.option value**
Anyway, lets start by building our configuration to connect to freenode.

Start by printing all  **irc.server** values.
{% highlight text %}/set irc.server.{% endhighlight %}

you will get the response:

{% highlight text %}
[server]
irc.server.freenode.addresses  = "chat.freenode.net/6667"
irc.server.freenode.autoconnect
irc.server.freenode.autojoin
irc.server.freenode.autoreconnect
irc.server.freenode.autoreconnect_delay
irc.server.freenode.autorejoin
irc.server.freenode.autorejoin_delay
irc.server.freenode.command
irc.server.freenode.command_delay
irc.server.freenode.ipv6
irc.server.freenode.local_hostname
irc.server.freenode.nicks
irc.server.freenode.password
irc.server.freenode.proxy
irc.server.freenode.realname
irc.server.freenode.sasl_mechanism
irc.server.freenode.sasl_password
irc.server.freenode.sasl_timeout
irc.server.freenode.sasl_username
irc.server.freenode.ssl
irc.server.freenode.ssl_cert
irc.server.freenode.ssl_dhkey_size
irc.server.freenode.ssl_verify
irc.server.freenode.username
{% endhighlight %}

As you can see freenode is already in the configuration by default _(irc.server.freenode.addresses  = "chat.freenode.net/6667")_. If you need to add another server use e.g.:
{% highlight text %}/server add oftc irc.oftc.net/6667{% endhighlight %}

Now, all we need to do is set the rest of the values for the freenode server.
To get help for a particular option -including the list of expected values- type e.g.:
{% highlight text %}/help irc.server.freenode.autoconnect{% endhighlight %}

So, we want to connect to freenode by default and reconnect automatically in case of connection problems:
{% highlight text %}
/set irc.server.freenode.autoconnect on
/set irc.server.freenode.autoreconnect on
{% endhighlight %}

Next we need to automatically join some channels, for multiple channels use commas as separators **without** spaces:
{% highlight text %}/set irc.server.freenode.autojoin = "#archlinux,#archlinux-offtopic,#archlinux-greece"{% endhighlight %}

After that we need to set our nicknames (also comma separated), username and realname (these are optional):
{% highlight text %}
/set irc.server.freenode.nicks = "nick1,nick2,nick3"
/set irc.server.freenode.username = "my-user-name"
/set irc.server.freenode.realname = "my-real-name"
{% endhighlight %}

Last but not least set the irc command to identify with the freenode server with :
{% highlight text %}/set irc.server.freenode.command = "/msg NickServ identify <your-password-goes-here>"{% endhighlight %}

Save your configuration with the **/save** command and restart weechat, **/EXIT** exits weechat.
Now you should be connected to freenode!

Time for some keybindings!!
* __F5__ / __F6__ : Cycle through the buffers
* __PageUp__ / __PageDown__ : scroll up/down main chat area
* __F11__ / __F12__ : scroll nickname list up/down

If you are running terminator and have problems with F11 interpreted as "go to full screen" add this to your **~/.config/terminator/config** :
{% highlight text %}
[keybindings]
  full_screen = Disabled
{% endhighlight %}

If you have the same problem with xfce Terminal go to <strong>Edit -> Preferences -> Shortcuts</strong> and kill the nasty fullscreen shortcut!

<a href="http://www.flickr.com/photos/tlatsas/5958779741/in/set-72157627118802299/"><img class="left" alt="weechat split screens" src="http://farm7.static.flickr.com/6129/5958779741_0230c56db1_m.jpg" /></a>

Lastly the cool stuff!
Weechat allows you to split the screen and have multiple channels open at the same time. Use the <strong>/window splitv</strong> to vertically split the screen and <strong>/window splith</strong> to horizontally split the screen. You can also provide a percentage to unevenly split the screen and use <strong>/window merge all</strong> to unify all buffers to default.
When you are happy with your window layout save it:

{% highlight text %}/layout save{% endhighlight %}

To automatically save window layout on exit use :
{% highlight text %}/set weechat.look.save_layout_on_exit all{% endhighlight %}

When in split mode use **F7** / **F8** to cycle through the active windows-buffers.

More resources:
* [Weechat Quickstart](http://www.weechat.org/files/doc/stable/weechat_quickstart.en.html)
* [Weechat User's Guide](http://www.weechat.org/files/doc/stable/weechat_user.en.html)
* [How to register a username to freenode] (http://www.wikihow.com/Register-a-User-Name-on-Freenode)
* [Weechat scrips](http://www.weechat.org/scripts/)

