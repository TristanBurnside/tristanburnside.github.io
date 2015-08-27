---
layout: post
title: Equivalence and identity checking in Swift
tags: iOS swift identity equal
---

I recently read a comment on a Stack Overflow question complaining about confusing irregularities in equality checking in Swift. Naturally I jumped straight into a playground and started trying to reproduce the behaviour that was being mentioned and to see if I could find any other counter-intuitive results.

<!--more-->

Initially I was confused, not because the output didn't make sense, but because it did. As far as I could tell equivalence in Swift worked exactly as expected.

    1==1 : true
    1==2 : false
    "apples" == "apples" : true
    "apples" == "oranges" : false
    1===1 : Error //This is expected as Int does not conform to AnyObject which defines the === operator.
    "apples" === "apples" : Error //Neither does String

So where was I going wrong (or possibly right depending on how you look at it)? Well one of the examples I had seen was the `1===1` returned true, This would mean these must actually be objects and the only way for that to be true is if they are implicitly bridged to NSInteger. This can be done by importing Foundation. So now the last 2 lines from above look like this

    import Foundation
    
    1===1 : true
    "apples" === "apples" : false

Ah ha, now there's some inconsistency. Except I know this already, NSInteger is just a long (or int) whereas NSString is a class extending NSObject. So while this will confuse some people it is exactly the same reason why you have isEqualToString: in Objective-C (Java, C# and many others also have special considerations when comparing strings). I really like that in swift you have to go out of your way to make this kind of error.

So, there must be more to it than that right? Well yes, there is. Remember this line:

    1===1 : true

Well look what happens if I declare one of these as a constant beforehand:

    let one = 1
    1===one : Error

Well that is kinda odd, it looks like the type of one is resolved to Int but the type of 1 is being resolved to NSInteger. To make this work you can force the Int to be bridged by casting it to AnyObject.


    1 === one as AnyObject : true

This is really quite strange behaviour because the compiler has no issues with the cast because it knows that Int can be bridge to NSInteger but it doesn't do it unless asked to.

We can observe much the same results for strings (except that 2 NSStrings are not necessarily the same instance because they are the same value). 

    let apple = "apple"
    "apple"===apple as AnyObject : false
    apple === "apple" : Error //This matches what Int did
    apple === apple : Error

So Swift literals are automatically bridged to Objective-C Types but variables and constants are not, even if you don't specify an explicit type when defining them. That's how it currently stands as of Swift 2.0 but Swift is still changing quickly and will continue to do so for some time it seems.

While this was a really good learning exercise in how type bridging between Objective-C and Swift works, it is all quite academic. This is because I would not ever recommend using `===` with anything that might even possibly be a value type, it just doesn't make sense. If you only use `===` when you absolutely, positively 100% must know whether these are references to the same object in memory then you would never run into any of these issues. 

So that about sums it up, use == wherever you can, extend custom structs so that they conform to the Equatable protocol and just generally think about what checking for the 'same' object really means for each case. Some good cases do exist for checking the identity of an object but not many.