---
layout: post
title: "Getting started with iOS development"
date: 2012-07-02 14:07
categories: [ios]
---

Given the recent boom in demand for iOS apps, and the fact that many of us own these shiny new devices, I'm often asked where to start in learning iOS development. To save myself writing the same email over and over, here is the approach I would currently take to learning iOS, and some of the resources I would use. I hope it's useful.

Starting out
------------

Some people like to just start to code an app for themselves and learn along the way. I'm sure it's possible for iOS as well, but I initially found Objective C to be a fairly strange language, and Xcode to be a very different IDE, that I wanted a guide to get me to a certain level of understanding. I wanted to know Objective C well enough to be able to write my own independent code, and understand solutions on Stack Overflow.

I'm still one of those people who likes to read a book cover to cover when learning a completely new platform. The two introductory books and an online course I would recommend are below. Make sure you get the latest editions, as the iOS platform evolves quickly and older books get out of date.

Read the [Big Nerd Ranch iOS Guide](http://www.bignerdranch.com/book/ios_programming_the_big_nerd_ranch_guide_rd_edition_) by Joe Conway and Aaron Hillegass. This is the most recommended introductory iOS book I've come across. If you have the time and resources to attend the Big Nerd Ranch training courses, that'd be a great way to start out, but given they are pretty expensive, this book is a great way to start. Get the 3rd Edition, which has been updated for all the iOS 5 goodness.

Read the [iOS SDK](http://pragprog.com/book/adios/ios-sdk-development) book from Bill Dudney and Chris Adamson for the Pragmatic Programmers. This book goes into a bit more depth about specific SDKs and tools (eg. Storyboards, iCloud), but is still suitable for people just starting out.

Watch a decent amount of Paul Hegarty's [Standford iOS Course](http://itunes.apple.com/us/itunes-u/ipad-iphone-application-development/id473757255) on iTunesU. Again, try to get the latest version which at the time of me writing this is the 2011 Fall semester. Watching videos can be a little slow. I would either watch them in double time or airplay it onto your TV and load up Xcode and code along with the lectures.

Build something
---------------

Those introductory materials should give you enough knowledge of Objective C, Xcode and iOS to really get going building your own app. Try to build something you care about, that solves a problem you have. Devote enough time to see it through.

The things I struggled with when I started to code my own stuff was about how to break down my iOS app code properly (MVC, subclasses, categories and subprojects), and how to get out of code signing and provisioning hell. There will be a bunch of common problems you run into early on. Don't code in isolation.

The biggest help to me was attending the local [Melbourne Cocoaheads](http://www.melbournecocoaheads.com/) meetup, and they're pretty widespread around the world these days. The group also has monthly hacknights in a bar. Once you get uses to a table of geeks with Apple hardware on the bench in the pub, it's a great place to work and ask questions from folks who have been doing this a while.

Understanding iOS
-----------------

Building your own app will get you to the point where you're quite comfortable coding apps independently. That's great, but also dangerous. Apple is a very controlling company, and their SDKs and development tools reflect that. You want to get a good grasp of the Objective C language, it's design and runtime environment, so you can effectively code for their platforms. You also want to read Apple's developer material to get an idea of how they're intending apps to be written and what direction the platform is heading in. There are two resources I'd recommend for this.

The [Cocoa Design Patterns](http://www.amazon.com/Cocoa-Design-Patterns-Erik-Buck/dp/0321535022) book is great. It's more about Mac OS X than iOS, but they're fundamentally very similar platforms. While the book is a few years old now (published in 2009), the fundamental design patterns of both the Objective C language and the runtime have not changed significantly in that time. Obviously, there have been plenty of superficial changes to the SDKs but this book will help you understand why things are the way they are.

Watch plenty of Apple's WWDC videos. You'll need an Apple developer account to get to these. The most recent [WWDC 2012](https://developer.apple.com/videos/wwdc/2012/) have recently become available, but there are plenty of talks from [WWDC 2011](https://developer.apple.com/videos/wwdc/2011/) and [WWDC 2010](https://developer.apple.com/videos/wwdc/2010/) that are still relevant as well.

Updated: [Mark Aufflick](http://mark.aufflick.com/) suggested that I also mention [Mike Ash's blog](http://mikeash.com/pyblog/), and I totally agree. Mike's posts offer great insight into the bowels of the Objective C language and runtime. It's definitely worth following and reading his material.

And if you really want to become an expect, work on an iOS app with some people who have already been on this journey and learn from them. I certainly learnt the most from pair programming with developers who are better than I will ever be.

Summary
-------

For all you slackers who don't like reading my carefully crafted prose, here's a list of my recommended resources, in the order I would tackle them:

* [Big Nerd Ranch iOS Guide](http://www.bignerdranch.com/book/ios_programming_the_big_nerd_ranch_guide_rd_edition_)
* [iOS SDK](http://pragprog.com/book/adios/ios-sdk-development)
* [Standford iOS Course](http://itunes.apple.com/us/itunes-u/ipad-iphone-application-development/id473757255)
* [Melbourne Cocoaheads](http://www.melbournecocoaheads.com/)
* [Cocoa Design Patterns](http://www.amazon.com/Cocoa-Design-Patterns-Erik-Buck/dp/0321535022)
* [WWDC Videos](https://developer.apple.com/videos/wwdc/2012/)

I haven't even mentioned the whole topic of mastering automated testing for iOS, which is something I'm passionate about. There's enough information on iOS testing for an entirely separate post, so stay tuned for that.

Done. Happy coding.
