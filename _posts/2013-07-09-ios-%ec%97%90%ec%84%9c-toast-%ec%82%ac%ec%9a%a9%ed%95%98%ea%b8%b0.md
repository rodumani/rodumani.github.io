---
title: iOS 에서 Toast 사용하기
author: Rodumani
layout: post
permalink: /2013/07/ios-%ec%97%90%ec%84%9c-toast-%ec%82%ac%ec%9a%a9%ed%95%98%ea%b8%b0/
categories:
  - iOS
tags:
  - iOS
  - Objective-C
  - Toast
---
Android에만 있는 특이한 UI중에 하나는 바로 Toast 라는 녀석이다. 

Activity 위에 반투명한 회색 상자를 통해 Text를 보여준다. 

iOS에서도 이런 Toast를 사용하고 싶을 때가 있는데, 그래서 Category 를 이용하여 UIViewController를 확장해 보았다. 

<pre class="lang:objc decode:true " title="UIViewController+Toast.h" >//
//  UIViewController+Toast.h
//  Shotly
//
//  Created by Changje Jeong on 13. 7. 9..
//  Copyright (c) 2013년 tgrape. All rights reserved.
//

#import &lt;UIKit/UIKit.h&gt;

@interface UIViewController (Toast)

-(void)toastText:(NSString *)text period:(NSTimeInterval)period;

@end
</pre>

<pre class="lang:objc decode:true " title="UIViewController+Toast.m" >//
//  UIViewController+Toast.m
//  Shotly
//
//  Created by Changje Jeong on 13. 7. 9..
//  Copyright (c) 2013년 tgrape. All rights reserved.
//

#import "UIViewController+Toast.h"
#import &lt;QuartzCore/QuartzCore.h&gt;

#define IS_IPHONE_5 ( fabs( ( double )[ [ UIScreen mainScreen ] bounds ].size.height - ( double )568 ) &lt; DBL_EPSILON )
#define MARGIN 10

@implementation UIViewController (Toast)

-(void)toastText:(NSString *)text period:(NSTimeInterval)period{
    UILabel *toastLabel;
    if(IS_IPHONE_5){
        toastLabel = [[UILabel alloc] initWithFrame:CGRectMake(160, 400, 0, 0)];
    }
    else {
        toastLabel = [[UILabel alloc] initWithFrame:CGRectMake(160, 300, 0, 0)];
    }
    [toastLabel setBackgroundColor:[UIColor colorWithRed:0.0f green:0.0f blue:0.0f alpha:1.0f]];
    [toastLabel setTextColor:[UIColor whiteColor]];
    [toastLabel setFont:[UIFont systemFontOfSize:10.0f]];
    [toastLabel setTextAlignment:NSTextAlignmentCenter];
    
    [toastLabel setText:text];
    [toastLabel sizeToFit];
    CGRect frame = toastLabel.frame;
    frame.origin.x = frame.origin.x - frame.size.width/2 - MARGIN;
    toastLabel.frame = CGRectMake(frame.origin.x,frame.origin.y-MARGIN,frame.size.width+2*MARGIN,frame.size.height+2*MARGIN);
    toastLabel.layer.cornerRadius = 4;
    
    [self.view addSubview:toastLabel];
    
    [toastLabel setAlpha:0.0f];
    [UIView beginAnimations:nil context:NULL];
    [UIView setAnimationBeginsFromCurrentState:YES];
    [UIView setAnimationDuration:0.5];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [toastLabel setAlpha:0.8f];
    [UIView commitAnimations];
    
    [self performSelector:@selector(removeToastAnimate:) withObject:toastLabel afterDelay:period];
}

-(void)removeToastAnimate:(id)object{
    [UIView beginAnimations:nil context:NULL];
    [UIView setAnimationBeginsFromCurrentState:YES];
    [UIView setAnimationDuration:0.5];
    [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
    [(UIView *)object setAlpha:0.0f];
    [UIView commitAnimations];
    
    [self performSelector:@selector(removeToast:) withObject:object afterDelay:1];
}

-(void)removeToast:(id)object{
    [(UIView *)object removeFromSuperview];
}
@end
</pre>

<pre class="lang:objc decode:true " title="Example" >[self toastText:@"Test" period:1.0f]; </pre>

이런 식으로 사용하면 된다. 

짧은 시간 동안 작성한 것이라, 화면의 가로보다 긴 Text에 대한 처리를 하지 않았다.