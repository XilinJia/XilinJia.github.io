---
layout: post
title:  "QuickFix on Amazon AWS EC2 Free Tier Server"
date:   2023-09-20 13:36:30 +0100
tags:   trading AWS
---

Lately, I built a FIX (Financial Information eXchange) client in Python using QuickFix to pair with a fairly customized FIX 4.2 server.  And I needed to run stability tests.  So I went to get a cloud server at Amazon.

With a new account at Amazon AWS, one is eligible for a free tier server good for a year.  But the server's specs of 1 CPU and 1GB RAM appeared nearly useless to me.  Just logging in my own computer without running any application, the Linux system uses about 1.5GB of RAM.

But since I get it anyhow, I would like to try it.  And it turned out very impressive.  Upon logging in, the Ubuntu 22.04 headless server (no GUI) uses only 160MB of RAM, leaving 840MB for the applications.  I updated system packages of Ubuntu, installed many essential tools for building software, and installed some Python packages, and compiled some packages from source (there are 30GB of SSD space).  The single core CPU acted very fast.

Then I came to install QuickFix.  Same as with other Python packages, I did `pip install quickfix`.  Well this turned out a bomber.  Actually it ran over the available RAM.  As I set 3GB of swap space, the system didn't crash, just went using the swap space.  Not sure what to do, I left it run overnight.  Then, the next morning, it's still running there - at least it proved very stable, didn't it!

But I can't let it continue, who knows how many days it may take.  So I would try building QuickFix from source and see if then I could install the binaries.  I then downloaded the source from QuickFix's Github site, did the build: `configure, make, make check, ...`.  It succeeded in not much time.  Then I looked to install the binaries for Python.  It turned out it's only the C++ package and there is no relevant Python setup files.

I googled, and learned that I needed to download the package with `pip download quickfix`.  It came in with the same file name as the C++ package.  Once uncompressed, it showed python related files.  Then I did `pip wheel .` in the uncompressed directory.  Well, it again went over the available RAM.

Well, how about building the wheel on my own computer and upload it to the server?  I got it down in no time.  Once the wheel file was uploaded, I did `pip install quickfix.whl` on the server, and succeeded.  And it appeared I was all set to run my FIX client.

Unfortunately, no.  The installed QuickFix library complained about missing some shared libraries.  I checked, the Ubuntu server already had the libraries, just of different versions.  So, my computer runs Manjaro, which has newer system packages than Ubuntu 22.04, so the built wheel file linked with the versions of libraries on my computer.  So this doesn't work.

OK, one other thing to do: install the same Ubuntu server in a virtual machine on my computer.  I allocated 1 CPU core and 3GB for the virtual machine, then repeated the above steps of building the QuickFix wheel.  Success!  Then I uploaded the wheel and installed it on the Amazon server.

Things run.  My FIX Python client, together with MongoDB, and other related packages run on the server using less than 200MB RAM.  And there is over 400MB of RAM left on the server for other applications.  Quite amazing!

I know you have fix income, Amazon's Free Tier is calling you not to squander it.


## Cheers!  Santé!  Prost!