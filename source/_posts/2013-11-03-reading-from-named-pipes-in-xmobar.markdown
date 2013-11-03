---
layout: post
title: reading from named pipes in xmobar
date: 2013-11-03 17:38:43
tags: xmobar
comments: true
---

I recently noticed that [xmobar]([https://github.com/jaor/xmobar) has a plugin to read data from Unix named pipes. Named pipes can be used for inter-process communication (IPC). Two different application can send and read data using named pipes. A named pipe operates much like the normal (unnamed) pipe you use in the shell. The difference is that named pipes must be explicitly created/deleted and they are accessed through the filesystem. You create named pipes using the `mknod` or `mkfifo` commands and delete them with `rm`.

In this tutorial we will use named pipes to read the current volume levels and display them in xmobar. Xmobar also has an [alsa plugin](https://github.com/jaor/xmobar/blob/master/readme.md#volume-mixer-element-args-refreshrate) but distributions do not always compile xmobar with alsa support. An alternative approach is to write a small script that will parse the output of the `amixer` command and then call it from xmobar in regular intervals to show the current volume levels. This however requires xmobar to frequently call an external command/script. Also if you happen to use a large frequency you might notice some "lag" from the time you change the volume levels to the time this change is reflected in xmobar. Using named pipes this change is instant and you also void redundant calls.

First, we need to create our named pipe. All my startup logic is inside a script which is called from `~/.xinitrc`. For simplicity let's create the pipe inside the `~/.xinitrc` before launching the window manager.

```bash
_volume_pipe=/tmp/.volume-pipe
[[ -S $_volume_pipe ]] || mkfifo $_volume_pipe
```
We create the pipe only if it does not exists. That way we can logout/login without getting any "File exists" errors. My `/tmp` is mounted as tmpfs so the pipes do not survive a reboot.

Next we need to configure xmobar to read from this pipe. Edit your `xmobar.hs` and put in the `command` list:

```haskell
Run PipeReader "/tmp/.volume-pipe" "vol_pipe"
```

This registers the PipeReader plugin to read from `/tmp/.volume-pipe`. It also makes the `vol_pipe` alias available in the output template. The template is used to describe how to display information gathered from the plugins. I use something like this to show the volume levels: `♫ <fc=#b4cdcd>%vol_pipe%</fc>`.

Basically, now you can pretty much feed anything to the xmobar through the pipe and it will display it without hesitation. Just write something to the pipe:

```bash
echo "something" > /tmp/.volume-pipe
```

In order to display the volume information we will create a small script which can be used to increase/decrease/mute/show the volume levels. After each operation it will send the output to the named pipe. I already have [a script](https://github.com/tlatsas/dotfiles/blob/master/bin/alsavol) I use for this purpose, but let's write something simpler.

```bash volume.sh
#!/usr/bin/bash

get_volume() {
  # return volume levels (0-100)
  vol=$(amixer sget Master | grep -o -m 1 '[[:digit:]]*%' | tr -d '%')
  echo ${vol}% | tee /tmp/.volume-pipe
}

case $1 in
  "")
    ;;
  "up")
    amixer set Master 5+ >/dev/null
    ;;
  "down")
    amixer set Master 5- > /dev/null
    ;;
  "toggle")
    amixer set Master "toggle" >/dev/null
    ;;
  *)
    echo "unknown command"
    exit 1
    ;;
esac
get_volume

```

The key point here is to send the output after each operation to the pipe (line 6), a simple redirection will do the trick. We use `tee` to send the volume output both in the stdout and in the named pipe.

The only problem is that when the xmobar first starts, it does not have any data to read from the pipe, so it displays this annoying message: "Updating...". In order to fix that we can run our script a single time after we create the pipe inside the `.xinitrc`.

```bash
_volume_pipe=/tmp/.volume-pipe
[[ -S $_volume_pipe ]] || mkfifo $_volume_pipe
/path/to/script/volume.sh
```

more reading about named pipes and xmobar:

- [named pipes (wikipedia)](https://en.wikipedia.org/wiki/Named_pipe)
- [introduction to named pipes](http://www.linuxjournal.com/article/2156)
- [xmobar documentation](https://github.com/jaor/xmobar/blob/master/readme.md)
