---
layout: post
title: "Adding Unit Tests to an Existing iOS Project"
date: 2011-10-30 16:30
categories: [ios, testing]
---

I recently came across a post from the guys at [Two Bit Labs](http://twobitlabs.com) on [Adding Unit Tests to an existing iOS project](http://twobitlabs.com/2011/06/adding-ocunit-to-an-existing-ios-project-with-xcode-4/) with Xcode 4.

I always include unit tests by default in any project that is more than a demo, but up until recently I have always used [GHUnit](https://github.com/gabriel/gh-unit). [Mark Makdad](http://twitter.com/#!/makdad) wrote a piece earlier in the year [comparing GHUnit and OCUnit](http://longweekendmobile.com/2011/04/15/unit-testing-in-xcode-4-use-ocunit-and-sentest-instead-of-ghunit). That post combined with [Kiwi](https://github.com/allending/Kiwi)'s Rspec-style wrappers around OCUnit has made me consider using Apple's built in unit testing again.

I worked through the post from Two Bit Labs to add a unit test target to an existing project, which all worked fine. Xcode automatically added a new scheme for my new unit test target that only had the *Test* build action hooked up. This isn't exactly what I wanted. Ideally, I want to run my unit tests without changing the current scheme, and just press `Cmd + U` or select `Product > Test` from the menu.

Adding the test action to an existing scheme
--------------------------------------------

When I select my target, called *Development*, that option is greyed out. I have a number of targets set up for this project, so perhaps for simple projects Xcode sorts it out for you... but here is what I had to do in order to get my unit tests hooked up to my main Development target's scheme.

Select your main target's scheme in the little scheme selector in the Xcode toolbar, and choose *Edit Scheme...* from the drop down menu. When you select the *Test* build action on the left, you'll notice no tests appear in the list. Tap the *+* button at the bottom edge of the table, which should show a list of available test bundles (I had a whole lot to choose from because my project has the unfortunate privilege of still including Three20). Choose the unit test bundle you just created you just created and press *Add*.

![Adding tests to the scheme's test action]({{ site.baseurl }}/assets/images/posts/adding-unit-tests/adding-test-action-to-scheme.png)

You should see the test bundle appear in the table, as shown in the image above. Tap on the *Build* action for this scheme, and you will notice that there are now two targets lists: our application and our unit tests. The tests are only linked up to the *Test* build action by default, which should look similar to the image below.

![Checking the two targets in the build action]({{ site.baseurl }}/assets/images/posts/adding-unit-tests/test-build-action.png)

Running your unit tests
-----------------------

Now that your unit tests are hooked up to your test build action, select your main target's scheme. The menu item `Product > Test` should now be active. Select *Test*, and Xcode will build your application and run your tests. When creating the test bundle, Apple's default test case includes a failure, so if the tests run, you will see that failure.

Just to double check you've got all your target dependencies set up correctly, try a full clean (hold *Option* while selecting `Product > Clean`) and then press *Cmd + U*. This should trigger a full build and run the unit tests and show your test failure again.

If you see an error like `ld: file not found:` `<path to app>/Debug-iphonesimulator/Development.app/Development`, it's likely your test target does not automatically build your main application under test, as explained in the [original article](http://twobitlabs.com/2011/06/adding-ocunit-to-an-existing-ios-project-with-xcode-4/): select your project in the navigator, select your unit test target, open the *Build Phases* tab and add your app target as a dependency. Ending up with something like the image shown.

![Setting up dependencies for your test target]({{ site.baseurl }}/assets/images/posts/adding-unit-tests/test-target-dependency.png)

Now... do I use this approach and start convert all my unit tests slowly from GHUnit to OCUnit? I think I might leave existing projects on GHUnit, and just use OCUnit for new tests.
