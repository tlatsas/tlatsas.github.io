--- 
wordpress_id: 55
layout: post
title: set mysql encoding to utf8
date: 2009-07-28 12:12:17 +03:00
wordpress_url: http://ttys.homelinux.net/?p=55
---
This one troubled me for a long time and it also resulted in a corrupted database at some point, because i changed collation encoding to utf8 but the database encoding was latin..(always make backups and make sure you don't erase them :-P )

Before doing the following make sure you backup any databases as this could possibly corrupt them. If you need to change the encoding of your database data, one possible solution would be to use mysqldump to dump the database with the latin (or whatever) encoding and then use the http://www.gnu.org/software/libiconv/documentation/libiconv/iconv.1.html iconv utility to change that encoding to utf8.Then change mysql encoding to utf8 (as follows) and then import the database with the new encoding. I haven't tested this myself though..so don't blame me!

So to set everything to utf8 encoding in mysql add the following lines to the [mysqld] section of my.cnf

{% highlight text %}
init-connect = 'SET NAMES utf8'
character-set-server = utf8
collation-server = utf8_general_ci
default-character-set = utf8
default-collation = utf8_general_ci
{% endhighlight %}

Also you need the following line under the [client] section of my.cnf
{% highlight text %}
default-character-set = utf8
{% endhighlight %}

then restart mysql.
You can verify the used encodings by loging in to mysql and running the following commands in cli:

{% highlight mysql %}
mysql> show variables like '%server%';
mysql> show variables like '%characters%';
mysql> show variables like '%collation%';
{% endhighlight %}
