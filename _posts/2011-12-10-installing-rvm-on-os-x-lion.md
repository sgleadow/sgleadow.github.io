---
layout: post
title: "Installing RVM on OS X Lion"
date: 2011-12-10 19:28
categories: [ruby, rvm, osx]
---

I wrote a post a little while ago about transferring my Ruby dev environments to use [rvm](http://beginrescueend.com/) to organise and separate Ruby envinroments and assosiated [gemsets](http://beginrescueend.com/gemsets/). I just got a new MacBook Air which I'm setting up at the moment, so I'm running through the set up process again. Here are my notes.

There are a few rvm haters out there who find it to be over-engineered and trying to do too much. If that's you, there is another, more lightweight tool called [rbenv](https://github.com/sstephenson/rbenv) that you can use and if you really like separate gemsets, someone has written [rbenv-gemset](https://github.com/jamis/rbenv-gemset) to go with it. I have a handful of projects already set up with rvm, so I'm going to keep using it for now.

Installing rvm itself is pretty easy. The [rvm homepage](http://beginrescueend.com/) gives a quick install command:

```
bash < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
```

and then add this to your shell profile:

```
[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
```

That works fine. Now it's time to install some rubies - at the moment I have projects using 1.8.7 and 1.9.2, and I'd like to use 1.9.3 going forward. I used the following command and got an error:

```
rvm install 1.8.7
...
ERROR: There has been an error while running configure. Halting the installation.
```

It turns out this is a common issue in OS X Lion because gcc is just a sym link to LLVM, and the whole installation gets a bit confused (that's the technical term for it). I read on [Matt Polito's blog](http://www.mattpolito.info/post/9383196211/rvm-ruby-install-on-lion-got-you-down) that you can simply point the CC environment variable straight to GCC 4.2 like this:

```
CC=/usr/bin/gcc-4.2 rvm install <ruby version>
```

That didn't work for me. I'm using a brand new machine with Xcode 4.2 installed. It turns out this solution worked for Xcode 4.1 but 4.2 does not install the right gcc to use. One solution I saw was to roll back to Xcode 4.1 and then install Xcode 4.2 again. That seemed like it would take a while, so I used the standalone OS X GCC installer from https://github.com/kennethreitz/osx-gcc-installer. After that, installing different Rubies worked fine. I'm hoping that installing gcc like this doesn't mess up my Xcode 4.2. So far so good, the few Xcode projects I've tried still compile and run fine.

**Edit:** I think next time I will try the approach mentioned by [Tony Arnold](https://twitter.com/tonyarnold) in the comments below, using the `--with-gcc=clang` option to rvm, something like this:

```
rvm install 1.9.3 --reconfigure --debug -C --enable-pthread --with-gcc=clang
```
