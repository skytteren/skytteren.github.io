--
layout: post
title: Installing Docker on OS X with Homebrew
categories: 
- code
tags: 
- docker
--

The best way to install [Docker.io](http://docker.io) on Mac OS X is by using Homebrew. 
At least at the time of writing this.

Before you start make sure you got a decent internet connection. 
There is going to be a lot of downloading. 
(Believe it or not but it beats maven).

h2. boot2docker

A Mac isn't linux it doesn't have the option of runing [LXC (linux containers)](https://linuxcontainers.org). 
The easiest way around that is using boot2docker.
It can be installed with brew.

``brew install boot2docker``

Then initialise and start docker:

``
  boot2docker init
  boot2docker start
``

This will have some output. 
The important part is: `export DOCKER_HOST=tcp://:XXXX`. 
Where XXXX is a port.
Export this. 
Eg. run:

``export DOCKER_HOST=tcp://:2375``

h2. docker

In order to install docker run.

``brew install docker``

To test the docker installation:

``docker run base /bin/echo 'Hello World'``

It should output "Hello World".


