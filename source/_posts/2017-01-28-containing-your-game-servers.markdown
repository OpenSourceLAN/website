---
layout: post
title: "Containing your game servers - Docker and game servers"
date: 2017-01-28 21:28:52 +1100
comments: true
categories: 
---

**Updated 2017-02-19 to include build process optimisations**

Over recent history, we've been updating our [gameservers-docker](https://github.com/OpenSourceLAN/gameservers-docker) repository with some useful dockerfiles. Many are still a work in progress, but all are usable. Let's take a walk through some examples to see the utility of this approach.


### tl;dr

Get a game server with no effort.

```
# Do these once to set up the image
git clone https://github.com/OpenSourceLAN/gameservers-docker.git .

# Download the prop hunt map packs (this is not auto-downloaded because
# of its size and you may want a different map pack)
wget --output-document tf2-prophunt/PHMapEssentialsBZ2.7z\
 https://github.com/powerlord/sourcemod-prophunt/releases/download/maps/PHMapEssentialsBZ2.7z

./build.sh tf2-prophunt

# Do this every time you need to start an additional
./start_server tf2-prophunt
```

Almost all resources are automatically downloaded (eg, steamcmd, sourcemod) and so
no action is needed from you to download them. There are a handful of files that
we chose not to auto-download for various reasons. The build scripts will not
let you build without them, so you can't accidentally miss them.

### Building the image

First we need to build our images. Let's build a TF2 prophunt image first.

The images are built in a heirachy. `base` is the common ancestor of all of our images, so that comes first.
If you try and build a descendant image and a dependency is not already built, the script will build it for you.

In this tutorial, we will do it the long and hard way so that you understand how it is structured.

All of the folders in the repository have a `build.sh` file. `cd` in to `base` and run `./build.sh`. Alternatively,
you can use `build.sh` in the root of the repository and pass an image name to the script as an argument.


```
sirsquidness@squid ~/projects/gameservers-docker/base $ ./build.sh 
Sending build context to Docker daemon 3.072 kB
Step 1/5 : FROM ubuntu:16.04
 ---> bd3d4369aebc
Step 2/5 : RUN sed -i 's/archive.ubuntu.com/au.archive.ubuntu.com/' /etc/apt/sources.list
 ---> Using cache
 ---> a4a4d42f1211
Step 3/5 : RUN apt-get update  && apt-get dist-upgrade -y &&	apt-get install -y unzip p7zip curl wget lib32gcc1 iproute2 vim-tiny bzip2 jq && 	apt-get clean
 ---> Using cache
 ---> 0325eb88ec60
Step 4/5 : RUN echo "Australia/Melbourne" > /etc/timezone
 ---> Using cache
 ---> deb81775de67
Step 5/5 : RUN ln -fs /usr/share/zoneinfo/Australia/Melbourne /etc/localtime
 ---> Using cache
 ---> 5efc7e451ece
Successfully built 5efc7e451ece
```
Note that the timezone is automatically detected based on your system's local timezone. 

The base image provides a set of common utilities, such as curl, jq, and common dependencies for games.

The next image in the chain for tf2-prophunt is `steamcmd`, which is a tool used to install Steam game servers.

```
sirsquidness@squid ~/projects/gameservers-docker/base $ cd ../steamcmd
sirsquidness@squid ~/projects/gameservers-docker/steamcmd $ ./build.sh 
Sending build context to Docker daemon 3.072 kB
Step 1/9 : FROM base
 ---> 28c1c378be5d
Step 2/9 : RUN useradd -m steam
 ---> Running in a0d250c083fa
 ---> 23f68f0a2e62
[...snip....]
[----] Verifying installation...
Steam Console Client (c) Valve Corporation
-- type 'quit' to exit --
Loading Steam API...Created shared memory when not owner SteamController_Shared_mem
OK.

Connecting anonymously to Steam Public...Logged in OK
Waiting for license info...OK
 ---> b4a6b84eeb5f
Removing intermediate container 49628529285b
Successfully built b4a6b84eeb5f
```

Great! `steamcmd` image is now built. If you looked carefully, you'd have noticed that we downloaded the 
steamcmd installation files automatically. This is a semi-pattern in this repository. Where it makes sense,
the build scripts will download dependencies automatically. But some other dependencies require manual
action as we will see soon.

`tf2-prophunt` depends on the `tf2` image. Build this now. TF2 is a large server - nearly 8GB - so make sure
that you have plenty of disk space free and something to entertain you while it downloads.

```
sirsquidness@squid ~/projects/gameservers-docker/tf2 $ ./build.sh 
Sending build context to Docker daemon 6.144 kB
Step 1/8 : FROM steamcmd
 ---> b4a6b84eeb5f
Step 2/8 : USER steam
 ---> Running in ae23e6b04ec1
 ---> d810f2252646
```

And so on. Some time later, it finishes.

