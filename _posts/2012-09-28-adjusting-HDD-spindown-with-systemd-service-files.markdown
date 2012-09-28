---
layout: post
title: adjusting HDD spindown with systemd service files
date: 2012-09-28 15:18:28
---

The preferred way to run on-shot commands after boot when using Archlinux + initscripts,
is placing them in the `rc.local` file. Also, initscripts provide a handy helper which creates a
service file that runs these commands on boot when using systemd. If you want to completely
move away from initscripts, to a pure systemd-based system, you need to create service
file(s) for these commands.

The following is an example service file that adjust the HDD spindown levels.

{% highlight ini %}

[Unit]
Description=Fix excessive HDD parking frequency

[Service]
Type=oneshot
ExecStart=/sbin/hdparm -B 220 /dev/sda
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

{% endhighlight %}

The most notable options here are the `oneshot` service type and the `RemainAfterExit` option.
The `oneshot` service type is used to indicate that the command should exit before starting any
follow-up units. We also set the `RemainAfterExit` option to `true` to indicate that this service
should be considered active after it exits. Finally, keep in mind that you need the full command
path in `ExecStart`.
