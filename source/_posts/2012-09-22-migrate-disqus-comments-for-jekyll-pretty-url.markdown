---
layout: post
title: migrate disqus comments for jekyll's pretty urls
date: 2012-09-22 20:16:05
tags: disqus jekyll
---
I recently enabled the pretty-url feature in Jekyll, which removes the redundant `.html` suffix in
URLs. You can enable it by adding `permalink: pretty` in your `_config.yml`. This of course
broke Disqus comments on all existing pages.

Hopefully, this can be easily fixed using the cool migration tools Disqus provides. The concept is to
write a text file which maps the old URLs to the new ones. The file format should be the like the
following snippet.

{% highlight perl %}
old_url1, new_url1
old_url2, new_url2
{% endhighlight %}

Disqus provides a CSV file which contains all your site's URLs. To download this file, navigate
to your site's admin panel and start the URL mapper tool under tools → migrate threads →
start URL mapper. Then download the CSV by clicking the `download CSV` link.

First we need to get rid of the silly windows CRLF line endings from the CSV file.
A little perl magic can help here.

{% highlight bash %}
perl -pi -e 's/\r\n/\n/g' disqus-comments-old.csv
{% endhighlight %}

Then, we need to format the file accordingly. The new URL is the old one without the `.html` suffix.

{% highlight text %}
http://example.com/post.html, http://example.com/post
{% endhighlight %}

Again using perl, we can cook a simple script to automate the process.

{% highlight perl %}
#!/usr/bin/env perl
use strict;

open(fin, 'disqus-comments-old.csv') or die $!;
open(fout, '>>', 'disqus-comments-new.csv') or die $!;

my $old; #keep the original lines

while(<fin>) {
    chomp;
    $old = "$_";
    s/\.html/\//;
    print fout "$old, $_\n";
}

close(fin);
close(fout);
{% endhighlight %}

I didn't bother to allow passing the filename as a script argument, so I just 'hardcoded'
the filenames in the script. Edit at will and don't forget to run it from the proper path.

Finally, upload the new CSV file to the URL migration tool and you will have your old comments back.
