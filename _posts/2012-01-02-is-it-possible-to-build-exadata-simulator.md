---
title: "Is It Possible to Build An Exadata Simulator?"
layout: post
---

The idea of creating an Exadata simulator arose at Oracle Day 2011 Istanbul. One of my friends was trying to fix a virtual machine in a hurry (right before his presentation), and he said his "fake Exadata" crashed. He was joking, but I wondered if it's possible to build an Exadata Simulator using virtual Box (or any other virtualization software). I googled it and found nothing useful, so I started to work on it.

The important point is, simulating Exadata does not mean simulating all features of the Exadata Database Machine. The key features of the Exadata Database Machine are Infiniband connections and Exadata Storage Servers (the offloading capabilities and Flash Cache). It's obvious that we do not need to simulate InfiniBand. All we need is to simulate "Exadata Storage Servers".

Smart scanning, storage indexes, hybrid columnar compression, I/O resource manager, smart flash cache are all handled by the Exadata Storage Server "Software". Although it's called Oracle Exadata Database "Machine", its heart is the Exadata Storage Server "Software". You may say that all hardware needs software, but the Exadata software is not an embedded one. It's just an application running on Oracle Linux 5.x 64bit.

I found a way to download the Exadata Storage Server Software. It took about three days to install it in a virtual box and one week to solve the problem of mapping physical "disks" to cell disks. By the way, I haven't modified any executable file or script. So it was a clean installation. Then I created an ASM disk group using my "fake" Exadata Storage and started to test the features of the Exadata Storage Server.

![Exadata Simulator](/assets/exadatasim.png)

<!--more-->

I've noticed an interesting thing about Exadata Storage Server Software. It detects that it’s running on a virtual/non-Exadata machine and reports it as "fake hardware" 🙂 So it seems that Oracle intentionally did not put a hardware lock on Exadata Storage Server Software.

````
CellCLI> list cell detail
         name:                   exadatacell1
         bbuTempThreshold:       45
         bbuChargeThreshold:     800
         bmcType:                absent
         cellVersion:            OSS_11.2.0.3.0_LINUX.X64_110929
         cpuCount:               2
         diagHistoryDays:        7
         fanCount:               1/1
         fanStatus:              normal
         id:                     6fae98c4-48a6-4004-aa2c-1466639ab336
         interconnectCount:      1
         interconnect1:          eth0
         iormBoost:              0.0
         ipaddress1:             192.168.100.75/24
         kernelVersion:          2.6.18-194.el5
         makeModel:              Fake hardware
         metricHistoryDays:      7
         offloadEfficiency:      2,141.5
         powerCount:             1/1
         powerStatus:            normal
         releaseVersion:         11.2.2.4.0
         releaseTrackingBug:     12708838
         status:                 online
         temperatureReading:     0.0
         temperatureStatus:      normal
         upTime:                 0 days, 0:04
         cellsrvStatus:          running
         msStatus:               running
         rsStatus:               running
````

Well, I should mention a problem with my Exadata Simulator. I did tens of tests with different data but couldn’t make “storage indexes” work. I found some articles, created the same sample data, and did the same queries, but my Exadata Simulator never used storage indexes.

When I was trying to map physical disks to cell disks, one of my friends said he read Arup Nanda's article about simulating Exadata. I know that article. It was about performing the Oracle Exadata simulation in SQL Performance Analyzer, but I wanted to recheck it. I googled “Exadata Simulator” (it seems "Exadata Simulator" are the right keywords to search), and I found an article that says Oracle has an Exadata Simulator, which is used in Exadata Training. It explains why Exadata Storage Server Software runs on non-Exadata hardware. Oracle probably wanted to run it as a simulator. I also learned that it’s possible to download Exadata installation ZIP files from Oracle Software Delivery Cloud. By the way, my friends (probably access to this simulator) never told me about it. It seems they are good at keeping secrets 🙂

In short, it’s possible to download Exadata Software from Oracle Software Delivery Cloud (edelivery.oracle.com – ZIP files of the latest versions are not password protected). You also get “Official” Exadata Documents when downloading the ZIP files. With some work, you can partially install Exadata Storage Server to any Oracle Linux 5.x 64bit running on Virtual Box (or any other virtualization software) and build a virtual Exadata Storage Server. I don’t know the legal restrictions, so I can’t write a step-by-step guide, but I’m sure that people will start to build their own Exadata Simulators because now they know it’s possible.

**Additional Info (1):** I said I had problems with storage indexes, but after some tweaks on parameters, I made it work. Exadata storage cell software seems to decide not to create storage indexes if the storage node has low memory.

**Additional Info (2):** Although software downloaded from e-delivery.oracle.com is free for education, Oracle says running Exadata storage server software on non-Exadata hardware is illegal. So I deleted my simulator. Please do not ask me to share it or the instructions to build it.
