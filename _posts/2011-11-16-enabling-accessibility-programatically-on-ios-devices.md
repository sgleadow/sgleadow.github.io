---
layout: post
title: "Enabling Accessibility Programatically on iOS Devices"
date: 2011-11-16 08:29
categories: [ios, testing, ci]
---

I wrote a recent post on [enabling accessibility for iOS applications](/blog/2011/10/14/enabling-accessibility-for-ios-applications/), which ended with a snippet of code for automatically enabling accessibility on the iOS simulator. This is essential if you want to have your tests running on a continuous integration server, since the accessibility inspector is off by default.

I hadn't yet found a solution for enabling accessibility on the device. We need accessibility turned on so that we can access the UIAccessibility values that we use in automated functional tests in one of the [common automated testing tools](/blog/2011/10/26/which-automated-ios-testing-tool-to-use/). My solution up until now has been to turn VoiceOver on, which is a real pain. Since I sometimes use screenshot-based regression tests, VoiceOver breaks the tests since it visually highlights the selected item.

[Cedric Luthi](http://twitter.com/#!/0xced) commented on my previous post that it may be possible to modify his code to also enable accessibility on the device. I wasn't sure how that would work with the app sandbox, and whether it was an application or system preference setting. Last night I gave it a try, and amazingly it worked.

I wrote a simple application based on the master-detail iPhone template in Xcode 4, and wrote a quick [KIF](https://github.com/square/KIF) test that checked a label on the master screen, pushed through to the detail screen and checked a label there also. It's a gimmick test, but enough that it will fail if accessibility is *not* enabled because none of the labels will be available to the test.

After following the standard KIF set up instructions, I write the following test:

```objc
- (void)initializeScenarios;
{
    KIFTestScenario *loadScreen = [KIFTestScenario scenarioWithDescription:@"Test app loads up on correct screen"];
    [loadScreen addStep:[KIFTestStep stepToWaitForViewWithAccessibilityLabel:@"Master"]];
    [self addScenario:loadScreen];


    KIFTestScenario *navigateToDetails = [KIFTestScenario scenarioWithDescription:@"Test can navigate"];
    [navigateToDetails addStep:[KIFTestStep stepToTapViewWithAccessibilityLabel:@"Detail"]];
    [navigateToDetails addStep:[KIFTestStep stepToWaitForViewWithAccessibilityLabel:@"Detail view content goes here"]];
    [self addScenario:navigateToDetails];
}
```

That works fine on the simulator, whether you have explicitly enabled accessibility or not now that the maintainers of KIF have merged in [my pull request](https://github.com/square/KIF/pull/78). That code only runs for the simulator, and does nothing if `IPHONE_SIMULATOR_ROOT` is not available in the `[[NSProcessInfo processInfo] environment]`. The KIF test will only run on the device (at least for me) if I turned VoiceOver on, which is a pain and not possible to automate.

I modified the code so that on the device, it does not try to prepend the simulator root when running on the device, and points to `@"/System/Library/PrivateFrameworks/AppSupport.framework/AppSupport"` directly. Amazingly, the KIF test then worked and all accessibility values were available to the tests. The updated code to enable accessibility programmatically on the device or the simulator looks like this:

```objc
+ (void)_enableAccessibilityInSimulator;
{
    NSAutoreleasePool *autoreleasePool = [[NSAutoreleasePool alloc] init];
    NSString *appSupportLocation = @"/System/Library/PrivateFrameworks/AppSupport.framework/AppSupport";

    NSDictionary *environment = [[NSProcessInfo processInfo] environment];
    NSString *simulatorRoot = [environment objectForKey:@"IPHONE_SIMULATOR_ROOT"];
    if (simulatorRoot) {
        appSupportLocation = [simulatorRoot stringByAppendingString:appSupportLocation];
    }

    void *appSupportLibrary = dlopen([appSupportLocation fileSystemRepresentation], RTLD_LAZY);

    CFStringRef (*copySharedResourcesPreferencesDomainForDomain)(CFStringRef domain) = dlsym(appSupportLibrary, "CPCopySharedResourcesPreferencesDomainForDomain");

    if (copySharedResourcesPreferencesDomainForDomain) {
        CFStringRef accessibilityDomain = copySharedResourcesPreferencesDomainForDomain(CFSTR("com.apple.Accessibility"));

        if (accessibilityDomain) {
            CFPreferencesSetValue(CFSTR("ApplicationAccessibilityEnabled"), kCFBooleanTrue, accessibilityDomain, kCFPreferencesAnyUser, kCFPreferencesAnyHost);
            CFRelease(accessibilityDomain);
        }
    }

    [autoreleasePool drain];
}
```

I'm a little bit unsure of the security of this, and whether you actually want your tests messing with your phones system settings on the device - but I usually have separate test devices to my personal phone, and the KIF code is never part of the actual production app that you submit to Apple, so I'm comfortable with it for now. I'm hoping this combined with my [recent work with fruitstrap](/blog/2011/11/05/installing-ios-apps-on-the-device-from-the-command-line) could get us all the way to functional tests running on a physical device in CI.

You can find this code in [my fork of KIF](https://github.com/sgleadow/KIF), and I'm hoping after a bit of testing it can be merged into the main KIF repo, so I sent them [this pull request](https://github.com/square/KIF/pull/93).
