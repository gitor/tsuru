.. Copyright 2013 tsuru authors. All rights reserved.
   Use of this source code is governed by a BSD-style
   license that can be found in the LICENSE file.

+++++++++++++++++++++++++++++++++++++++++
Build your own PaaS with tsuru and Docker
+++++++++++++++++++++++++++++++++++++++++

This document describes how to create a private PaaS service using tsuru and docker.

This document assumes that tsuru is being installed on a Ubuntu 13.04 64-bit machine. If
you want to use Ubuntu LTS vesion see `docker documentation <http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-precise-12-04-lts-64-bit>`_ on how to install it.
You can use equivalent packages for beanstalkd, git, MongoDB and other tsuru
dependencies. Please make sure you satisfy minimal version requirements.

You can use the scripts bellow to quick install tsuru with docker:

.. highlight:: bash

::

    $ curl -O https://raw.github.com/globocom/tsuru/master/misc/docker-setup.bash; bash docker-setup.bash

Or follow this steps:

DNS server
----------
You can integrate any DNS server with tsuru. Here: `<http://docs.tsuru.io/en/latest/misc/dns-forwarders.html>`_ you can find a example of how to install a DNS server integrated with tsuru

docker
------


Install docker:

.. highlight:: bash

::

    $ sudo echo 'deb http://ppa.launchpad.net/dotcloud/lxc-docker/ubuntu precise main' >> /etc/apt/sources.list
    $ sudo apt-get update -y
    $ sudo apt-get install lxc-docker -y

MongoDB
-------

Tsuru needs MongoDB stable, distributed by 10gen. `It's pretty easy to
get it running on Ubuntu <http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/>`_

Beanstalkd
----------

Tsuru uses `Beanstalkd <http://kr.github.com/beanstalkd/>`_ as a work queue.
Install the latest version, by doing this:

.. highlight:: bash

::

    $ sudo apt-get install -y beanstalkd

Configuring beanstalkd to start, edit the `/etc/default/beanstalkd` and uncomment this line:

::

    START=yes

Then start beanstalkd:

.. highlight:: bash

::

    $ sudo service beanstalkd start

Gandalf
-------

Tsuru uses `Gandalf <https://github.com/globocom/gandalf>`_ to manage git repositories, to get it installed `follow this steps <https://gandalf.readthedocs.org/en/latest/install.html>`_

Installing git
~~~~~~~~~~~~~~

.. highlight:: bash

::

    $ sudo apt-get install git -y

Creating git user
~~~~~~~~~~~~~~~~~

.. highlight:: bash

::

    $ sudo useradd git

Creating directories for repositories and template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's create the directory for bare repositories:

.. highlight:: bash

::

    $ sudo mkdir -p /var/repositories
    $ sudo chown -R git:git /var/repositories

And the directory for template and add the tsuru hooks:

.. highlight:: bash

::

    $ sudo mkdir -p /home/git/bare-template/hooks
    $ curl https://raw.github.com/globocom/tsuru/master/misc/git-hooks/post-receive > /home/git/bare-template/hooks/post-receive
    $ sudo chown -R git:git /home/git/bare-template

Configuring gandalf
~~~~~~~~~~~~~~~~~~~

.. highlight:: bash

::

    sudo bash -c 'echo "bin-path: /usr/bin
    database:
      url: 127.0.0.1:27017
      name: gandalf
    git:
      bare:
        location: /var/repositories
        template: /home/git/bare-template
      daemon:
        export-all: true
    host: localhost
    webserver:
      port: \":8000\"" > /etc/gandalf.conf'

Change the 'host: localhost' to your base domain.

Tsuru api and collector
-----------------------

You can download pre-built binaries of tsuru and collector. There are binaries
available only for Linux 64 bits, so make sure that ``uname -m`` prints
``x86_64``:

.. highlight:: bash

::

    $ uname -m
    x86_64

Then download and install the tsr binary:

.. highlight:: bash

::

    $ curl -sL https://s3.amazonaws.com/tsuru/dist-server/tsr-master.tar.gz | sudo tar -xz -C /usr/bin

These commands will install ``tsr`` command in ``/usr/bin``
(you will need to be a sudoer and provide your password). You may install this
command in your ``PATH``.

Configuring
~~~~~~~~~~~

Before running tsuru, you must configure it. By default, tsuru will look for
the configuration file in the ``/etc/tsuru/tsuru.conf`` path. You can check a
sample configuration file and documentation for each tsuru setting in the
:doc:`"Configuring tsuru" </config>` page.

You can download the sample configuration file from Github:

.. highlight:: bash

::

    $ [sudo] mkdir /etc/tsuru
    $ [sudo] curl -sL https://raw.github.com/globocom/tsuru/master/etc/tsuru-docker.conf -o /etc/tsuru/tsuru.conf

By default, this configuration will use the tsuru image namespace, so if you try to create an application using python platform,
tsuru will search for an image named tsuru/python. You can change this default behavior by changing the docker:repository-namespace config field.

Running
~~~~~~~

Now that you have ``tsr`` properly installed, and you
:doc:`configured tsuru </config>`, you're three steps away from running it.


Start api, collector and docker-ssh-agent

.. highlight:: bash

::

    $ tsr collector &
    $ tsr api &
    $ tsr docker-ssh-agent &

You can see the logs in:

.. highlight:: bash

::

    $ tail -f /var/log/syslog


Creating Docker Images
======================

Now it's time to install the docker images for your neededs platform. You can build your own docker image, or you can use ours own images as following

.. highlight:: bash

::

    # Add an alias for docker to make your life easier (add it to your .bash_profile) 
    $ alias docker='docker -H 127.0.0.1:4243'
    # Build the wanted platform, here we are adding the static platform(webserver)
    $ docker build -t tsuru/static https://raw.github.com/flaviamissi/basebuilder/master/static/Dockerfile
    # Now you can see if your image is ready - you should see the tsuru/static as an repository
    $ docker images
    # If you want all the other platforms, just run the command bellow
    $ for image in nodejs php python ruby; do docker build -t tsuru/$image https://raw.github.com/flaviamissi/basebuilder/master/$image/Dockerfile;done 
    # To see if everything went well - just take a look in the repository column
    $ docker images
    # Now try to create your apps!

Using tsuru
===========

Congratulations! At this point you should have a working tsuru server running
on your machine, follow the :doc:`tsuru client usage guide
</apps/client/usage>` to start build your apps.


Adding Services
===============
Here you will find a complete step-by-step example of how to install a mysql service with tsuru: `http://docs.tsuru.io/en/latest/services/mysql-example.html <http://docs.tsuru.io/en/latest/services/mysql-example.html>`_
