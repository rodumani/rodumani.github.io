---
title: iOS7 viewport is not working
author: Rodumani
layout: post
permalink: /2013/09/ios7-viewport-is-not-working/
categories:
  - Development
  - iOS
tags:
  - iOS7
  - UIWebView
  - viewport
  - Webkit
---
If your mobile webpage&#8217;s viewport is not interpreted correctly on iOS7&#8217;s UIWebView or Mobile Safari(and Chrome after version 30.0.1599.12), it&#8217;s caused by iOS7&#8217;s new viewport interpretation rule. 

<a href="https://developer.apple.com/library/ios/releasenotes/General/RN-iOSSDK-7.0/index.html#//apple_ref/doc/uid/TP40013202" title="iOS7 Release Notes" target="_blank">iOS7 Release Notes</a>&#8216;s Webkit part contains followings. 

> Previously, when the viewport parameters were modified, the old parameters were never discarded. This caused the viewport parameters to be additive.  
> For example, if you started with width=device-width and then changed it to initial-scale=1.0, you ended up with a computed viewport of width=device-width, initial-scale=1.0.
> 
> In iOS 7, this has been addressed. Now you end up with with a computed viewport of initial-scale=1.0. 

It means that if you wrote viewport like 

<pre class="lang:xhtml decode:true " title="old viewport (not working with iOS7)" >&lt;meta name="viewport" content="width=device-width" /&gt;
&lt;meta name="viewport" content="initial-scale=1.0" /&gt;
&lt;meta name="viewport" content="user-scalable=yes" /&gt;
&lt;meta name="viewport" content="minimum-width=1.0" /&gt;
&lt;meta name="viewport" content="maximum-width=1.0" /&gt;</pre>

,  
result of viewport that iOS7&#8217;s webkit interprets is **content=&#8221;max-width=1.0&#8243;** only. 

You have to write it as 

<pre class="lang:xhtml decode:true " title="New viewport(Compatible with iOS7)" >&lt;meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, maximum-scale=1.0, minimum-scale=1.0" /&gt;</pre>