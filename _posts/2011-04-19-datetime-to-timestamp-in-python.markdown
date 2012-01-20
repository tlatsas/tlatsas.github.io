--- 
layout: post
title: datetime to timestamp in python
date: 2011-04-19 08:51:27 +03:00
tags: [ python ]
---
Say you got a file's last update time, using the stat command
{% highlight python %}
>>> import os
>>> disk_ts = os.stat('filename').st_ctime
>>> disk_ts
1303113637.0
{% endhighlight %}

and you want to compare it with a datetime object:
{% highlight python %}
>>> import datetime
>>> dt = datetime.datetime(2011, 2, 23, 13, 48, 0, 499000)
>>> dt
datetime.datetime(2011, 2, 23, 13, 48, 0, 499000)
{% endhighlight %}

Convert the datetime object using the time module:
{% highlight python %}
>>> import time
>>> dtime_ts = time.mktime(dt.timetuple())
>>> dtime_ts
1298461680.0
{% endhighlight %}

now compare and rejoice! Also, if you want to display timestamps in a human-friendly format use time.ctime
{% highlight python %}
>>> time.ctime(disk_ts)
'Mon Apr 18 11:00:37 2011'
>>> time.ctime(dtime_ts)
'Wed Feb 23 13:48:00 2011'
{% endhighlight %}
