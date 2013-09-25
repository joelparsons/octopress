---
layout: post
title: "The Queue Observer Pattern"
date: 2013-09-09 00:28
comments: false
categories: [queues, threads, NSNotification, NSNotificationCenter]

---

I've been using the "new" block based `NSNotificationCenter` addObserver methods for a long time now but I've come across some projects that use the old target action style notifiction listening so I thought I would post about why I believe this newer method is better. 
<!-- MORE -->

To recap the old way of subscribing to `NSNotifications` from an `NSNotificationCenter` was like this:

{% codeblock lang:obj-c %}
NotificationCenter defaultCenter] addObserver:self
                                     selector:@selector(someSelector:)
                                         name:NSSomeNotificationName
                                       object:nil];
{% endcodeblock %}

This method is not without its problems in the current versions of Xcode. For example if one was to use the Refactor Rename tool it doesn't automatically rename the selector in the `@selector()` block. Currently Xcode only suggests the changes. 

![Refactor Rename not picking up selector automatically]({{ root_url }} /images/refactor.png")

Secondly you will appear to have a dead method when using the Xcode Callers list. The Callers list shows all the methods across the project that call that particular selector. When set as the selector of a notification it wont show in the callers list. While it may be obvious you intend the method to fire as a result of a notification you would still have to search for the notification listening code to find out if it existed and the `:(NSNotification *)` parameter is optional.

![The callers menu]({{ root_url }} /images/callers_list.png")

Thirdly this method is more crash prone. If the observer is deallocated without you calling `[[NSNotificationCenter defaultCenter] removeObserver:self];` your app will crash. Admittedly in the scope of `NSNotificationCenter` observers this can be a good thing, alerting the programmer to a forgotten call to `removeObserver:` but its not a good bug to have slip through QA and land infront of users when theres an alternative that can fail silently.

## The Modern Way

So the "new" alternative (Available in iOS 4.0 and later) is the following:

{% codeblock lang:obj-c %}
id observer = [[NSNotificationCenter defaultCenter] addObserverForName:NSSomeNotificationName
                                                                object:self
                                                                 queue:[NSOperationQueue mainQueue]
                                                            usingBlock:^(NSNotification *note) {

                                                                }];
{% endcodeblock %}

The first thing you might notice is that you supply a block that is invoked when the notification is fired. You can also specify a queue for this to be executed on. Compared to the old method which will call all observers in turn and wait for their methods to finish before calling the next one queueing these blocks will exit immeadiately, even when queued to run on the main thread. If you know the work in the block will take time you can even farm it off to another queue right in the listener method. 

But what about that wierd observer object. The method returns an object of opaque, unspecified type that you have to keep around. The reason is that your class is no longer the observer here, the returned object is! This is why this new method can fail silently if the end recipient of these notifications, `self`, gets deallocated without calling `removeObserver:`, the new observer object is actually retained by the notification center and will stay around.

### Caveats

There are things you need to know to use this new method. First of all that observer object is your responsibility. You have to keep a reference to it around and eventually remove it from the NSNotificationCenter by calling `[[NSNotificationCenter defaultCenter] removeObserver:observer];`. I like to have a lazily created mutable array that I use to store observers, and then I can unsubscribe from them all in one go when needed using something like this:

{% codeblock lang:obj-c %}
    for (id observer in self.observers) {
        [[NSNotificationCenter defaultCenter] removeObserver:observer];
    }
    self.observers = nil;
{% endcodeblock %}

The second caveat is down to block memory semantics. That block behaves like all others and retains everything in its scope. This means that if you rely on removing observers in the dealloc method, for example, it may never get called. It also means that, if you forget to remove the observer, whatever you reference in that block will be retained, sit there in memory and potentially execute its code in total isolation. This can cause untold trouble and hard to debug problems. The solution is to make sure that all references to self in the block are weak:

{% codeblock lang:obj-c %}
    __weak JPCloudViewController * weakself = self;
    id observer = [[NSNotificationCenter defaultCenter]
                    addObserverForName:UIApplicationDidEnterBackgroundNotification
                    object:nil
                    queue:[NSOperationQueue mainQueue]
                    usingBlock:^(NSNotification *note) {
                        weakself.cloudContainerView.alpha = 0.0;
                    }];
    [self.observers addObject:observer];
{% endcodeblock %}

Now if the notification is fired and your object is released all the references to self will be nil and nothing will happen. No crashes or anything.

## So why is it better?

Finally heres why I think the new syntax is better. Firstly programmer error will not cause a crash for users. Arguably it would be better to know about programmer error but I think an a memory leak is a better experience than a crash for the end user. If you wanted you could even put an `NSAssert` in the block to throw if weakself is nil. Assert macros are off by default for release builds so you can get the best of both worlds. Another benefit is you define the code to execute where you listen for the notificaion. The Xcode refactor tools work properly with the code in the block. The Callers list also shows correctly for functions called from the block. Code is guaranteed to run on the queue you specify, unlike the older style of notificaion where your code would run on the same thread that posted the notification. I also like the way blocks can be offloaded to background threads with ease.

Its good to try new things so give the new Notification syntax a try today!
