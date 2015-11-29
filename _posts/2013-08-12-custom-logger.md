---
title: Custom Logger
author: Rodumani
layout: post
permalink: /2013/08/custom-logger/
categories:
  - iOS
tags:
  - iOS
  - NSLog
  - Obj-C
  - Objective-C
---
iOS 개발을 하면서 안드로이드에 부러운 한가지는 Logcat이다. Log-level, Class에 따른 여러가지 정보들을 필터링 해서 볼 수 있다는 점이 굉장히 좋은 점이다. 

하지만, iOS 에서는 NSLog함수를 쓰면, 단순히 클래쓰정보와 날짜 정보를 포함하여 메시지를 stdout을 통해 Output에 적어주는 일 외에 로그레벨이 별도로 있지 않다. 

개발도중에 HTTP, Socket, DB이런 로그를 따로 보고 싶거나 개발에 필요없어진 로그를 보고 싶지 않은 경우가 발생하게 되는데, 이런 경우 Custom Logger를 사용하면 편리하다. 

본 예시에서는 LogLevel.h, Logger.h,Logger.m 이렇게 세개의 파일을 작성할 것인데, NSLog의 경우 C 함수 이고, Objective-C 함수가 아니다. 그리고 프로젝트 전반에 걸쳐 NSLog와 동일한 형태로 사용하는 것이 편리하기 때문에, 다음과 같이 C함수로 작성한다. 

다만, Foundation Framework에서 제공하는 stacktrace를 사용해야 하기 때문에, .m 의 Objective-C파일로 선언한다. 

먼저, LogLevel.h에서 메크로 선언을 통해 Logger와 Log Level을 정의한다. 

LogLevel.h

<pre class="lang:objc decode:true " >#import "Logger.h"

#ifndef LogLevel_h
#define LogLevel_h

#ifndef CustomLog
    #define CustomLog LogIt
#endif

#ifndef LOG_ENABLED
    #define LOG_ENABLED 1  // Log 를 켜고 끌수있다. 
#endif 

#ifndef NetworkLog 
    #define NetworkLog 1 // 이부분을 추가하여 로그종류를 지정할 수 있다. 
#endif 

#define OtherLog
    #define OtherLog 1
#endif

#endif </pre>

Logger.h 예제를 보면 CustomLog를 Macro로 Define하는데, 이제 우리는 코드에서 CustomLog라는 함수를 쓰게 될 것이다. 

어차피 CustomLog는 LogIt이라는 함수이니 LogIt을 쓰면 되지 않냐고 물어볼 수도 있지만, LogIt을 쓰게 되면 나중에 NSLog를 쓰고 싶을 때 돌아가려면 코드 전체를 수정해야한다. 

하지만 Macro로 선언해 두면, `#define CustomLog NSLog`로 변경하면 전체 코드에서 NSLog로 변경된다. 

Logger.h

<pre class="lang:objc decode:true " >#import &lt;Foundation/Foundation.h&gt;

void LogIt(NSString *format,...);
</pre>

Logger.m

<pre class="lang:objc decode:true " >#import "Logger.h"

const char* cString(const NSString *string){
    return [string cStringUsingEncoding:NSUTF8StringEncoding];
}

void LogIt(NSString *format,...){
#if LOG_ENABLED
    NSString *sourceString = [[NSThread callStackSymbols] objectAtIndex:1];
    // Example: 1   UIKit                               0x00540c89 -[UIApplication _callInitializationDelegatesForURL:payload:suspended:] + 1163
    NSCharacterSet *separatorSet = [NSCharacterSet characterSetWithCharactersInString:" -[]+?.,"];
    NSMutableArray *array = [NSMutableArray arrayWithArray:[sourceString componentsSeparatedByCharactersInset:separatorSet]];
    [array removeObject:@""];

    const char *lineNumber = cString([array objectAtIndex:5]);
    const char *className = cString([array objectAtIndex:3]);
    const char *memoryAddress = cString([array objectAtIndex:2]);

    NSString *classNameNSString = [array objectAtIndex:3];
#if NetworkLog == 0
    if([classNameNSString isEqualToString:@"NetworkHandler"]) return; //함수를 호출한 클래쓰를 기준으로 로그를 필터링 하였다. 이 부분을 변경하면 다른 로직의 로그를 만들어낼 수 있다. 
#endif 

#if OtherLog == 0
    return;
#endif
    
    va_list args;
    va_start (args,format);
    NSString *string;
    string = [[NSString alloc]initWithFormat:format arguments:args];
    va_end(args);

    const char*message = cString(string);

    NSDate *now = [NSDate date];
    printf("%s %s(%s):%s %s\n",cString([now description],className,memoryAddress,lineNumber,message);
#endif
}</pre>

printf안의 서식을 변경하면 내가 원하는 LogFormat을 지정할 수 있다. 

또한, sourceString을 파싱하는 법을 변경하면, 다른 정보를 얻어올수도 있다. 

안타깝게도 Linenumber의 경우 NSLog와 마찬가지로 정확하게 일치하지 않는다. 

한가지 주의할 점은 이러한 형태로 사용한 로그 함수가 포함되어 있을 경우, Release컴파일을 하면, Debugger연결을 하지 않기 때문에, StackSymbol이 날아간다는 것이다. 

이로인해 App이 크래쉬 하게 된다. 

Macro지정자를 통해 Release모드에서는 컴파일이 안되게 할 수도 있다.