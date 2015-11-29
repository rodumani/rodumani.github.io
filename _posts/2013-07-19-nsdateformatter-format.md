---
title: NSDateFormatter Format
author: Rodumani
layout: post
permalink: /2013/07/nsdateformatter-format/
categories:
  - iOS
  - OS X
tags:
  - iOS
  - NSDate
  - NSDateFormatter
  - Obj-C
  - Objective-C
---
NSDateFormatter 를 이용하여 Format을 지정하는 경우가 굉장히 자주 있는데, 애플 홈페이지에서 링크된 정식 명세는 너무 복잡하여 요약 정리해 두었다. 

YYYY : 연도  
MM : 월  
dd : 일  
HH : 시간 (1-24)  
KK : 시간 (0-23)  
hh : 시간 (1-11)  
kk : 시간 (0-11)  
EEE : 짧은 요일 Ex. Tue, Wed  
EEEE : 긴 요일 Ex. Tuesday, Wednesday  
F : 요일을 숫자로  
a : PM/AM  
SSS : Millisecond  
zzzz : General time zone Ex. Pacific Standard Time; PST; GMT-08:00

다음은 NSDateFormatter 사용 예시 

<pre class="lang:objc decode:true " >NSDate *date = [NSDate date]; // Now 
NSDateFormatter *formatter = [[NSDateFormatter alloc]init];
[formatter setDateFormat:@"YYYY.MM.dd HH:mm:ss"]; // 2013.07.19 11:31:24
NSString *dateString = [formatter stringFromDate:date];

NSLog(@"%@",dateString);</pre>