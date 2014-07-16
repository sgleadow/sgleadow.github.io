---
layout: post
title: "Cleaning Out Ruby Gems Under OS X"
date: 2011-03-27 15:55
categories: [ruby, osx, rvm]
---

As a bit of background, I have been getting into Ruby development for the past year, but I have been getting into gem dependency hell recently. Different projects need different gemsets. [Bundler](http://gembundler.com/) helps but I want to use that in combination with [rvm](http://rvm.beginrescueend.com/) to keep things organised.

To get a clean start with rvm, so I wanted to remove all my existing system gems. My first attempt used the little shell script below to loop over all installed gems and remove them. That worked for a lot of gems, but left some behind, possibly all the default gems that come pre-installed with OS X.

``$ for x in `gem list --no-versions` ; do gem uninstall -aIx $x; done``

After some googling, the next step I tried was to physically remove the gem directories that are returned by `gem env paths`. After that, `gem list` said there were no gems installed. There were still some binaries left over that depended on these gems, like `spec`, `rails`, `rake` in `/usr/bin`. These binaries no longer worked as the actual gem was gone, so I removed them. I couldn't find a definitive list of the OS X default gems so I stopped here.

I'm not a shell guru by any means, but below is my attempt at a quick one line shell script to remove all the existing gem directories.

``rm -rf `gem env paths | tr ":" " "` ``
