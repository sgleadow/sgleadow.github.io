---
layout: post
title: "Running OCUnit &amp; Kiwi Tests on the Command Line"
date: 2012-02-09 07:54
categories: [ios, testing, ci]
---

A common question I get asked is: how to I run my OCUnit tests from the command line? I've answered this a number of times, so here is a quick brain dump so I can just point to this post each time. I say OCUnit, but since [Kiwi](https://github.com/allending/Kiwi) is just a wrapper above OCUnit, these instructions should also work for Kiwi tests.

Xcodebuild
----------

OCUnit tests are run from a shell script during the build for that target, which is usually called something like MyAppTests. If you look at the *Build Phases* for that target, you'll see a little RunScript phase that actually runs the tests:

```
# Run the unit tests in this test bundle.
"${SYSTEM_DEVELOPER_DIR}/Tools/RunUnitTests"
```

We can then use the `xcodebuild` command line utility to run this same build from the command line. On the command line (or from a Makefile if you are so inclined), try building and running the tests:

```
xcodebuild -sdk iphonesimulator -configuration Debug -scheme MyAppTests build
```

This may or may not work, depending on whether or not you have *Logic* or *Application* tests. What?


Logic &amp; Application Unit Tests
----------------------------------

I'm not sure why Apple bothers to differentiate these tests. The basic difference is that Application tests need to be run inside of a UIKit environment, while Logic tests don't. I'd prefer if the build system just detected whether it needed UIKit and decided for you. There's a longer explanation in  [Apple's unit testing guide](https://developer.apple.com/library/ios/#documentation/DeveloperTools/Conceptual/UnitTesting/00-About_Unit_Testing/about.html#//apple_ref/doc/uid/TP40002143). I often want to write unit tests for my UIViewController classes and any other UI logic, so I always tick the Application checkbox when adding unit tests to an Xcode project. If you're making a static library with no UI then you might want to select Logic.

Anyway, if your project has *Logic* tests, the above `xcodebuild` command should run your unit tests fine and you will see the test output in the Terminal window. However, if they are Application tests, the command silently succeeds without actually running your tests. If you look near the bottom of the output, you'll see this in the output:

```
/Developer/Platforms/iPhoneSimulator.platform/Developer/Tools/RunPlatformUnitTests:95:
warning: Skipping tests; the iPhoneSimulator platform does not currently support application-hosted tests (TEST_HOST set).
```

So the tests didn't even run, but the command succeeds, the build stays green and everyone's happy... but the tests aren't running.


Running From The Command Line
-----------------------------

I usually point people to [a post from the guys at Long Weekend](http://longweekendmobile.com/2011/04/17/xcode4-running-application-tests-from-the-command-line-in-ios/) for how to get these Application tests running from the command line. Here's quick summary:

Open up `/Developer/Platforms/iPhoneSimulator.platform/Developer/Tools/RunPlatformUnitTests` in your favourite editor and check out the code around line 95. You'll see the culprit line:

```
Warning ${LINENO} "Skipping tests; the iPhoneSimulator platform does not currently support application-hosted tests (TEST_HOST set)."
```

THe `TEST_HOST` is only set when running the tests through Xcode, not from the command line. If the host isn't set, the tests don't run. We want them to run in this case. We could replace that line with the following:

```
export OTHER_TEST_FLAGS="-RegisterForSystemEvents"
RunTestsForApplication "${TEST_HOST}" "${TEST_BUNDLE_PATH}"
```

This command will actually boot up your app in the iOS simulator and run the tests in that UI environment.

I don't really feel comfortable hacking scripts in my Developer directory, so I'd rather copy this script into my project and run it from there. Also, editing the actual installed tools like this means that you have to do it on each and every machine running the tests. Even once you get it running on your machine, you'll need to log into your build machine and make the same hack there. Try making a copy of the same shell script Apple uses into your project directory:

```
cd your-project
mkdir scripts
cp /Developer/Platforms/iPhoneSimulator.platform/Developer/Tools/RunPlatformUnitTests scripts/
```

Now edit that local copy of your shell script to replace line 95 to actually run the tests in the simulator environment. The MyAppTests target is still pointing to the system developer script. In Xcode, open up the *Build Phases* for your test target and change the *Run Script* to point to your local script:

```
# Run the unit tests in this test bundle.
"${SRCROOT}/../scripts/RunPlatformUnitTests"
```

Run the tests in the IDE to make sure you didn't screw anything up. Now quit the simulator, and run them from the command line with xcodebuild again. You should see the test output printed and a few statements at the bottom saying that the tests succeeded. Your tests do pass, right?

```
/Developer/Tools/RunPlatformUnitTests.include:334: note: Passed tests for architecture 'i386' (GC OFF)
/Developer/Tools/RunPlatformUnitTests.include:345: note: Completed tests for architectures 'i386'
```

I [uploaded a gist of my modified shell script](https://gist.github.com/2410839) that I use that runs the unit tests from the command line.

One simulator to rule them all
------------------------------

The reason I said to close the simulator before running the tests from the command line is that the tests don't seem to like running when the iOS Simulator is already doing something. Try it. Open the simulator, and then run the tests. If your system is anything like mine, you get an error like this:

```
** BUILD FAILED **

The following build commands failed:
	PhaseScriptExecution "Run Script" path-to-the-script-in-derived-data
```

If I'm running them locally, I see this, quit the simulator and try again. That's a bit of a pain, and it certainly wont work on the CI server. To solve that, in my Makefile, I just use a little Applescript to close the simulator before running the tests:

```
test:
	osascript -e 'tell app "iPhone Simulator" to quit'
	xcodebuild -sdk iphonesimulator -configuration Debug -scheme MyAppTests build
```

Now I can use the command `make test` to run the tests from the command line both on my machine and the CI server without having to change the development environment.
