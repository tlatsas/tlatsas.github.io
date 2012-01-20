---
layout: post
title: generate jekyll front-matter with bash
date: 2011-08-31 20:35:11
tags: [ jekyll, bash ]
---
Creating a post in a jekyll-powered blog is pretty straightforward. Just create a file in markdown format and put your post in.
Every post has a special [YAML Front Matter](https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter) block with some variables.
Using a bash script we can easily generate this file with proper values for the YAML variables.
For example, for this post, the filename contains the post date and title.

{% highlight text %}
2011-08-31-generate-jekyll-front-matter-with-bash.markdown
{% endhighlight %}

The same values (date, title) are contained in the YAML block inside the file, but a bit more formatted.
A Bash script can take care the formatting for us by only supplying the post title as the script argument.
This assumes that the date (and optionally time) are set to the current time we run the bash script.

First we need to grab the date and optionaly the time.

{% highlight bash %}
_date=$(date +'%Y-%m-%d')
_datetime=$(date +'%Y-%m-%d %H:%M:%S')
{% endhighlight %}

**\_date** will be used for the filename and **\_datetime** for the _date_ variable.

We also need to replace whitespace with dashes in the post title, for use in the filename.

{% highlight bash %}
_post=$(echo $1 | tr ' ' '-')
{% endhighlight %}


The filename is supplied a an argument, so we might want to bail if the user does not supply one

{% highlight bash %}
if [[ -z $1 ]]; then
    echo "A post title is required. Bye.."
    exit 1
fi
{% endhighlight %}


Also, another safe switch would be to check if the file already exists to prevent accidental overwrites.

To do that we need the absolute path for our post file. Putting all these together we have:

{% highlight bash %}
_title="${_date}-${_post}.markdown"
_cwd=$(pwd)
_post_file="${_cwd}/${_title}"

if [[ -f ${_post_file} ]]; then
    echo "File already exists. Bye.."
    exit 1
fi
{% endhighlight %}

Finally, the magic code that creates our post file uses [here documents](http://tldp.org/LDP/abs/html/here-docs.html)

{% highlight bash %}
cat << EOF >| ${_post_file}
---
layout: post
title: $1
date: $_datetime
---
EOF
{% endhighlight %}

Make sure to avoid quoting of the **EOF** as bash variable subsitution won't work.
Also, note that I used the >| notation because I have the bash [noclobber](http://www.cyberciti.biz/tips/howto-keep-file-safe-from-overwriting.html) variable set.

The [jekyll-post script](https://github.com/tlatsas/utils-scripts/blob/master/jekyll-post) is on github, also added some code to prompt user for launching a text editor.