Right now, you can run a vanilla TF2 server - what joy!

Documentation is still a work in progress, but you can [check the start-tf2.sh](https://github.com/OpenSourceLAN/gameservers-docker/blob/master/tf2/start-tf2.sh)
script to see what environment variables are available. Let's set a
hostname and an RCON password on a vanilla TF2 server!

`docker run --net=host -it --rm -e "SV_HOSTNAME=A super cool TF2 server" -e "RCON_PASSWORD=OSLisSuperCool" -e LAN=1 tf2`

Boom, LAN server. But it's just a plain TF2 server. Time for a PropHunt server.

[The tf2-prophunt download script](https://github.com/OpenSourceLAN/gameservers-docker/blob/master/tf2-prophunt/download.sh)
downloads a few dependencies, such a metamod and sourcemod. However, it does not
download the PH map packs. [PHMapEssentialsBZ2.7z](https://github.com/powerlord/sourcemod-prophunt/releases/download/maps/PHMapEssentialsBZ2.7z)
 is few hundred MB and there a few options available, so download the PH map pack that is right for you.
Place the 7z file in the tf2-prophunt directory, and run the build script

```
sirsquidness@squid ~/projects/gameservers-docker/tf2 $ cd ../tf2-prophunt/
sirsquidness@squid ~/projects/gameservers-docker/tf2-prophunt $ wget https://github.com/powerlord/sourcemod-prophunt/releases/download/maps/PHMapEssentialsBZ2.7z
[...snip...]
sirsquidness@squid ~/projects/gameservers-docker/tf2-prophunt $ ./build.sh
[...snip...]
```

And now you have a TF2 Prophunt server installed with only a few _super simple commands_. 


### Running a server

As we're building docker images, you're free to use the docker command line
as you see fit. During the image building, there was an example of doing just that
to prove that the server worked.

But to make it easier, there is a simple helper script included in the root of the repository.

The simplest use of the script is to invoke it by itself. 

```
sirsquidness@squid ~/projects/gameservers-docker $ ./start_server.sh 
Please select from one of the following images, or ctrl+c to exit:
7daystodie
armagetron
base
csgo
csgo-comp
csgo-ebot
csgo-gg
csgo-warmod
factorio
hl2dm
minecraft
openttd
quake3
steamcmd
tf2
tf2-prophunt
trackmania
unreal4
Image: 
```

Type in the name of the server you want, press enter, and away it goes!

I typed `tf2-prophunt` and pressed enter, and this is what happened:

```
Image: tf2-prophunt
docker run --tty --interactive --net host --restart=unless-stopped --name tf2-prophunt_1 --detach tf2-prophunt
54da57b4c67c8e669667f43ecca88a0317c8284e91003e2d4c62ac82f0546b52
```

```
sirsquidness@squid ~/projects/gameservers-docker $ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
cd105fff9efc        tf2-prophunt        "./start-tf2-proph..."   28 seconds ago      Up 27 seconds                           tf2-prophunt_1

```

We can also pass the name of the server directly to the script.

```
sirsquidness@squid ~/projects/gameservers-docker $ ./start_server.sh tf2-prophunt
docker run --tty --interactive --net host --restart=unless-stopped --name tf2-prophunt_1 --detach tf2-prophunt
4bac06dd1a6a2437ecbdcd4d8e91c449c1360965927d7a7b5daf86053a2d072f
```

Notice the convention for `--name` - `tf2-prophunt_1`. Every server started will get a unique number.
You can use this name to get a console to the server.

```
sirsquidness@squid ~/projects/gameservers-docker $ docker attach  tf2-prophunt_1 
status
hostname: TF2 Prophunt Server
version : 3694595/24 3694595 secure
```
Press ctrl+p, then ctrl+q to detach again. Don't press ctrl+c here! If you press ctrl+c,
it will send the kill signal to the game server and it will shut down.

Of course, that's not always convenient. So let's add an RCON password.

### Configuring the servers

Most of the images are set up to accept environmental variables to dictate behaviour.
For example, a common one in most of the images is `RCON_PASSWORD`.

The `start_server.sh` script doesn't currently have a simple way of password long env vars
through, so let's use docker directly.

```
docker run --net=host -it --rm -e "SV_HOSTNAME=A super cool TF2 server" -e "RCON_PASSWORD=OSLisSuperCool" -e LAN=1 tf2
```

Cool!

### Data persistence

Some images support exporting important data out of the running container so that
you can delete and/or upgrade your containers without losing data. Currently only
factorio and 7daystodie support this.

The `start_server.sh` script will automatically detect games that support this and
create a folder of the same name as the container in the `data` folder. 

Please double check that it is working before relying on it - file and folder
permissions are finnicky for exported mounts in docker. The easiest (and worst!)
way to make it work is making the folder globally readable and writable - 
`chmod -R 777 data/`. 


