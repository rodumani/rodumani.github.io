---
title: iOS StatusBar over UINavigationBar
author: Rodumani
layout: post
permalink: /2013/08/ios-statusbar-over-uinavigationbar/
categories:
  - iOS
tags:
  - iOS
  - StatusBar
  - UINavigationBar
---
Sometimes you have to present FullScreenModal such as UIImagePicker(for capture from camera). 

After dismiss FullScreenModal, if you don&#8217;t setStatusBarHidden:NO, the StatusBar doesn&#8217;t come back. 

So, you should perform

`[[UIApplication SharedApplication] setStatusBarHidden:NO]`

Then the StatusBar will come back to your screen. 

However, sometimes, the StatusBar shows over the UINavigationBar. (It means that the StatusBar overlaps the UINavigationBar as layer) 

I also tried to fix this bug with CGRect frame, it doesn&#8217;t work well. 

The main problem is the iOS SDK&#8217;s bug. 

**To fix this bug, go to the InterfaceBuilder and change the view&#8217;s attribute to show StatusBar and NavigationBar. **

Then the UINavigationBar won&#8217;t invade StatusBar area.