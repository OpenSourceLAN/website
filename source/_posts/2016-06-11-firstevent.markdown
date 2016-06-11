---
layout: post
title: "Projects up and our first major event"
date: 2016-06-11 12:00
comments: true
categories: 
---

After a long time going "one day", I've finally released some software under the Open Source LAN name. a
It's all on GitHub under the [OpenSourceLAN](https://github.com/OpenSourceLAN/)
organisation. 

[*steam-discover*](https://github.com/OpenSourceLAN/steam-discover) is a 
tool to report data about how many people are playing which steam games 
on your LAN. Includes a simple live updating graph which you could 
include on your website, and connectors to send the data to arbitrary 
places like Redis, ELK, syslog or similar. 

[*srcds-log-receiver*](https://github.com/OpenSourceLAN/srcds-log-receiver) 
is a simple tool that records all of the SRCDS logs sent to it. No need
to worry about copying all of the logs from your game servers - they're
already in one spot. 

[*origin-docker*](https://github.com/OpenSourceLAN/origin-docker) is 
a terribly named repository of a single docker container which contains
Nginx set up to cache all of the major game distributions for your LAN.

[*Vectorama*](http://vectorama.info) becomes the first LAN which I'm
not involved with to use OpenSourceLAN software. Check out 
[this thread on reddit](https://www.reddit.com/r/lanparty/comments/4nb8rs/vectorama_2016_trying_out_opensourcelans/)
to see how they used SteamDiscover and ELK to create a super cool
dashboard of Steam activity at their event.

