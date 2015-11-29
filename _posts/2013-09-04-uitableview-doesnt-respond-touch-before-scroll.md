---
title: 'UITableView doesn&#8217;t respond touch before Scroll'
author: Rodumani
layout: post
permalink: /2013/09/uitableview-doesnt-respond-touch-before-scroll/
categories:
  - iOS
tags:
  - iOS
  - Objective-C
  - UITableView
---
UITableView doesn&#8217;t respond touch before Scroll.(tableview:didSelectRowAtIndexPath isn&#8217;t called). 

There are many possible reasons. 

1. You didn&#8217;t set single or multiple selection at xib. 

2. You implemented at didDeselectRowAtIndexPath. It was popular answer at stackoverflow. 

3. UserInteractionEnable:NO would make tableview won&#8217;t respond to any touch event. 

However my reason was at NSNotificationCenter. I used NSNotificationCenter and added many objects to respond same notification. However NSNotificationCenter&#8217;s notification system is synchronous implementation which means that if the registered objects&#8217; methods are heavy, NSNotification posting will cause blocking until all of the receiver&#8217;s methods are executed. 

So, Apple recommends to call removeObserver to  
NotificationCenter when dealloc time or end of use. 

Or the NSNotificationQueue will help improving your application&#8217;s performance.