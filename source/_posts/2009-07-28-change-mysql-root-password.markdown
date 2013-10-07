---
layout: post
title: change mysql root password
date: 2009-07-28 12:33:16 +03:00
tags: mysql
---
If you have forgotten your mysql root password all you need is the root password of the system to create a new one.

First, stop mysql and then start it in the background with the skip-grant-tables parameter:
{% highlight bash %}
mysqld_safe --skip-grant-tables &
{% endhighlight %}

Then login to mysql as root (you don't need a pass anymore)
{% highlight bash %}
mysql -uroot
{% endhighlight %}

and type the following:
{% highlight mysql %}
mysql> use mysql;
mysql> update user set password=PASSWORD(your-new-root-pass) where User='root';
mysql> flush privileges;
{% endhighlight %}

Don't forget to kill the mysql process when you are done and restart it properly.
