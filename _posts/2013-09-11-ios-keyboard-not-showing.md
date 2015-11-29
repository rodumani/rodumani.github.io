---
title: iOS Keyboard Not Showing
author: Rodumani
layout: post
permalink: /2013/09/ios-keyboard-not-showing/
categories:
  - iOS
---
I experienced that UIKeyboard is not showing after present modalview. 

I found that Keyboard Frame&#8217;s origin is not defined well. 

If you&#8217;re application also refuse to show the keyboard, check the keyboard&#8217;s origin is (inf,inf). 

<pre class="lang:objc decode:true " title="Keyboard Height Checking Codes">/* Source from Stackoverflow : http://stackoverflow.com/questions/11989306/get-the-frame-of-the-keyboard-dynamically */
[[NSNotificationCenter defaultCenter] addObserver:self
                                     selector:@selector(keyboardWasShown:)
                                         name:UIKeyboardDidShowNotification
                                       object:nil];

- (void)keyboardWasShown:(NSNotification *)notification
{

// Get the size of the keyboard.
CGSize keyboardSize = [[[notification userInfo] objectForKey:UIKeyboardFrameBeginUserInfoKey] CGRectValue].size;

//your other code here..........
}</pre>

And also I found a Stackoverflow who asked same symptom.

<a href="http://stackoverflow.com/questions/12893660/keyboard-shows-partially-ios6" title="Keyboard shows partially ios6" target="_blank"></a>

The answer of this problem was that the code I added to AppDelegate to support orientation changing. 

<pre class="lang:objc decode:true " >@implementation UINavigationController (OrientationSettings_IOS6)

-(BOOL)shouldAutorotate {
    return [[self.viewControllers lastObject] shouldAutorotate];
}

-(NSUInteger)supportedInterfaceOrientations {
    return [[self.viewControllers lastObject] supportedInterfaceOrientations];
}
@end</pre>

According to Apple, this keyboard-hiding symptom&#8217;s reason is invalid InterfaceOrientationValue. If it is nil, than Keyboard cannot find the origin of view, and fails to show correctly. 

I just removed the portion of the code and just to use OrientationChange Notification. 

Rotating support is not an easy job :(