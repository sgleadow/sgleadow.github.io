---
layout: post
title: "Installing iOS Apps on the Device From the Command Line"
date: 2011-11-05 10:50
categories: [ios, testing, ci]
---

One of the reasons I run most of my tests in the simulator is that it is easy to install and run applications automatically from the command line with tools like [ios-sim](https://github.com/Fingertips/ios-sim). Being able to run your tests from the command line make s it simple to set up continuous integration. I've used an Applescript to drive iTunes back in Xcode 3 days but it just felt wrong.

The most recent functional testing tool I've been playing with is Apple's own *UIAutomation*, which runs within the Instruments app. I'll write more about that separately, but let's just say I don't think it's an ideal testing tool, especially if TDD and CI are important to you. Some details are in my [comparison of a few functional testing tools](/blog/2011/10/30/adding-unit-tests-to-an-existing-ios-project). The `instruments` command line tool does not seem to install the app on the device before running tests, which means you still need to use Xcode to manually install the app. Enter `fruitstrap`.

Fruitstrap
----------

[Fruitstrap](https://github.com/ghughes/fruitstrap) is a command line tool that uses the private MobileDevice API to install an iOS application on a physical device over USB. It's pretty easy to get set up.

```
git clone git://github.com/ghughes/fruitstrap.git
cd fruitstrap
make fruitstrap
```

Fruitstrap comes with a demo applicaiton, which you can compile and install on a device using `make install`. I actually had a few issues getting the demo app to work, but I did get it working for actual sample applications. You now have the `fruitstrap` command compiled and ready to go - if you want to access the command from anywhere you probably want to add it to you path, or sym link it to `/usr/local/bin` or however you like to tinker with your machine.

Building from the command line
------------------------------

I made a little sample application to play around with `fruitstrap` and scripting on my [github](https://github.com/sgleadow), in a project called [fruitstrap-demo](https://github.com/sgleadow/fruitstrap-demo). It's just a simple Single View Application with a couple of labels so you know it's the right app. To get the repository:

`git clone git://github.com/sgleadow/fruitstrap-demo.git`

Just to make sure everything is working, open up the project in Xcode and build and run to your device. If that doesn't work, fruitstrap isn't going to help you much. The `xcodebuild` command allows us to build our iOS apps from the command line fairly easily. The command I used was:

```
xcodebuild -scheme fruitstrap-demo -sdk iphoneos build
```

Remember to use the `iphoneos` so that it builds your app for the device. Note, I originally tried this with the old *target* settings for `xcodebuild`, but it turned out I needed to use schemes for reasons explained below. The app will be built to `build/Debug-iphoneos/fruitstrap-demo.app`.

Try out fruitstrap
------------------

Now we have an application build on the command line, let's make sure fruitstrap works for our app. Make sure to remove your sample app from the device beforehand so you know it's working. Then use the `xcodebuild` command above to compile the app so it's ready to go, and make sure you know the full path to fruitstrap, or you've put fruitstrap on your path.

```
fruitstrap build/Debug-iphoneos/fruitstrap-demo.app
```

You should see a bunch of output and progress information finishing with the magic `[100%] Installed package build/Debug-iphoneos/fruitstrap-demo.app`, and in a few moments, the app appears in *Springboard* on your phone. That's pretty cool - I've been trying to find a solution for installing on the device like that for a long time!

If you have more than one device plugged in (which is usually the case on a mobile continuous integration server), you'll need to also specify the device id.

Scripting the fruitstrap installation
-------------------------------------

My next goal is to write a little shell script that we can integrate into a build phase in Xcode, so that we don't have to hard code the path `build/Debug-iphoneos` into out shell script. I immediately reached for the Build Phases tab of the fruitstrap-demp target to optionally run the fruitstrap install code. However, these shell scripts seem to get called *before* the code signing is run, in which case, installing to the device will fail.

I found out from [this stackoverflow thread](http://stackoverflow.com/questions/1409981/how-to-run-a-script-after-xcode-runs-codesign-on-my-iphone-app) that you can run pre and post scripts for a scheme. This allows us to hook up a shell script to run fruitstrap *after* the code signing.

There is only one scheme in the sample project, so select *Edit Scheme...*, and select the *Build* action from the list on the left hand side. There are no actions at the moment, so press the **+** button and add a *Run Script Action*. Since we need to know where our target has been compiled, make sure that the *provide build settings from* option is set to *fruitstrap-demo*, as show in the following screenshot.

{% img /images/posts/fruitstrap/fruitstrap-scheme-post-script.png Running a script in the post action for a scheme %}

The actual script code is shown below. It only runs if the `FRUITSTRAP_CLI` environment variable is set, since most of the time we don't want Xcode to be using this third party tool to install on the device. We only need it to run when running from the command line as part of our continuous integration build. It seems the scheme scripts do not get run in the same working directory as you run xcodebuild, so our script makes sure to change to SRCROOT before running fruitstrap.

```
# Do nothing unless we are running from the command line
if [ "$FRUITSTRAP_CLI" = "" ]; then
exit 0
fi

echo "******************"
echo "Installing app to device using fruitstrap..."
echo "Workspace location: $SRCROOT"
echo "Install location: $TARGET_BUILD_DIR/$FULL_PRODUCT_NAME"
echo "******************"

cd $SRCROOT
fruitstrap $TARGET_BUILD_DIR/$FULL_PRODUCT_NAME

echo "******************"
```

Check that when you run the xcodebuild example above that it *does not* run fruitstrap, since we don't want it to in that normal operation. Now, try building the scheme with the environment variable set and check that it does in fact build and install to the device.

```
FRUITSTRAP_CLI=1 xcodebuild -scheme fruitstrap-demo -sdk iphoneos build
```

Done. One command to build and install the app on the device.

More about fruitstrap
---------------------

There is more information about fruitstrap on [its github page](https://github.com/ghughes/fruitstrap).

One extra feature fruitstrap has is to be able to launch the application and attach a debugger, by using the `-d` option. I've had mixed success with this feature, it doesn't always work for me. I'm not sure how much use it is to me anyway if the point of this is running in CI.

```
fruitstrap -d build/Debug-iphoneos/fruitstrap-demo.app
```

Summary
-------

Now we can build the application and install it on the device from the command line. From here, the next step is to hook it up to the `instruments` command line interface. Massive thanks to [Richie](https://twitter.com/#!/heardrwt) for letting me know about fruitstrap.

Ideally, I would like to be able to boot the app on the device *without* being hooked into the debugger. I'm not sure if this is possible, I certainly haven't got it working with fruitstrap yet - and the hairy C code isn't making me want to jump in and try just yet. What we have now is enough to get UIAutomation up and running, since `instruments` will boot the app when the tests start. However, I'd prefer to use Frank or KIF in which case I need to find a way to boot onto the device.
