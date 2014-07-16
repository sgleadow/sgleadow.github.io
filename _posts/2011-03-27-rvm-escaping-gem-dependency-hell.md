---
layout: post
title: "RVM: Escaping Gem Dependency Hell"
date: 2011-03-27 16:02
categories: [ruby, osx, rvm, frank]
---

This is my second post on my move to rvm. The first being about [cleaning out my existing gems](blog/2011/03/27/cleaning-out-ruby-gems-under-os-x/) for a fresh start.

Bundler seemed like it was the ideal solution to having specific gems for specific projects, but it seems to install in your system gems by default causing version and dependency conflicts with other projects I'm working on. I know I can use `bundle --deployment` but then I have to `bundle exec` everything and I don't have enough Ruby-fu to use the bundler-managed gems within Rakefile etc.

I'm looking to [rvm](http://rvm.beginrescueend.com/) to solve the problem. The steps I followed are below:

Installing RVM
--------------

There are some pretty good [installation instructions](http://rvm.beginrescueend.com/rvm/install/) on the rvm site. The actual installation is a just single command.

```bash
$ bash < <(curl -B http://rvm.beginrescueend.com/install/rvm)
```

You'll see a bunch of output, and towards the bottom, a confirmation message saying "Installation of RVM to /Users/sgleadow/.rvm/ is complete.", but that's not all. You need to set up your shell to know about rvm. Put the following line into your .bash_profile or wherever your chosen shell wants it.

```bash
[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
```

After that, source your profile and check rvm is present and accessible.

```bash
$ source ~/.bash_profile
$ which rvm/Users/sgleadow/.rvm/bin/rvm
/Users/sgleadow/.rvm/bin/rvm
$ rvm --version
rvm 1.5.2 by Wayne E. Seguin (wayneeseguin@gmail.com) [http://rvm.beginrescueend.com/]
$ type rvm | head -1
rvm is a function
```

Installing some Rubies
----------------------

Now that rvm is installed, let's install some rubies to use with it. First, check which ruby versions are available.

```bash
$ rvm list known
...
[ruby-]1.8.7[-p334]
...
```

I'll install the main two versions that I'm using at the moment. Each install will take a few minutes, so this is a good time to make a cup of tea. I would usually say coffee but I haven't bought any fresh coffee beans in a while and the stuff I has is so old that the coffee tastes like sawdust.

```bash
$ rvm install 1.8.7
$ rvm install 1.9.2
```

I'm still using 1.8.7 for some things at the moment, so I'll set that up as the default, and double check that the active ruby binary is now the one I just installed using rvm.

```bash
$ rvm --default use 1.8.7
$ ruby -v
ruby 1.8.7 (2011-02-18 patchlevel 334) [i686-darwin10.7.0]
$ which ruby
/Users/sgleadow/.rvm/rubies/ruby-1.8.7-p334/bin/ruby
$ which gem
/Users/sgleadow/.rvm/rubies/ruby-1.8.7-p334/bin/gem
$ rvm list default
Default Ruby (for new shells)
ruby-1.8.7-p334 [ x86_64 ]
```

Done. We have rvm up and running

Setting up your gemsets
-----------------------

While rvm solves the issue of having multiple versions of ruby installed on the one machine, and switching between them, it also allows you to set up multiple gemsets, which is the feature that started me on this journey in the first place. I'd like to set up a separate gemset for each separate project to try and keep them separate and escape from dependency hell.

I should have a pretty clean slate to start on for my gems. I'm not 100% on whether rvm will use my old system gems or not, but I've cleaned them all out anyway, so it shouldn't be a problem.

```bash
$ gem list
*** LOCAL GEMS ***
rake (0.8.7)
$ rvm list gemsets
rvm gemsets
=> ruby-1.8.7-head [ x86_64 ]
   ruby-1.8.7-head@global [ x86_64 ]
   ruby-1.9.2-head [ x86_64 ]
   ruby-1.9.2-head@global [ x86_64 ]
```

rvm will set up a default gemset for you, but I'd like to have each project automatically set its own ruby version and gemset. We can use the .rvmrc file to achieve this. I'm setting a project up that uses the [Frank](https://github.com/moredip/Frank) iOS testing tool, below is a sample of the .rvmrc file I have *inside* that my project root directory. When I enter that directory in the shell, rvm picks up the rc file and will switch to that version of ruby (the part before the @) and also switch to a specific gemset (the part after the @). The `--create` means that if the gemset does not already exist, create one. The first time you enter the directory, you will need to specifically allow rvm to do this. After that, it's automatic.

```bash
$ cat my_project/.rvmrc
rvm --create use ruby-1.8.7-p334@frank
```

Once the .rvmrc file is created, leave and return to the directory to kick it off, authorise it and you should see something like `Using /Users/sgleadow/.rvm/gems/ruby-1.8.7-p334 with gemset frank` printed to the console. To double check we are, in fact, using the newly created gemset, use `rvm list gemsets` or `rvm gemset list` and check the little hashrocket is pointing to the new gemset. Do another `gem list` to double check you have a clean slate.

Most of my projects use bundler, so first I'll need to install that, then I can use bundler to manage all the gems I need installed. Before using bundler, I wanted to remove the existing gems that had been used by bundler.

```bash
$ rm .bundle/config
$ rm -r vendor/bundle
$ gem install bundler
$ bundle
```

Bundler should install all the gems listed in the Gemfile into the gemset you have just defined for this project. Once bundle is complete, check the gems were installed in the gemset and not just in vendor with `gem list`. If you're project is all set up with a Rakefile to run your tests, try that to double check everything has gone smoothly.
