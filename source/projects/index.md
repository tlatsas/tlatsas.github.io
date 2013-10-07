---
title: projects
layout: page
---

personal projects
-----------------

### [xcolors][xcolors]
Source code for [xcolors.net][xcolors-net]. A flask-powered color theme directory for
Xresources-aware terminals. Contains a parser that generates the theme templates when the flask
starts, much like a static page generator, but instead of html it generates jinja2 templates.

### [omup][omup]
Command line ompldr.org uploader written in Python.
Does not depend on cURL or other external modules.

### [line-coding][lc]
A set of functions that emulate popular line coding techniques line NRZ and Manchester. Run on
Matlab and Octave. You supply an array of 1s and 0s and get a [nice plot][lc-wiki].

### [darui][darui]
Searches RSS feeds and matches entries with sets of keywords, then emails results.

### [ipz][ipz]
A service that shows your public IP address, it also supports json responses.
Written in Node.js, [deployed at heroku][ipz-h].

### [jinja2-highlight][j2h]
A jinja2 extension that uses Pygments to highlight source code blocks. Tested with flask framework.

### [sensors-test for android][sensors]
Android application that displays data from the device's accelerometer and magnetic sensors.
Also calculates Azimuth, Pitch and Roll of the device based on the values from the sensors.

### [baisho][baisho]
A simple wrapper over `dd` and `cdrkit` I wrote some time ago. Makes ripping and burning
iso images easier.

configs / scripts
-----------------

### [PKGBUILDS][pkg]
Pkgbuilds for packages I maintain in [AUR][aur].

### [dotfiles][dot]
Configuration files from `~`. I like the simplicity of window managers over full DEs, so having
all configuration files in one place is handy.

{:.del}
### [utilities][util]
Various useful little scripts. Now merged in [dotfiles][dot].

master thesis
-------------

"Incident Identification Platform Based on Mobile Devices"

The incident locator platform aims to detect incidents (such as forest fires) by utilizing reports sent from mobile clients. We assume that the clients know only the approximate location of such incidents. Each client sents its coordinates in terms of longitude and latitude and the general direction where the device is pointing at. The platform tries to combine multiple reports in order to accurately determine the incident's location on the map. This is a pilot implementation and currently only the Android platform is supported. The server and the client code is freely available under the 3-clause BSD license. 

All sources are under the [incident locator organization][ilocp] on github.


[xcolors]: https://github.com/tlatsas/xcolors
[xcolors-net]: http://xcolors.net/
[omup]: https://github.com/tlatsas/omup
[lc]: https://github.com/tlatsas/line-coding
[lc-wiki]: https://github.com/tlatsas/line-coding/wiki
[darui]: https://github.com/tlatsas/darui
[ipz]: https://github.com/tlatsas/ipz
[ipz-h]: http://ipz.herokuapp.com/
[j2h]: https://github.com/tlatsas/jinja2-highlight
[sensors]: https://github.com/tlatsas/sensors-test-android
[baisho]: https://github.com/tlatsas/baisho
[pkg]: https://github.com/tlatsas/pkgbuilds
[dot]: https://github.com/tlatsas/dotfiles
[util]: https://github.com/tlatsas/utils-scripts
[aur]: https://aur.archlinux.org/packages.php?SeB=m&K=tasidus
[ilocp]: https://github.com/ilocp
