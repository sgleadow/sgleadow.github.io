---
layout: post
title: "Which Automated iOS Testing Tool To Use"
date: 2011-10-26 22:38
categories: [ios, testing, frank, uiautomation, kif]
---

I've been playing around with testing frameworks on iOS for over a year now. There are quite a few out there, all with communities building around them, but I think there are currently a few that stand out:

* Frank
* KIF
* UIAutomation

Other tools that might be worth looking at, but I haven't used and wont comment on are: NativeDriver, LessPainful, iCuke and UISpec.

I recently watched a recording of Pete Hodgson (@beingagile), the primary maintainer of Frank in which he gave a quick summary of the other testing tools, and when it might be appropriate to use them. I made a few notes, and added some of my own thoughts. There is a [video of the talk](http://pivotallabs.com/talks/147-frank-cucumber-tests-for-native-ios-apps) Pete gave at [Pivotal Labs](http://pivotallabs.com/).


Frank
-----

Frank has been described as Selenium for native iOS apps. It's a tool written mainly by some fellow Thoughtworkers that uses a combination of [Cucumber](http://cukes.info/) and JSON commands that are sent to a server running inside your native application, and leverages [UISpec](http://code.google.com/p/uispec/) to run the commands. Frank used to require changes to your production code, but that's got a lot easier with the new static library approach.

I've used Frank quite a bit, and I like it. I have a lot more I could say about it, but I'll leave that for another time. You can find out more about Frank at:

* [Frank's new home page](http://www.testingwithfrank.com)
* [The github page for Frank](https://github.com/moredip/Frank)
* [A slightly out of date talk I gave on Frank](/talks.html)

*** The good parts of Frank ***

* Clean CSS-like selector syntax, allowing for fairly tolerant tests
* Active community discussing and extending the library
* Driven by Cucumber (if you're a cuke fan)
* Includes *Symbiote*, a live introspection tool
* Command line and CI come pretty much for free

*** The bad parts of Frank ***

* Difficult, but not impossible, to run on the device
* Separation between tests/native code over HTTP can confuse the cause of failure
* Decoupling test/native code into separate processes makes it a bit slower (Pete makes the good point that most of the slowness in your iOS tests are likely to be waiting for animations and transitions, not the HTTP communication)
* Very little support for gestures (but hopefully that's coming soon)

*** When using Frank might be appropriate ***

* Your team has experience with web application testing tools (Selenium, Cucumber)
* You prefer a BDD style of development, with CI support
* Your app is not driven by complex gestures
* You have a goal of having cross platform tests across Android, iOS and mobile web


KIF
---

KIF is an Objective C based framework written at Square. It's a nice solid implementation from what I've seen, with active development going on. Since it's Objective C, you can call out to anything else in your app as well. It's a brand new test framework, so it's not going to immediately integrate with other tools (like JUnit-style XML output and that kind of thing). I haven't KIF a lot yet, so I can't comment too much but I think it shows a lot of promise. You can find out more about KIF at:

* [The github repo](https://github.com/square/KIF)
* [KIF Google group](https://groups.google.com/group/kif-framework)
* [The initial blog post on KIF from Square](http://corner.squareup.com/2011/07/ios-integration-testing.html)

*** The good parts of KIF ***

* Your tests are in Objective C. Everything in one language, easier for pure iOS devs to pick up
* Active community and good support
* Pretty reasonable support for gestures
* Command line and CI

*** The bad parts of KIF ***

* Your tests are in Objective C. Non-devs will find it hard to read, doesn't make a great executable spec)
* Tricky to integrate with back end stubs because it's all running in-process
* Not stand alone

*** When using KIF might be appropriate ***

* Primarily a developer driven team
* Developers have stronger Objective C skills than Ruby/Cucumber/Javascript
* Don't need business folk to read or write test specs
* Don't want to deal with the whacky regex from Cucumber
* Don't have really complex back end interactions to stub out
* Don't have cross platform requirements


UIAutomation
------------

[UIAutomation](http://developer.apple.com/library/ios/#documentation/DeveloperTools/Reference/UIAutomationRef/Introduction/Introduction.html) is Apple's own solution for automated testing. It runs tests written in Javascript through the Instruments application that comes with the developer tools. It sounds like a no-brainer to use Apple's solution for building iOS apps, but I've found it a real pain to deal with in practice. UIAutomation is pretty good for actually driving the UI, but not great for organising and running a test suite, especially from CI. My impression is that Apple QA staff must use UIAutomation for test scripts with profiling instruments attached and actually sit there and watch, it doesn't seem to be purpose built for fully automated testing.

At this point I would like to link to a lot of useful documentation about UIAutomation, but in practice I have found Apple's documentation to be either minimal or non-existent, and very little online discussion past the "boot up an app and tap a button in a single javascript file".

*** The good parts of UIAutomation ***

* Apple's own tool. Doing things the way Apple wants is generally a good idea (and as Pete mentions, Apple's not going to go bust or quit iOS any time soon, so the chances are it will be supported)
* More closely linked to the device, I primarily run UIAutomation tests on the device, not in the simulator
* Good support for gestures (pinch zooming and swipes) and rotation

*** The bad parts of UIAutomation ***

* Not built with CI in mind. The command line integration is pretty bad
* Can't integrate with other tools very well
* It's Apple's tool, and it's not open source. You can't jump in and fix the bits that are missing
* Runs within Instruments, seems to be aimed at regression testing not TDD and aimed at QAs not devs

*** When using UIAutomation might be appropriate ***

* You have a separation between development and QA on your team
* You prefer regression test suites over a test-first approach
* You don't really care about CI
* You prefer manual QA and you just want to speed that up a bit


Conclusion
----------

So far I've used Frank and UIAutomation on fairly large projects, and I'm very keen to try KIF. Ideally what I would like is the Frank architecture using Cucumber to drive your tests, but using KIF's implementation on the native side which is a lot more solid than UISpec. Frank would give nice clean readable test features, and integration with other tools through cucumber as well as the concise selector syntax. KIF would give gesture support and a much cleaner implementation of those features.
