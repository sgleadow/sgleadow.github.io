---
layout: post
title: "Tolerant JSON parsing for iOS"
date: 2012-05-18 09:38
categories: [ios]
---

Many useful iOS apps need to talk to some kind of backend web service. If you have any control over the back end, it will be talking JSON with your app. Each of these apps has some kind of JSON parsing behaviour. We've had many debates on the merits of parsing the dictionary into our own plain old Objective C objects (POCOs) versus just passing around `NSDictionary` objects. I'll put on my best consultant voice and say "it depends", let's not have that debate again today.

Either way, at some stage you need to be pulling values out of an NSDictionary, and you want to do so safely. Martin Fowler wrote a few years ago about the [tolerant reader](http://martinfowler.com/bliki/TolerantReader.html) approach, in which he says:

> *"My recommendation is to be as tolerant as possible when reading data from a service. If you're consuming an XML file, then only take the elements you need, ignore anything you don't."*


Simple mapping of NSDictionary to properties
-------------------------------------------------

Let's say we have the following JSON to represent a person:

```js
{
  "name" : "stew",
  "role" : "developer",
  "level" : "awesome"
}
```

NSDictionary has some simple KVC methods we can use to pull out the keys we want to map them to the properties on our person class. The properties are `name`, `role`, `level`, all of which are of type `NSString *` for simplicity. Something like this:

```objc
self.name = [dict valueForKey:@"name"];
self.role = [dict valueForKey:@"role"];
self.level = [dict valueForKey:@"level"];
```


Dynamic mapping of NSDictionary to properties
---------------------------------------------

That kind of works, it grabs the right keys... but the developer in you thinks, hey, that seems pretty repeatable. I bet I could make that dynamic so as I add more data to the `NSDictionary`, new properties on my `Person` object are automatically populated. There are frameworks that do that kind of stuff for you, but a simple version could look like:

```objc
[dict enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
  [self setValue:obj forKey:key];
}];
```

Awesome. Less code is better, right?


Take only the elements you need
-------------------------------

The problem with dynamically mapping data from a web service is that it assumes that the JSON data pulled down exactly matches what is expected, and never changes. If you so much as add a new data element to the JSON coming from the server, the app with break with an exception like `raised [<Person 0xe328ce0> setValue:forUndefinedKey:]: this class is not key value coding-compliant for the key new-key.`

Now, even if you control both the client and the server, you want have a bit more flexibility than that. Sure, you could version the API and make it clever enough to serve the right data, but creating new API versions every time you add a new field is a bit of overhead. It's better to *"take only the elements you need"*, then you can add new data as much as you want.

So you ditch the dynamic approach, it saves a few lines of Objective C but creates more problems elsewhere. We're back with the explicit dictionary-to-property mapping, which at least gives us the flexibility to add new data to the API and our iOS app just ignores the extra data.


Be as tolerant as possible
--------------------------

The issue we still have with our *valueForKey* approach is that we are assuming that the API is giving us back the correct data for each key. Objective C is pretty dynamic, so we just grab the value out of the dictionary and set it on the properties, which are NSString types. If, for some reason, the API starts returning a different type for the *level*. Maybe they've decided the level is now the number 11 and not a string, or maybe they've introduced a more complex representation of the level as a dictionary with extra keys, for example:

```js
{
  "name" : "stew",
  "role" : "developer",
  "level" : {"title" : "awesome", "rating" : 5}
}
```

The actual parsing code still works fine. At runtime, Objective C will grab that dictionary for the *level* and shove it into the NSString. Now, everywhere else in our code will make the assumption that `person.level` will give back an NSString, and so it should - but when we start to call methods on this property that are specific to NSString, we're going to get some strange errors. Say you had some code looking at the prefix of the level property:

```objc
NSString *level = person.level;
if([level hasPrefix:@"awesome"])
{
  // do something for awesome people
}
```

Try running that and you'll get an error like `[__NSCFDictionary hasPrefix:]: unrecognized selector sent to instance`. So what we really want is to be tolerant of receiving incorrect data back when you are parsing it. We're already ignoring extra data, now we also want to ignore data that is of the incorrect type.

Safe dictionary extraction
--------------------------

On a number of projects now I've use a little category on NSDictionary that does this type checking for me, and also allows for feeding in a default value if the data is missing. In practice I don't usually feed in a default value, I like the properties to end up being *nil* if the data doesn't come back in the expected format. In the instance where I want to display certain values in the UI to show data is missing, that's UI logic and doesn't really belong in the data parsing in my opinion.

A short and sweet implementation of this safe NSDictionary category could be something like:

```objc
@implementation NSDictionary (SafeAccess)

- (id)valueForKey:(NSString *)key
         ifKindOf:(Class)class
     defaultValue:(id)defaultValue
{
    id obj = [self objectForKey:key];
    return [obj isKindOfClass:class] ? obj : defaultValue;
}

@end
```

Our parsing code might then end up looking more like this:

```objc
self.name = [dict valueForKey:@"name" ifKindOf:[NSString class] defaultValue:nil];
self.role = [dict valueForKey:@"role" ifKindOf:[NSString class] defaultValue:nil];
self.level = [dict valueForKey:@"level" ifKindOf:[NSString class] defaultValue:nil];
```

That's it. Pretty simple. Credit should be given to [Kevin O'Neill](http://twitter.com/#!/kevinoneill) who originally introduced this method, and has a more complete implementation in his [Useful Bits github project](https://github.com/kevinoneill/Useful-Bits).
