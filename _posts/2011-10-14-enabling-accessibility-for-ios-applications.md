---
layout: post
title: "Enabling Accessibility for iOS applications"
date: 2011-10-14 16:24
categories: [ios, testing]
---

I'm looking to write up a few posts about using accessibility for testing native iOS applications. Here is the first one, dealing with enabling accessibility for your apps in the simulator and on the device.

Why use accessibility
---------------------

Firstly, the iPhone and iPad are setting a new standard for usability by impaired users. That's a great thing, and I think we should be making a bigger effort to support these features. Apple provides assistants like VoiceOver that use the building UIAccessibility framework. If you'd like to find out more about accessibility for the sake of accessibility, [Matt Gemmell](http://mattgemmell.com/2010/12/19/accessibility-for-iphone-and-ipad-apps/) has a great post on the topic

A great side effect of making an app accessible to assistive devices is that the app also becomes easier to test in an automated fashion. This post is not about how and why to use accessibility to test your app (although that is a valid topic that I will write a separate post on), it a quick guide to turning accessibility on. Generally, the properties of the UIAccessibilty framework are only available to third party testing tools like [Frank](http://www.testingwithfrank.com/) and [KIF](https://github.com/square/KIF) if you have accessibility enabled for the application, either in the simulator or on the device.

Enabling accessibility in the Simulator
---------------------------------------

During development, we need to enable accessibility for both OS X and the iOS Simulator. Under Mac OS X, open up the System Preferences and open the *Universal Access* pane at the top right. In the pane that opens, check the box for *Enable Access for Assistive Devices*. The OS will persist this setting from now on.

{% img /images/posts/accessibility/osx-prefs.png Universal Access in the OS X Preferences %}
{% img /images/posts/accessibility/enable-osx-prefs.png Universal Access in the OS X Preferences %}

Load the iOS Simulator and open the *Settings* application. Enable accessibility using the switch under *General > Accessibility*. If you see a little coloured box appear, you have successfully enabled accessibility in the simulator. The setting is stored in an underlying plist file under that iOS version of the simulator, so you will need to enable this setting for for both iOS 4 and 5 separately, but that will effect all of the device types (iPhone, iPhone Retina and iPad) for that OS version. The iOS Simulator will keep accessibility enabled as long as you don't clear out its settings. If you're anything like me, you have *Reset Content and Settings* mapped to a keyboard shortcut, you'll regularly need to navigate in and re-enable accessibility.

{% img right /images/posts/accessibility/enable-on-simulator.png Enable accessibility in the iOS Simulator %}

The small coloured box that appears is called the accessibility inspector. It shows a small summary of what is available from the UIAccessibility framework for iOS. There are two main types of information shown in the inspector: notifications and properties. Notifications are fired when the UI changes. To be honest, I've not played around with firing accessibility notifications much at all. Notifications could be a potential solution for tests that need to 'wait' until a screen transition is finished before continuing rather than busy-waiting or just a plain old sleep (you know it's not a real test suite unless there are a couple of *sleep* calls in there!).

The properties show aspects of the selected UI element. With the accessibility inspector expanded, tap around in some of Apple's built in applications to see the UIAccessibility properties. With the accessibility inspector expanded like this, the first touch even brings up the accessibility - which is great if that's the only way to use the app, but can get in the way if you're not used to it. If you collapse the inspector using the little cross-button, touch interaction returns to normal.  In the image above I've collapsed the accessibility inspector and dragged it to the side, since the properties are available to our tests anyway, so it's easiest to get it as far off the screen as possible.

Enabling accessibility on the device
------------------------------------

Usually, to get access to the accessibility framework on an actual iOS device, you need to enable *VoiceOver*. If you do your testing with Apple's sanctioned UIAutomation Instrument, it seems to be able to hook in automatically without you having to specifically enable those features. Although with iOS 5, I've found that to not always be the case.

VoiceOver is pretty easy to enable in the Settings app under *General > Accessibility > VoiceOver*. Once this is enabled, the device acts in a similar way to when you have the accessibility inspector visible in the simulator. A pleasant computerised voice now describes your every gesture, and more importantly, activates the accessibility framework for all applications, including the one you want to test. Your first tap will select a UI element and read the available information about it. Double-tapping actually executes the action for a button. People using VoiceOver as a means to navigate the OS are likely to drag their finger on the screen to get a better idea of where items are relative to each other, so single-finger scrolling is also disabled. You can scroll by dragging with three fingers.

{% img left /images/posts/accessibility/itunes-configure-accessibility.png Configure Universal Access in iTunes %}
{% img left /images/posts/accessibility/itunes-voiceover-on.png Enable VoiceOver in iTunes %}

Navigate some well known apps on the phone, and see how Apple's own applications integrate with VoiceOver. To actually design an accessible application, you'll be wanting to spend a whole lot of time using VoiceOver yourself to get an idea of what information is useful and necessary. At some stage, you will pick up a test device with VoiceOver on, so it’s good to know at least how to get into the settings and turn accessibility off in order to operate manually.

If you are only enabling VoiceOver for testing purposes, the changed gestures and audio instructions can seem to get in the way of you just using the device. If you regularly switch accessibility on and off, using iTunes is going to be much more convenient. Before the latest version of iTunes, enabling accessibility this way required the device to be plugged in via USB. I was pleasantly surprised to discover that this VoiceOver can now be toggle on and off over wifi.

Enabling accessibility programmatically
---------------------------------------

*Note: these steps apply to the iOS Simulator only. If someone knows how to enable accessibility programmatically on the device, I would love to know*

When you are running tests in the simulator, it's likely that you use the *Reset Content and Settings...* menu item frequently. If you forget to reenable accessibility after this, tests will fail because UI information will not be available to the tests. We need a programmatic way to turn accessibility on. I'm not sure how to do this on the device, but in the simulator, this is just a matter of setting a flag in one of the underlying plist files. It's possible to call plist editors from the command line to do this, but then the plist file is separate for each iOS version supported by the simulator. I find it's easier to call from the Objective C code at run time, since it it possible to obtain the root for that version of the simulator.

```objc
#import <dlfcn.h>

+ (void)load
{
    NSAutoreleasePool *autoReleasePool = [[NSAutoreleasePool alloc] init];

    NSString *simulatorRoot = [[[NSProcessInfo processInfo] environment] objectForKey:@"IPHONE_SIMULATOR_ROOT"];
    if (simulatorRoot) {
        void *appSupportLibrary = dlopen([[simulatorRoot stringByAppendingPathComponent:@"/System/Library/PrivateFrameworks/AppSupport.framework/AppSupport"] fileSystemRepresentation], RTLD_LAZY);
        CFStringRef (*copySharedResourcesPreferencesDomainForDomain)(CFStringRef domain) = dlsym(appSupportLibrary, "CPCopySharedResourcesPreferencesDomainForDomain");

        if (copySharedResourcesPreferencesDomainForDomain) {
            CFStringRef accessibilityDomain = copySharedResourcesPreferencesDomainForDomain(CFSTR("com.apple.Accessibility"));

            if (accessibilityDomain) {
                CFPreferencesSetValue(CFSTR("ApplicationAccessibilityEnabled"), kCFBooleanTrue, accessibilityDomain, kCFPreferencesAnyUser, kCFPreferencesAnyHost);
                CFRelease(accessibilityDomain);
            }
        }
    }

    [autoReleasePool drain];
}
```

I first tried to use this code in `init` for a test framework class, and it didn't seem to work. Moving the code to be called from `load` solved the problem. Since this code actually changes a *plist* file on the file system, perhaps it needs to be executed early before the UI loads so that the rest of the system acts as if accessibility is enabled. Not only does this approach ensure accessibility is always enabled in the simulator, it doesn't bring up the inspector over the top of the UI.

Special thanks to [Cedric Luthi](http://twitter.com/#!/0xced), who originally wrote the code to enable accessibility programmatically for DCIntrospect [in this commit](https://github.com/0xced/DCIntrospect/commit/49b76a6630cc29444aac30f14fd0fc17e22b37cf), and to an awesome colleague of mine, Sadat Rahman (@sadatrahman), for bringing that code to my attention.
