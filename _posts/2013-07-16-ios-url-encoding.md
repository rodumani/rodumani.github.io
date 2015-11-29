---
title: iOS URL Encoding
author: Rodumani
layout: post
permalink: /2013/07/ios-url-encoding/
categories:
  - iOS
tags:
  - iOS
  - Objective-C
  - URLEncoding
---
GET 을 통한 Request를 할 때에 %를 이용한 URL Encoding은 필수적인 부분이다. 

대부분이 알고 있는 [stringObject stringByAppendingpercentEscapeUsingEncoding:NSUTF8StringEncoding]; 이 함수는 올바르게 URLEncoding을 해주지 못한다. 

대표적인 문제가 & + 와 같은 특수문자의 인코딩이 올바르지 않다. 

<del>따라서 다음의 코드를 이용하면 올바르게 인코딩할 수 있다. </del>

다음의 코드를 이용할 경우, http://와 같은 콜론, 슬래쉬 들이 같이 URL로 Encode된다. 때로는 이것을 서버나 라이브러리에서 잘 알아보지 못하는 경우가 발생한다. 

<pre class="lang:objc decode:true " >NSString * encodedString = (NSString *)CFURLCreateStringByAddingPercentEscapes(
    NULL,
    (CFStringRef)unencodedString,
    NULL,
    (CFStringRef)@"!*'();:@&=+$,/?%#[]",
    kCFStringEncodingUTF8 );</pre>

ARC를 사용하는 경우 NSString으로 바로 캐스팅이 되지 않는다. 

<pre class="lang:default decode:true " >NSString * encodedString = (NSString *)CFBridgingRelease(CFURLCreateStringByAddingPercentEscapes(
                                                                                       NULL,
                                                                                       (CFStringRef)unencodedString,
                                                                                       NULL,
                                                                                       (CFStringRef)@"!*'();:@&=+$,/?%#[]",
                                                                                       kCFStringEncodingUTF8 ));</pre>

위의 함수를 쓰는 것 보다는 NSString 의 

<pre class="lang:default decod:true ">stringByReplacingOccurrencesOfString:withString:</pre>

을 이용하여 자신이 보낼 인자에 포함될 문자들을

http://www.w3schools.com/tags/ref_urlencode.asp 의 기준표를 참조하여 replacing하는 함수를 작성하는 것이 낫다.

다음은 예제 : 

<pre class="lang:objc decode:true " >-(NSString *)urlEncoding:(NSString *)urlString {
    NSMutableString *resultString = [NSMutableString stringWithString:urlString];
    [resultString replaceOccurrencesOfString:@"," withString:@"%2C" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"+" withString:@"%2B" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"&#038;" withString:@"%26" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"{" withString:@"%7B" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"}" withString:@"%7D" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"[" withString:@"%5B" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    [resultString replaceOccurrencesOfString:@"]" withString:@"%5D" options:NSLiteralSearch range:NSMakeRange(0, [resultString length])];
    return resultString;
}</pre>