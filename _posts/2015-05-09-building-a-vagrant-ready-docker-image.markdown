---
layout: post
title: building a vagrant ready docker image
date: 2015-05-09 20:08:19
categories: [docker, vagrant]
comments: true
---

[Vagrant][vagrant-url] is a tool which creates virtual development environments
using various virtualization platforms. These platforms are called "providers" in
the Vagrant universe.
It supports VirtualBox, VMware, Hyper-V and as of version 1.6 it also offers
built-in support for using [Docker][docker-url] as a provider.

Docker can be very helpful when creating virtual environments because containers
are spawned way faster than virtual machines and additionaly you don't have the
virtual machine overhead.

Vagrant has the notion of [base boxes][vagrant-base-box]. Base boxes are OS images with bare minimum
packages installed. Vagrant can use these to run any supplied provisioner such as
Puppet, Chef, SaltStack etc. to create the desired virtual environment. However,
Vagrant has some requirements in order to be able communicate and run commands
in the virtual machines, such as properly configured ssh and sudo access.

If we have our Puppet/Chef/whatever code already in place and want to switch from,
let's say VirtualBox, to docker, we need a docker image which supports these Vagrant requirements.
We can use a [Dockerfile][dockerfile-ref] to define the Vagrant requirements and then build our
image. We will create a Vagrant-ready docker image starting off the [debian base image][docker-debian].

First, we need to create a file named `Dockerfile`. A Dockerfile is a text file which describes all
the instructions/commands that docker needs to run in order to build a docker image.
For starters we add the following lines:

{% highlight docker %}
FROM debian:7.8
MAINTAINER Author Name "email@example.com"
{% endhighlight %}

The first line defines which debian base image we are going to use to build ours.
We will be using the image tagged as `7.8`. The maintainer line is informative
showing the author of the generated images.

Next we are going to create the `vagrant` user and set the password to "vagrant":

{% highlight docker %}
RUN useradd --create-home --shell /bin/bash vagrant
RUN echo vagrant:vagrant | chpasswd
{% endhighlight %}

Also, setting the root password to "vagrant" is considered a good practice if we are going
to distribute our image/Dockerfile, so let's do that too:

{% highlight docker %}
RUN echo root:vagrant | chpasswd
{% endhighlight %}

Next we need to configure proper ssh access for the vagrant user. First we add the vagrant public
key to the authorized_keys file and then apply the approrpiate permissions.
The `ADD` instruction will retrieve the `vagrant.pub` key from github and add it in the `authorized_keys`
file of the vagrant user.

{% highlight docker %}
ADD https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub \
    /home/vagrant/.ssh/authorized_keys
RUN chown -R vagrant:vagrant /home/vagrant/.ssh
RUN chmod 0600 /home/vagrant/.ssh/authorized_keys
RUN chmod 0700 /home/vagrant/.ssh
{% endhighlight %}

Next we are going to install the minimum required packages. This is a good time
to also install puppet if you are going to use it to provision the container.
Feel free to ignore it if you don't need it or install chef or whatever tool you need.
We also run apt-get clean to free some space from the resulting image.

{% highlight docker %}
RUN apt-get update \
    && apt-get install -y openssh-server sudo wget curl puppet \
    && apt-get clean
{% endhighlight %}

The vagrant user should have root access to the system. We are going
to take advantage of the `sudoers.d` folder. Sudo parses files in the /etc/sudoers.d
folder so that we don't have to edit /etc/sudoers directly. The format for the files in the
sudoers.d folder is the same as for /etc/sudoers.

Create a folder next to the docker file named `sudoers.d` and place the file `01_vagrant` inside
with the following contents:

{% highlight text %}
vagrant ALL=(ALL) NOPASSWD: ALL
{% endhighlight %}

Then we use the `ADD` instruction to add it to the system and also ensure it has the proper permissions.

{% highlight docker %}
ADD sudoers.d/01_vagrant /etc/sudoers.d/
RUN chmod 0400 /etc/sudoers.d/01_vagrant
{% endhighlight %}

