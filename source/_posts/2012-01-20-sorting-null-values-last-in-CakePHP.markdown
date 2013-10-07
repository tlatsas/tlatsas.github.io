---
layout: post
title: sorting null values last in CakePHP
date: 2012-01-20 18:33:42
tags: [ php , cakephp ]
---
Assume your data contain a field with some NULL and some not NULL values.
If you want to sort your entries against that field in a descending order, NULL
is interpreted as 0 and entries with NULL in that field go at the bottom.
This is perfectly desirable. But what if you want to sort them into an ascending order?
Entries with the NULL field would show up first. In some cases, you might want to sort
entries with actual data in that field first, and let the rest show up last.

To accomplish that in CakePHP you need two entries of the same field in the order array.
Assume our model is **House** (which obviously describes some houses) and we want to sort
against the field **geo_distance** which contains the distance between a particular house and
some other place. As this is an optional field, it might contain NULL values, but we want
to promote the houses which contain distance information in the search results.

{% highlight php startinline %}
$order = array('geo_distance' => 'IS NULL ASC', 'House.geo_distance' => 'ASC');
{% endhighlight %}

Note that you need the model name for the second array entry in order to trick CakePHP into
taking it into account.
