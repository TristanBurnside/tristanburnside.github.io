---
layout: post
title: Sorry Swizzlers
tags: iOS objc swizzle
---

When I first saw an implementation of method swizzling in a codebase I was working on I just stared at it for several minutes without a clue what was happening. Luckily it was documented with a link to http://nshipster.com/method-swizzling/. After reading this I (vaguely) understood what was happening which lead me to ask: Why? Why would this ever be considered the right way to do something? It breaks OOP principle of encapsulation, it's very difficult to read (at first glance it looks like an infinite recursion loop) and it bypasses several compiler checks because the swap is made at runtime. Eventually I came to the conclusion that if you need to use method swizzling you have something wrong with your design and need to fix that first then you will realise you didn't actually need method swizzling after all.

<!--more-->

For those that don't know,method swizzling is performing a runtime swap of 2 method implementations. This is very powerful, as you can cause an existing method to do whatever you would like it to do, and also very dangerous, as you can cause an integral part of someone else's code to perform incorrectly. I'm not going to explain here exactly how or why it works, if you are interested check out th nshipster link and also this blog post from new relic http://blog.newrelic.com/2014/04/16/right-way-to-swizzle/. 

Last week, to my dismay and disgust I found myself thinking "the only way I can do this is with method swizzling". I'd checked my design several times, i'd read Apple's documentation, i'd searched for other implementations and i'd read through dozens of answers on stackoverflow. It had all come to nought, the only weapon left in my coding arsenal that looked like having any hope of working was the method swizzle, which like a nuclear bomb I had hidden away in the very back hoping I would never be forced to actually use it.

So what was this magical problem that left my with no other option but to resort to swizzling? Shake gestures. 

Now I know what you are thinking, shake gestures are easy to detect, just override ` - (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event` in the view or view controller that will be processing the shake gesture. This wasn't enough though as I needed to be able to detect gestures from any view/view controller in the app, so I didn't want to have to include an implementation of the gesture handling in every view controller or even a call to a common handler method from each one (what if I missed a view controller or if more view controllers are added by other developers later on, should they need to know about this required call?).

I know that there is a relatively simple way around this problem (and was deliberately ignoring it before to make a point). I could override the ` -(void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event` function in a custom subclass of either UIWindow or UIApplication. This method allows catching of all shake events (as long as they were not handled by a view controller but this was acceptable for my problem). Yet in this case I was writing a library component (another vital piece of information I held back) and therefore I did not have the permission to require UIWindow or UIApplication to be subclassed.

So this brings me back to method swizzling. In this case it was UIApplication's `- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event` that I swizzled for my own method, calling the default implementation in the first line to achieve an almost completely transparent injection of my shake gesture handling code. Using this method my library can be invoked with only a single line of code in the appDelegate (or almost anywhere else for that matter) and will handle shake events on any view controller that doesn't already implement them itself.

So thats what led me to write this to say sorry to all the swizzlers out there, all the proponents of the method swizzle that I didn't believe when they said there were cases where it was useful or even necessary to swizzle and times where it would be worth the extra maintenance complexity. You were right and i was wrong.

P.S. I still believe you should try not to use method swizzling, there are many cases where there is a reasonable way to avoid it and you should never use it where a subclass or a category could have the same effect.