Finally we need to have the ssh server running in order for Vagrant to ssh into the mahine. First we create
the /var/run/sshd folder because we are going to skip running the ssh server init script and run it
using the `CMD` instruction. Also we expose the port 22 using the `EXPOSE` instruction.


{% highlight docker %}
RUN mkdir /var/run/sshd
CMD ["/usr/sbin/sshd", "-D", "-e"]
EXPOSE 22
{% endhighlight %}


Building the docker image is pretty straightforward, running:

{% highlight text %}
docker build -t vagrant-debian .
{% endhighlight %}

will create the image with name `vagrant-debian`. In order to spawn a docker container from that
image we run the `docker run` command. We can see that the default action of this container
is to run an ssh server.

{% highlight text %}
$ docker run -i -t vagrant-debian 
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
{% endhighlight %}

However the whole point is to run this through Vagrant. First stop the running container using the
container id or the container name:

{% highlight text %}
$ docker ps
CONTAINER ID        IMAGE                   COMMAND                CREATED             STATUS              PORTS               NAMES
95041c73dbb9        vagrant-debian:latest   "/usr/sbin/sshd -D -   2 seconds ago       Up 1 seconds        22/tcp              elegant_lumiere
$ docker stop elegant_lumiere
{% endhighlight %}

Then let's create a `Vagrantfile` with the following contents:

{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  vm_name = 'docker-example'
  config.vm.define vm_name.to_sym do |box|
    box.vm.hostname = vm_name

    box.vm.provider 'docker' do |d|
      d.image = 'vagrant-debian'
      d.has_ssh = true
    end
  end
end
{% endhighlight %}

and run `vagrant up --provider=docker`.

{% highlight text %}
$ vagrant up --provider=docker  
Bringing machine 'docker-example' up with 'docker' provider...
==> docker-example: Creating the container...
    docker-example:   Name: tmp_docker-example_1431294926
    docker-example:  Image: vagrant-debian:latest
    docker-example: Volume: /tmp:/vagrant
    docker-example:   Port: 127.0.0.1:2222:22
    docker-example:  
    docker-example: Container created: dbb4b1e104201ba6
==> docker-example: Starting container...
==> docker-example: Waiting for machine to boot. This may take a few minutes...
    docker-example: SSH address: 172.17.0.25:22
    docker-example: SSH username: vagrant
    docker-example: SSH auth method: private key
    docker-example: 
    docker-example: Vagrant insecure key detected. Vagrant will automatically replace
    docker-example: this with a newly generated keypair for better security.
    docker-example: 
    docker-example: Inserting generated public key within guest...
    docker-example: Removing insecure key from the guest if its present...
    docker-example: Key inserted! Disconnecting and reconnecting using new SSH key...
==> docker-example: Machine booted and ready!
{% endhighlight %}

Now we can ssh in the running container through Vagrant using `vagrant ssh`. We can use this
base image to create other images that work with Vagrant (use the image name in the `FROM`
instruction). Or we can use this image together with Vagrant + puppet, or any other configuration
management tool using the `--provision-with` flag. If we just wanted to run a specific service/application we might be better off
putting the service installation/setup in the Dockerfile in the first place and skip the provisioning part.
However, this setup is super helpful if we already have a working puppet configuration and use Vagrant with
VirtualBox et. al. You can find my Vagrant-ready debian images in my [dockerhub repo][dockerhub-debian-vagrant]
and the Dockerfiles in my [github repo][github-debian-vagrant].

[vagrant-url]: https://www.vagrantup.com/
[docker-url]: https://www.docker.com/
[vagrant-base-box]: https://docs.vagrantup.com/v2/boxes/base.html
[dockerfile-ref]: https://docs.docker.com/reference/builder/
[docker-debian]: https://registry.hub.docker.com/_/debian/
[dockerhub-debian-vagrant]: https://registry.hub.docker.com/u/tlatsas/debian-vagrant/
[github-debian-vagrant]: https://github.com/tlatsas/docker-debian-vagrant/
