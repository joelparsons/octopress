---
layout: post
title: "Code Runner"
date: 2013-08-26 14:35
comments: false
categories: [tools]
keywords: developer,tools,Xcode,mac,ios,development,code,runner,tips
description: Short explanation of why Code Runner is great and some usage tips

---

[Code runner](http://krillapps.com/coderunner/) (also available in the [Mac App Store](http://itunes.apple.com/us/app/coderunner/id433335799?mt=12)) is a fantastic app that should be in every iOS and Mac developer's toolkit. 

If you are ever in the middle of a project and want to write a quick snippet to determine if an API is going to respond the way you think it will then Code Runner can help you out. Quite simply it sets up an environment where you can just start typing in your language of choice and, when you're ready to test, has a giant play button similar to Xcode that will run your code. 
<!-- MORE -->
Heres an example from the other day. I wanted to do a check on the iOS system version for a very particular edge case and I was browsing through all the terrible and overly complicated comparison functions proffered in [this Stack Overflow Question](http://stackoverflow.com/questions/3339722/check-iphone-ios-version) and came across the option of using the built in `NSNumericSearch` option for `compare:options:`. I wanted to verify this method would work correctly with different version numbers so I fired up code runner and tried out this small programme with a few different version numbers

{% codeblock lang:obj-c %}
#import <Foundation/Foundation.h>

int main(int argc, char \*argv[]) {
    @autoreleasepool {
        NSString * version1 = @"1.9.1";
        NSString * version2 = @"1.100.1";
        
        NSComparisonResult result = [version1 compare:version2 options:NSNumericSearch];
        
        if (result == NSOrderedAscending){
            NSLog(@"Ascending");
        }
        else if (result == NSOrderedSame){
            NSLog(@"Same");
        }
        else{
            NSLog(@"Decending");
        }
    }
}
{% endcodeblock%}

Very quickly I had determined that the option worked as advertised and was able to include it in my code with the relevant test cases.

## Debugging

If you make a typo in Code Runner it can be hard to debug as you dont necessarily get a stack trace or other information on code crash. Luckily Code Runner has the option to add custom running environments to its editor and by duplicating and slightly tweaking the existing objective-c template I was able to make it run the code in the lldb debugger.

![configuration for running with the lldb debugger]({{ root_url }} /images/lldb.jpg")

When you click the play button it fires up lldb and will load your executable. To run you can type r and hit return. You can also use all the other lldb commands you would expect like add breakpoints and inspect variables at runtime. Its not as nice as the interface in Xcode but the liklihood is that the issues you will encounter wont be as hard to debug!

## Classes, Protocols, Categories and more

I'm not sure its widely known that we keep our classes in separate files out of convention rather than necessity but with this knowledge your use of Code Runner can become quite advanced indeed. It can be used to prototype classes and methods on those classes that can quickly be integrated into a larger application. It can even be handy if practising computer science style challenges.

Heres an example where I was practising determining if a string was a permutation of another string. This is more of a trick question as once you realise that all permutations of any particular string are contained in a doubled version of that string the implementation is straightforward. Here I've implemented the check in code runner but as a category method on `NSString`

{% codeblock lang:obj-c %}
#import <Foundation/Foundation.h>

@interface NSString (permutation)
-(BOOL)isPermutationOfString:(NSString \*)string;
@end

@implementation NSString (permutation)
-(BOOL)isPermutationOfString:(NSString \*)string{
    
    NSString * doubleSelf = [self stringByAppendingString:self];
    
    NSRange range = [doubleSelf rangeOfString:string];
    if (range.location == NSNotFound) return NO;
    
    return YES;
}

@end

int main(int argc, char \*argv[]) {
    @autoreleasepool {
        NSString * permutation = @"elloh";
        NSString * test = @"hello";
        
        NSLog(@"%@",[permutation isPermutationOfString:test] ? @"YES" : @"NO");
    }
}
{%endcodeblock %}

So I reccommend you check out Code Runner. Its well worth the money and it can save you time with your development.
