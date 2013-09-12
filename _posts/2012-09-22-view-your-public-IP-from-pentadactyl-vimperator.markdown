---
layout: post
title: view your public IP from pentadactyl/vimperator
date: 2012-09-22 09:41:07
tags: pentadactyl vimperator firefox
---
[Pentadactyl](http://5digits.org/pentadactyl/) and [vimperator](http://www.vimperator.org/vimperator)
extensions allow you to write handy functions in javascript. These
functions can be later called from the command mode or from keybindings.

Some time ago, I wrote a simple function, that makes a request to a page which returns your
public ip address. That way you don't have to actually visit that page and interrupt your
flow of work. I find this extremely useful as I frequentlty use VPNs and SOCKS proxies.

The js code is the following:

{% highlight javascript %}
function ip() {
    var req = new XMLHttpRequest();
    req.open('GET', 'http://ipz.herokuapp.com/', true);

    req.onreadystatechange = function (ev) {
        if (req.readyState == 4) {
            try {
                dactyl.echo(req.responseText);
            }
            catch (err) {
                dactyl.echo(err);
            }
        }
    }
    req.send(null);
}
{% endhighlight %}

I used a very simple service I wrote and is currently deployed at heroku,
[ipz](https://github.com/tlatsas/ipz). You can use any of the popular services
like [icanhazip](http://www.icanhazip.com/) or [checkip](http://checkip.dyndns.org)
service from dyndns.

In order for this to work, you need to put the function in your `~/.pentadactylrc` enclosed
in a here-document block:

{% highlight bash %}
javascript << EOF
 function-here
EOF
{% endhighlight %}

I also set a command to call this function by typing `:ip`, the command is:

{% highlight bash %}
command ip -js ip()
{% endhighlight %}

In order for this to work in vimperator, we need some adjustments. Put the following code in your
`~/.vimperatorrc`.

{% highlight javascript %}
    ip = function() {
        var req = new XMLHttpRequest();
        req.open('GET', 'http://ipz.herokuapp.com/', true);

        req.onreadystatechange = function (ev) {
            if (req.readyState == 4) {
                try {
                    liberator.echo(req.responseText);
                }
                catch (err) {
                    liberator.echoerr(err);
                }
            }
        }
        req.send(null);
    }
{% endhighlight %}

Again don't forget the here-document block and the command, which are a bit different.

{% highlight bash %}
:js << EOF
 function-here
EOF
{% endhighlight %}

{% highlight bash %}
command! ip js ip()
{% endhighlight %}
