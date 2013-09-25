---
layout: post
title: "Debugging Like a Gentleman"
date: 2013-09-16 08:05
comments: false
categories: lldb debugger commands
keywords: [lldb, debugger, debugging, commands, debug, bugs, printf, print, po]
description: Debugging effectively is quick to learn and easy to do.
---

I was listening to an [episode](http://edgecasesshow.com/062-primal-debugging-systems.html) of the excellent podcast ["Edge Cases"](http://edgecasesshow.com) by Andrew Pontious and Wolf Rentzsch titled ["Primal Debugging Systems"](http://edgecasesshow.com/062-primal-debugging-systems.html) and they basically said that for all the debugging systems we have these days sometims they will just use `NSLog` statements to debug (Like an animal) because it was easier than getting information out of lldb. While sometimes an `NSLog` can be enlightening I have a few quick tips that can have you using lldb like a pro in very little time at all.

<!-- MORE -->

## Breakpoints

This post is all about the "console" in Xcode. The area of the Xcode interface that pops up when you run your app using the "Run" scheme which outputs your `NSLog` statements. This is all fine and dandy but when your code hits a breakpoint thats when the magic happens. The console turns into a prompt and is ready to accept commands. Before learning how to use the console you need to know how to set a breakpoint. Luckily its incredibly easy!

A breakpoint is a place where the debugger will stop the code running when the path of execution gets to that point. From the breakpoint you can then inspect the runtime and even execute the next satements one by one, helping you to understand what is happening in your code. To set a breakpoint you find the line of code you would like the debugger to stop at and click next to that line in the gutter of the xcode editor window.

![An image of a blue breakpoint arrow]({{ root_url }} /images/debugging/breakpoint.png")

You can also set breakpoints to stop code based on symbols you dont have in code you can edit, like framework symbols, by setting a symbolic breakpoint in the Xcode breakpoint navigator.

![The breakpoint navigator]({{ root_url }} /images/debugging/navigator.png")

To set a symbolic breakpoint click the plus button in the lower left hand corner and click "Add Symbolic Breakpoint". You then enter a symbol in the form suggested by the popover:

{% codeblock lang:obj-c %}
    //For instance methods:
    -[NSClassname some:selector:with:args:]
    //For Class Methods
    +[NSClassname someClassMethod:withArgs:]
{% endcodeblock %}

![the box where you add a symbolic breakpoint]({{ root_url }} /images/debugging/symbolicAdd.png")

You can also set a breakpont for exceptions, stopping the code at the point the exception is thrown rather than when the app terminates due to the exception being thrown. You do this from the same add button, instead selecting "Add Exception Breakpoint".

![the plus button for adding a breakpoint]({{ root_url }} /images/debugging/addbutton.png")

## po

So your code hits a breakpoint. What next? Xcode will show you all the variables in scope in the right hand "Variables View" of the debugger pane that pops up from the bottom. In Xcode 5 this view has gotten some really impressive quicklook features but thats all point and click. I'm here to tell you about some commands that you can run in the debugger area. If your code is stopped at a breakpoint you will get the lldb prompt:

{% codeblock lang:obj-c %}
(lldb) 
{% endcodeblock %}

Rather unassuming. The first command we will try is `po`. `po` prints out objective-c objects. Well technically it prints out the result of the `-(NSString*)description` method defined on `NSObject`. How useful that information is is up to the developer of the class you are printing. You can print any variables in scope by typing `po <variablename>`

Great! Most Apple classes have some great descriptions. You can also print the result of methods on your classes if those methods return Objective-C Objects. The debugger will call `-(NSString *)description` on the results of that method and print that. Example:

{% codeblock lang:obj-c %}
(lldb) po [UIApplication sharedApplication]
<UIApplication: 0xb074780>
(lldb) 
{% endcodeblock %}

Not the most useful result. Lets chain some messages together and see what happens:

{% codeblock lang:obj-c %}
(lldb) po [[[UIApplication sharedApplication] delegate] window]
<UIWindow: 0xb2b3340; frame = (0 0; 320 480); opaque = NO; autoresize = RM+BM; layer = <UIWindowLayer: 0xb2b3440>>
(lldb)
{% endcodeblock %}

Excellent! We've printed the description of the main window. Heres a cool snippet that can help you debug your view heierarchy from anywhere you break in your app:

{% codeblock lang:obj-c %}
(lldb) po [[[[UIApplication sharedApplication] delegate] window] recursiveDescription]
<UIWindow: 0xb2b3340; frame = (0 0; 320 480); opaque = NO; autoresize = RM+BM; layer = <UIWindowLayer: 0xb2b3440>>
   | <UIView: 0xb07c6b0; frame = (0 20; 320 460); autoresize = W+H; layer = <CALayer: 0xb07c710>>
   |    | <UIImageView: 0xb07c740; frame = (0 0; 320 460); autoresize = H; userInteractionEnabled = NO; layer = <CALayer: 0xb07c7a0>>
   |    | <UIView: 0xb07bb10; frame = (0 -88; 320 248); alpha = 0; autoresize = W+TM; layer = <CALayer: 0xb076bf0>>
   |    |    | <UIImageView: 0xb077490; frame = (196 51; 102 55); transform = [1, 0, 0, 1, -45, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07b630>; }; layer = <CALayer: 0xb076700>>
   |    |    | <UIImageView: 0xb078ba0; frame = (254 151; 88 30); transform = [1, 0, 0, 1, 28, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07cb20>; }; layer = <CALayer: 0xb078c00>>
   |    |    | <UIImageView: 0xb079ae0; frame = (176 130; 66 30); transform = [1, 0, 0, 1, 41, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07cce0>; }; layer = <CALayer: 0xb079b40>>
   |    |    | <UIImageView: 0xb07aa00; frame = (5 179; 64 30); transform = [1, 0, 0, 1, -49, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07d3b0>; }; layer = <CALayer: 0xb07aa60>>
   |    |    | <UIImageView: 0xb077a10; frame = (37 37; 93 43); transform = [1, 0, 0, 1, 28, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07c960>; }; layer = <CALayer: 0xb077a70>>
   |    |    | <UIImageView: 0xb07b990; frame = (-49 108; 88 30); transform = [1, 0, 0, 1, -21, 0]; clipsToBounds = YES; opaque = NO; animations = { transform=<CABasicAnimation: 0xb07d570>; }; layer = <CALayer: 0xb07b9f0>>
   |    | <UIView: 0xb07c620; frame = (0 0; 320 460); autoresize = W+H; layer = <CALayer: 0xb07c680>>
(lldb) 
{% endcodeblock %}

### Printing Pointers

So if we look at the last example, what if I want to print information about the layer of the window? I could totally do this:

{% codeblock lang:obj-c %}
(lldb) po [[[[UIApplication sharedApplication] delegate] window] layer]
{% endcodeblock %}

OR I could take the pointer referenced in the previous printout and `po` that instead:

{% codeblock lang:obj-c %}
(lldb) po 0xb2b3440
<UIWindowLayer:0xb2b3440; position = CGPoint (160 240); bounds = CGRect (0 0; 320 480); delegate = <UIWindow: 0xb2b3340; frame = (0 0; 320 480); opaque = NO; autoresize = RM+BM; layer = <UIWindowLayer: 0xb2b3440>>; sublayers = (<CALayer: 0xb07c710>); clearsContext = NO; backgroundColor = <CGColor 0xb2b3a00> [<CGColorSpace 0xb279620> (kCGColorSpaceDeviceRGB)] ( 1 1 1 1 )>
(lldb) 
{% endcodeblock %}

You can even send messages to the object at that pointer like you would a named variable:

{% codeblock lang:obj-c %}
(lldb) po [0xb2b3440 delegate]
<UIWindow: 0xb2b3340; frame = (0 0; 320 480); opaque = NO; autoresize = RM+BM; layer = <UIWindowLayer: 0xb2b3440>>
(lldb) 
{% endcodeblock %}

You can also even aritrarily pause the execution of your app using the pause button (where the continue button is when your app hits a breakpoint) and `po` that variable, or some property of it without even needing a breakpoint! Just remember that pointers will probably change from run to run of your app.

### Rinse and Repeat

Pretty short section but its worth mentioning that like in a terminal, hitting the up arrow will cycle through your debugger command history. That is all!

## Printing Core Foundation Things

So this is all great until you want to print a `CGRect`, `CGPoint`, or even a `CGAffineTransform`. In fact any of the C-struct style objective-c things we pass around can also be printed out by the debugger really easily. The long command is `print` but you only need to type `p`. This time, however you need to tell the debugger what kind of struct you are expecting to get a decent output. Its easier to demo this feature than to explain it so I put a breakpoint inside `-(void)viewDidAppear:(BOOL)animated` of a `UIViewController`. Lets see what debugger magic I can work:

{% codeblock lang:obj-c %}
(lldb) p (BOOL) animated
(BOOL) $0 = NO
(lldb) p (CGRect) [[self view]frame]
(CGRect) $1 = origin=(x=0, y=20) size=(width=320, height=460)
(lldb) p (CGAffineTransform) [[self view] transform]
(CGAffineTransform) $2 = {
  a = 1
  b = 0
  c = 0
  d = 1
  tx = 0
  ty = 0
}
(lldb) p (CGFloat) [[self view] alpha]
(CGFloat) $4 = 1
{% endcodeblock %}
  
Easy to use and awesome.

Hopefully these simple tips can help make your life debugging easier!
