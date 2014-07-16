---
layout: post
title: "Installing ImageMagick on Lion"
date: 2012-03-03 13:49
categories: [ios, testing, osx]
---

I needed to get ImageMagick installed this morning to play with an iOS testing tool called [Zucchini](http://www.zucchiniframework.org/) that I'll write about separately. My new MacBook Air is running OS X Lion (never ran Snow Leopard), and Xcode 4.3. I've found installing native tools like this has been pretty painful on Lion from the default compiler changes (like [installing rvm](/blog/2011/12/10/installing-rvm-on-os-x-lion/)), so I wasn't surprised ImageMagick didn't install correctly first time. First I tried to `brew install imagemagick`. While it succeeded in installing the library, right at the end there was the following error:

```
ln: ImageMagick: Permission denied
Error: The linking step did not complete successfully
The formula built, but is not symlinked into /usr/local
You can try again using `brew link imagemagick'
```

There were similar errors talking about little-cms, libtiff and jpeg as well. I thought, I'll ignore that, maybe it'll work anyway and went ahead and tried to use imagemagick. When I tried to use the tool though, I saw errors like `dyld: Library not loaded: /usr/local/lib/liblcms.1.0.19.dylib`.

A bit of googling took me to a thread on the [homebrew issues on github](https://github.com/mxcl/homebrew/issues/6891). I didn't actually need to uninstall/reinstall like some of the comments there suggest, I just needed to link the libraries giving the errors into /usr/local like this:

```
sudo brew link little-cms
sudo brew link imagemagick
sudo brew link libtiff
sudo brew link jpeg
```

Maybe it's a better solution to allow myself write access to /usr/local, and then the brew install would have linked automatically. I've just ended up with such a mess of libraries in /usr/local on old machines, I'm trying to keep my new little machine as clean as possible.
