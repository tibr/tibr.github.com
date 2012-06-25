---
layout: post
---
Today I was working on server synchronisation of tasks in my upcoming app [Eisenhower](http://eisenhower.me). Of course, when a user completes a task it can't be deleted right away because the change needs to be pushed to the server. To make this invisible to the user, every task has a `deleted` attribute which gets set when the user marks it as done. I spent some time debugging and was wondering why the tasks were not going away. Ultimately, I debugged the following line:

{% highlight objc %}
task.deleted = [NSNumber numberWithBool:YES];
{% endhighlight %}

which was clearly hit, but the debugger still said

    deleted: 0

after stepping over that line. The `deleted` attribute was not set on the task.

Then it hit me - using `deleted` as an attribute name might not have been a good idea. This is obvious for an attribute like `id` but I didn't think about it for `deleted` when designing the Core Data model.  
Indeed, the `NSEntityDescription` documentation says

> Note that a property name cannot be the same as any no-parameter method name of NSObject or NSManagedObject. For example, you cannot give a property the name "description". There are hundreds of methods on NSObject which may conflict with property names—and this list can grow without warning from frameworks or other libraries. You should avoid very general words (like "font”, and “color”) and words or phrases which overlap with Cocoa paradigms (such as “isEditing” and “objectSpecifier”).

`isDeleted` is a no-parameter method of NSManagedObject so `deleted` cannot be used as an attribute name. Xcode doesn't give any kind of warning when using a reserved name, which it really should.

*Next time you name your attributes, do a quick check in the documentation to make sure the name is (still) valid.*

I renamed `deleted` to `taskDeleted` which solves my problem. But what happens when Apple decides that `text` or `position` should be attributes of NSManagedObject? I also use these as attribute names in my objects.
Plus, I didn't want to mix simple attribute names like `createdAt` with prefixed ones like `serverID` or `taskDeleted`, so here's what I recommend:

*Prefix all your Core Data attribute names, just like you do with all your classes.*

As much as I like having an attribute named `deleted` of the entity `task`, it will be `b2deleted` of `b2task` from now on, just to be safe.