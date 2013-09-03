---
layout: post
title: "Background core data with privateQueueContext"
date: 2013-09-02 09:00
comments: false
categories: [iOS, core data, background]
keywords: developer,coredata,core,data,ios,development
description: Why you should love privateQueueContext
---

Working with core data on multiple threads can be challenging, as can working across multiple threads in general, but the privateQueueContext method hasn't been as talked about as some of the other methods. 

## privateQueueContext

The old model with `NSManagedObjectContext` was that you had to create a context on every thread you were going to access core data on. When this context was saved it would write back to the persistent store and notifications would fire that would allow you to update your other contexts on other threads.
<!-- MORE -->
The new model changes all this and makes working with core data on other threads easy. In fact instead of thinking in threads its better to think of everything in Queues ([its based on GCD](https://www.google.co.uk/search?q=grand+central+dispatch). You create a context for working on the main queue with like you normally would but you use the following new method:
{% codeblock lang:obj-c %}
NSManagedObjectContext * mainQueueContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
{% endcodeblock%}
There are many methods you can use to share this context among your view controllers like pass the baton, using the app delegate or a shared instance of your store object in a [model-view-controller-store](http://programmers.stackexchange.com/questions/184396/mvcs-model-view-controller-store) style project. Whats best is really a personal choice.

But the exciting part is how you work with core data in the background. Create a context with the private queue concurrency type like so:
{% codeblock lang:obj-c %}
NSManagedObjectContext * priveteQueueContext = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSPrivateQueueConcurrencyType];
{% endcodeblock%}
I like to keep this context private within a store object, since I mostly use the background thread for data processing and most of that happens within the store or its scope. One can then submit work to be done asyncronously in the background like so:
{% codeblock lang:obj-c %}
[self.privateQueueContext performBlock:^{
        Magazine * magazine = (Magazine *) [self.privateQueueContext objectWithID:magazineObjectID];
        magazine.headerImage = image;
        NSError * error;
        [self.privateQueueContext save:&error];
        if (error) {
            //handle error
        }
    }];
{% endcodeblock %}

## Queueing things up

Queues can give you some quite complex behaviour. Say you want to add a bunch of objects to Core Data and then save at the end. You send off several worker blocks, one to add each item, and then add block at the end that saves all the objects. No need to keep track of state or figure out when you are on the last object.

You can import a bunch of objects into core data, and then add a block at the end which works with all the objects you just added. This can be really helpful for importing objects and then adding relationships between them once they are all imported.

## Syncing across

The only issue is syncing your changes made in the background to your foreground context. Listening to the notificaiton is fairly simple:
{% codeblock lang:obj-c %}
id observer = [[NSNotificationCenter defaultCenter]
                   addObserverForName:NSManagedObjectContextDidSaveNotification
                   object:self.privateQueueContext
                   queue:[NSOperationQueue mainQueue]
                   usingBlock:^(NSNotification *note) {
                       [self.mainQueueContext mergeChangesFromContextDidSaveNotification:note];
                   }];
{% endcodeblock %}

