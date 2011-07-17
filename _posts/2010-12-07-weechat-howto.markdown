--- 
layout: post
title: weechat howto
date: 2010-12-07 19:49:52 +02:00
---
http://www.weechat.org/ Weechat (Wee Enhanced Environment for Chat) is a lightweight, extensible, console based irc client. It is written in C and licensed under GNU GPL3.

Launch weechat from console with the weechat-curses command.

One of the most important commands is the <strong>/help</strong> command. So, when in doubt use /help.
Also using the command **/set config.section.option** will print the value of **config.section.option** and if you want to set a value to an option you type **/set config.section.option value**
Anyway, lets start by building our configuration to connect to freenode.

Start by printing all  **irc.server** values.
{% highlight %}/set irc.server.{% endhighlight %}

you will get the response:

{% highlight %}
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
{% highlight %}

As you can see freenode is already in the configuration by default <em>(irc.server.freenode.addresses  = "chat.freenode.net/6667")</em>. If you need to add another server use e.g.:
{% highlight %}/server add oftc irc.oftc.net/6667{% endhighlight %}

Now, all we need to do is set the rest of the values for the freenode server.
To get help for a particular option -including the list of expected values- type e.g. :
{% highlight %}/help irc.server.freenode.autoconnect{% endhighlight %}

So, we want to connect to freenode by default and reconnect automatically in case of connection problems:
{% highlight %}
/set irc.server.freenode.autoconnect on
/set irc.server.freenode.autoreconnect on
{% endhighlight %}

Next we need to automatically join some channels, for multiple channels use commas as separators <strong>without</strong> spaces:
{% highlight %}/set irc.server.freenode.autojoin = "#archlinux,#archlinux-offtopic,#archlinux-greece"{% endhighlight %}

After that we need to set our nicknames (also comma separated), username and realname (these are optional):
{% highlight %}
/set irc.server.freenode.nicks = "nick1,nick2,nick3"
/set irc.server.freenode.username = "my-user-name"
/set irc.server.freenode.realname = "my-real-name"
{% highlight %}

Last but not least set the irc command to identify with the freenode server with :
{% highlight %}/set irc.server.freenode.command = "/msg NickServ identify <your-password-goes-here>"{% endhighlight %}

Save your configuration with the <strong>/save</strong> command and restart weechat, <strong>/EXIT</strong> exits weechat.
Now you should be connected to freenode!

Time for some keybindings!!
<strong>F5</strong> / <strong>F6</strong> : Cycle through the buffers
<strong>PageUp</strong> / <strong>PageDown</strong> : scroll up/down main chat area
<strong>F11</strong> / <strong>F12</strong> : scroll nickname list up/down

If you are running terminator and have problems with F11 interpreted as "go to full screen" add this to your <strong>~/.config/terminator/config</strong> :
{% highlight %}
[keybindings]
  full_screen = Disabled
{% endhighlight %}

If you have the same problem with xfce Terminal go to <strong>Edit -> Preferences -> Shortcuts</strong> and kill the nasty fullscreen shortcut!

Lastly the cool stuff!
Weechat allows you to split the screen and have multiple channels open at the same time. Use the <strong>/window splitv</strong> to vertically split the screen and <strong>/window splith</strong> to horizontally split the screen. You can also provide a percentage to unevenly split the screen and use <strong>/window merge all</strong> to unify all buffers to default.
When you are happy with your window layout save it:

{% highlight %}/layout save{% endhighlight %}

To automatically save window layout on exit use :
{% highlight %}/set weechat.look.save_layout_on_exit both{% endhighlight %}

When in split mode use <strong>F7</strong> / <strong>F8</strong> to cycle through the active windows-buffers.

More resources :
<a href="http://www.weechat.org/files/doc/stable/weechat_quickstart.en.html">Weechat Quickstart</a>
<a href="http://www.weechat.org/files/doc/stable/weechat_user.en.html">Weechat User's Guide</a>
<a href="http://www.wikihow.com/Register-a-User-Name-on-Freenode">How to register a username to freenode</a>
<a href="http://www.weechat.org/scripts/">Weechat scripts</a>
