---
layout: post
lang: en
title: Running an Akka Cluster on Docker with Fig.sh
date: 2014-09-05
categories:
 - code
tags:
 - akka
 - cluster
 - docker
 - scala
 - fig.sh
---
[Akka](http://akka.io) got this amazing technology called [clustering](http://doc.akka.io/docs/akka/snapshot/scala/cluster-usage.html).
It would be nice to simulate a real world with [Docker](http://docker.io).
To orchistrate multiple docker instances [fig.sh](http://fig.sh) works well.
The example can be found in on github - [Akka test/fig.sh](https://github.com/skytteren/akka_cluster_test/tree/figsh).

The different nodes:

* seed node - Used for lookup. Two would be nice, but it is not very important for the example.
* client 1 - A client for the cluster
* client 2 - Could be different than client 1 but this is just for show.
* backend 1 - A simple backend to do the work for the frontend clients.
* backend 2 - Just like backend 1 but with persistence (because it is another cool technology from the Akka community).

The important parts are are:

fig.sh
------
```
seed:
  build: seed/target/docker
  ports:
   - "2551:2551"
  hostname: seed
backend1:
  build: backend1/target/docker
  expose:
   - "10101"
  links:
   - seed:seed
  hostname: backend1
backend2:
  build: backend2/target/docker
  expose:
   - "10101"
  links:
   - seed:seed
  hostname: backend2
client1:
  build: client1/target/docker
  expose:
   - "10101"
  links:
   - seed:seed
  hostname: client1
client2:
  build: client2/target/docker
  expose:
   - "10101"
  links:
   - seed:seed
  hostname: client2
```

System IP configuration
-------------------------
To fix problems with hostname and IP addresses added this to the system configuration (scala):

```
val address = InetAddress.getByName(System.getenv.get("HOSTNAME"))
val hostAddress = address.getHostAddress
println("\nHostAddress: " + hostAddress)

val config = ConfigFactory.parseString("akka.remote.netty.tcp.hostname=" + hostAddress).
    withFallback(ConfigFactory.load())

val system = ActorSystem("ClusterSystem", config)
```

Running the cluster
-------------------
To run this (as it says in the README file):

```
install docker.io
install fig.sh

install sbt

run

sbt docker:stage
fig build
fig up
```

The drawback here is ``sbt docker:stage``.
Ideally there should be a ``DockerFile`` in every project instead of generating it.
It would also be nice to have a super quick turn around for each of these projects.
However that is another problem and maybe another blog post.
