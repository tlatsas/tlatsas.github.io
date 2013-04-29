---
layout: post
title: installing Arch Linux ARM in Raspberry Pi
date: 2013-04-29 13:44:38
---

This tutorial guides you through the process of installing the ARM flavour of
Arch Linux in a RaspberryPi. This is a headless installation procedure,
no monitor/tv or keyboard are required.


### Get Arch Linux ARM

You can download the image using a direct download link or torrent. The latest version
available (at this time) is the `archlinux-hf-2013-02-11`. After downloading the zip archive containing the
image verify the SHA-1 checksum. First download the `archlinux-hf-2013-02-11.zip.sha1` file
in the same path as the zip archive and then run:

{% highlight text %}
$ sha1sum -c archlinux-hf-2012-09-18.img.sha1
{% endhighlight %}


### Install on SD card

The installation process is pretty straighforward. Insert the SD card in your computer
and transfer the image to the card using the `dd` command as root. As always, pay attention
to the supplied destination device. Make sure you use the one corresponding to the SD card
or you could trash all your data on other devices. Also make sure that the destination device
is not mounted. You can use the `lsblk` command to verity that. We will just use `sdX` as an example.

{% highlight text %}
# dd bs=1M if=archlinux-hf-2013-02-11.img of=/dev/sdX
{% endhighlight %}


### Expanding the root partition

After the image transfer is complete two partitions are created. The `sdX1` partition will be mounted
at `/boot` and the filesystem is `VFAT`. The other partition, `sdX2` holds the root filesystem and is
formatted as `ext4`.

The root filesystem is a little smaller than 2GB. If the SD card is bigger (it should be) we need to expand
this partition to fill the remaining space. Alternatively we could create a second filesystem and mounted at
a mount point of our choice. The creation of a new filesystem is really simple, so it is not covered here.

In order to utilize the remaining free space, first we expand the partition and then we expand the filesystem.
As always make sure that the partitions are not mounted and that you are altering the proper ones. The following
examples assume that the SD card is the device `/dev/sdb` and the root partition the `/dev/sdb2`.

Start `fdisk`. The `p` command lists the partitions in the device `/dev/sdb`.

{% highlight text %}
# fdisk -u=sectors /dev/sdb
   Command: p

   Disk /dev/sdb: 7822 MB, 7822376960 bytes
   241 heads, 62 sectors/track, 1022 cylinders, total 15278080 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk identifier: 0x000c21e5

   Device Boot      Start         End      Blocks   Id  System
   /dev/sdb1   *        2048      194559       96256    c  W95 FAT32 (LBA)
   /dev/sdb2          194560     3862527     1833984   83  Linux
{% endhighlight %}

Then, we delete the the second partition (the roor):

{% highlight text %}
Command (m for help): d 
Partition number (1-4): 2
Partition 2 is deleted
{% endhighlight %}

and re-create it. We create the new partition as primary and we set the last sector at the end of the SD card.
The default settings should be fine here.

{% highlight text %}
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2): 2
First sector (186368-15278079, default 186368): 
Using default value 186368
Last sector, +sectors or +size{K,M,G} (186368-15278079, default 15278079): 
Using default value 15278079
Partition 2 of type Linux and of size 7.2 GiB is set
{% endhighlight %}

The partition is already set to Linux (83) so there is no need to alter it. Finally, save the changes:

{% highlight text %}
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
{% endhighlight %}

In order to expand the filesystem, first we run the checkdisk program to detect and fix inconsistencies and
then use the resize program.

{% highlight text %}
# e2fsck -f /dev/sdb2
# resize2fs /dev/sdb2
{% endhighlight %}

### IP discovery and remote login

The Arch Linux ARM image has the ssh deamon enabled by default. In order to login we must first
determine which IP the rpi has acquired from the local network. This assumes that there is a
working DHCP server on the LAN. One way to get the IP address is to check the DHCP logs or the router
logs if DHCP is running from a router. As an alternative, we can run a ping scan on the LAN using
nmap. Assuming that the lan subnet is 192.168.0.0 with mask 255.255.255.0 run:

{% highlight text %}
# nmap -PE -sn -n 192.168.0.0/24
{% endhighlight %}

After finding the IP login using the username and password `root`.

### Post-install configuration

After you login, you should change the root password using `passwd`. Also, it is a good idea to re-generate
the rsa and dsa keys.

{% highlight text %}
# ssh-keygen -f /etc/ssh/ssh_host_dsa_key -N "" -t dsa
# ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N "" -t rsa
{% endhighlight %}

Answer "yes" on prompt to overwrite. To run a full system upgrade run:

{% highlight text %}
# pacman -Syu
{% endhighlight %}

### Memory allocation

Depending on the usage you are planning to do with the rpi it might be a good idea to adjust the RAM
split between the CPU and the GPU. Edit the file `/boot/config.txt` file and change the value of the
variable `gpu_mem_256` or `gpu_mem_521` depending on the rpi model you have.
[This post](http://raspberrypi.stackexchange.com/a/1675) shows the valid memory values.

### Sources and further reading

* [memory allocation](http://raspberrypi.stackexchange.com/a/1675)
* [resize physical parition](http://litwol.com/content/fdisk-resizegrow-physical-partition-without-losing-data-linodecom)
* [Arch Linux ARM for RPI](http://archlinuxarm.org/platforms/armv6/raspberry-pi)
* [Arch Linux post-installation notes](https://wiki.archlinux.org/index.php/Beginners%27_Guide#Post-installation)
