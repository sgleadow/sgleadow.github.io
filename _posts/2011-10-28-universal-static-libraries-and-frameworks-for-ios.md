---
layout: post
title: "Universal Static Libraries and Frameworks for iOS"
date: 2011-10-28 17:48
categories: [ios, frank]
---

You only have to look at github to see the explosion of new open source iOS frameworks. Incorporating third party libraries into your app is still a pain. I spent today fighting with static libraries and frameworks for the [DCIntrospect](https://github.com/domesticcatsoftware/DCIntrospect) library, and I think I won, so that's got to be work writing about.

The changes I made can be see on [my fork of DCIntrospect](https://github.com/sgleadow/DCIntrospect).

Static libraries and frameworks
-------------------------------

Until recently, my default use case was to actually drag in the source files from a third party library and compile it into my application. For libraries that I have forked and plan to modify or extend, this would still be my preferred option. Some third party libraries are stable and reliable, and I do not wish to change the source code at all. For these tools, I would prefer to just drag in a static library or framework and get on with my own app.

I've tried my hand at static libraries before, and it definitely wasn't plain sailing. A twitter conversation between two local iOS devs, [Pat](http://twitter.com/#!/patr) and [Oliver](http://twitter.com/#!/orj) prompted me to have another go at getting these libraries working. Pat's own tool, [DCIntrospect](https://github.com/domesticcatsoftware/DCIntrospect) is a perfect example. It's a great tool for introspecting and testing strange UI layout behaviour in your app, and it's a library that *just works* (famous last words, I bet you find a bug in it now)... but credit to Pat, I've used it on a few apps, and never touched the source code. DCIntrospect seems like a good case for bundling everything up in a library.

Static Library
--------------

I created an  Xcode 4 workspace for DCIntrospect, which initially just contained the existing *DCIntrospectDemo*. The demo included the source code directly, so I removed the references to those files and created a new static library project that included the existing DCIntrospect source files. That basically compiled straight away, but to make it useful, I had to tweak a few build settings.

The target name is DCIntrospectStaticLib, but I wanted the output to be libDCIntrospect.a, which can be done by changing the **Product Name** build setting. I also wanted to group the header files with the compiled binary, so I set the **Public Headers Folder Path** to be `$(PRODUCT_NAME)` so that it would match whatever I entered for the product name.

To link to this static library, you simply have to drag it into your project. Xcode sorts out adjusting the **Library Search Paths** automatically when you add the library. However, if you want to import headers and make calls to the library, you will need to manually set the **Header Search Paths**. For DCIntrospectDemo, the search path is `"$(SRCROOT)/../Products/lib"` and then use `#import <DCIntrospect/DCIntrospect.h>`. That's easy enough to do, but it is one more barrier to entry for people using the library. A static framework could make it a single step.

Static Framework
----------------

I followed the steps described in [Diney Bomfim's](http://twitter.com/#!/dineybomfim) updated post on building a [universal framework for iPhone](http://db-in.com/blog/2011/07/universal-framework-iphone-ios-2-0/), and it all went pretty smoothly. He describes the different build settings you have to adjust to get it all to work.

Diney's instructions also include a sample shell script for compiling a universal static library using the `lipo` tool. I adjusted the script slightly, so that it would build both a universal static library as well as the static framework. I haven't had much success running DCIntrospect on the device, so I'm not sure if that part is working or not.

The joy of using the static framework in this way is that installing DCIntrospect is now a single drag and drop of a framework into your source code. To get it working at this point, you would still need to add `#import <DCIntrospect/DCIntrospect.h>` into your app delegate and after you have loaded the window, call `[[DCIntrospect sharedIntrospector] start]`.

Automatic Loading of DCIntrospect
---------------------------------

My next task is to get DCIntrospect to automatically load if it is linked to your target. I had a play around with this a few months ago with the [Frank iOS testing library](https://github.com/moredip/Frank), with some success but never pursued the final solution, but [Pete Hodgson](http://twitter.com/#!/beingagile) obviously got sick of waiting for me and did it himself.

The solution is pretty neat. You basically hook into when a class in the library is being loaded, and set yourself up to listen for the `UIApplicationDidBecomeActiveNotification` notification, which gets called after the app delegate `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` gets called. When that happens, we start DCIntrospect. I've wrapped the call to load DCIntrospect in a check that we are running in the simulator, so it will never be called on the app we submit to Apple. Initially I tried to do a check using `#ifdef DEBUG`, but that wont work with the static precompiled library approach, since those flags are only valid when the library is initially compiled, not at run time.

```objc
#import "DCIntrospect.h"

@interface DCIntrospectLoader : NSObject
@end

@implementation DCIntrospectLoader

// This is called after application:didFinishLaunchingWithOptions:
// so the statusBarOrientation should be reported correctly
+ (void)applicationDidBecomeActive:(NSNotification *)notification
{
    NSString *simulatorRoot = [[[NSProcessInfo processInfo] environment] objectForKey:@"IPHONE_SIMULATOR_ROOT"];
    if (simulatorRoot)
    {
        NSLog(@"Running in simulator, loading DCIntrospect");
        [[DCIntrospect sharedIntrospector] start];
    }
}

+ (void)load
{
    NSLog(@"Injecting DCIntrospect loader");

    [[NSNotificationCenter defaultCenter] addObserver:[self class]
                                             selector:@selector(applicationDidBecomeActive:)
                                                 name:@"UIApplicationDidBecomeActiveNotification"
                                               object:nil];
}

@end
```

With the auto loading class above now part of DCIntrospect, installation is literally a single drag and drop of the framework. It will automatically load as needed, as long as we are running in the simulator. I usually have a separate target for App Store submission without any test libraries linked into it just in case, since they bloat the binary and probably call some private API magic at some point.

DCIntrospect Demo Project
-------------------------

To use either the static library or the framework in the two demo project targets, the only change I had to make was to add the *-ObjC* setting to **Other Linker Flags**. For the static library, you still need to set a header path but only if you want to make direct calls to DCIntrospect, since it now loads automatically.

Since I had both the DCIntrospect and DCIntrospectDemo projects in the same workspace, XCode 4 had the smarts to find the *implicit dependency*, so when I edit the library source code, it's automatically rebuilt. However, I set up the project to use the *BundleProducts* target to create the universal distributable binaries, which did not get picked up as a dependency.

You can create *explicit dependencies* between targets in different projects by first creating a project dependency. That is, dragging the dependent project into the demo project. Once that is done, when editing the **Target Dependencies** in the demo target's *Build Phases*, you will be able to explicitly select the dependency. In this case, it is the *BundleProducts* target.

Unless you plan to edit the source code and recompile the binary often, the chances are you wont need to set up the project dependencies like this.

Issues with static libraries and frameworks
-------------------------------------------

DCIntrospect is a fairly simple framework to generate. Larger libraries are likely to come across two issues, neither of which affect DCIntrospect.

* categories from static libraries don't seem to load properly
* resources is static libraries or frameworks need to be linked into the target using them separately

For the categories issue, I know if two solutions. First, you can add *-all_load* to the **Other Linker Flags** section of the target using the static library. We've already added the *-ObjC* flag in there, so it's not really adding much of a burden. The other option is to use Karl Stenerud's *MAKE_CATEGORIES_LOADABLE* as [used in Frank](https://github.com/moredip/Frank/blob/master/LoadableCategory.h).

For the resources issue, you can generate a resource bundle along with your static library or framework. When linking the library to your target, you can also drag in the resources bundle as a dependency and it should work. Karl has a neat way of making this even easier in his [iOS Univeral Framework](https://github.com/kstenerud/iOS-Universal-Framework) project in github. He creates an *embedded framework*, which is essentially a single container for both the static framework and the resources bundle. This extra container makes it easy to drag in a single *.embeddedframework* file into your project and have everything link up correctly.


Final Thoughts
--------------

This is probably the first time I have fought with static libraries and frameworks and felt like I came out on top, and DCIntrospect seems like a good case for using it. That said, there are lots of libraries I use where I prefer being able to see the source code, and potentially edit the implementation. For those, git submodules and adding the source directly is still my preferred option. The single drag-and-drop solution here does make using a new library pretty easy though.
