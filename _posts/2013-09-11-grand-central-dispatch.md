---
title: Grand Central Dispatch
author: Rodumani
layout: post
permalink: /2013/09/grand-central-dispatch/
categories:
  - iOS
  - OS X
tags:
  - GCD
  - Grand Central Dispatch
  - iOS
  - Objective-C
---
Grand Central Dispatch 란, Apple iOS와 OS X에서 지원하는 중앙통제형 쓰레딩 기법이다.

Objective-C가 아닌, C level에서 구현되어 있는 Multi-Threading 기술로, Foundation Framework의 하위에,Kernel 바로 위에 존재한다.

CPU를 이용한 병렬처리를 지원하는 기법중 MultiThreading은 꽤 보편화된 기법이지만, 쓰레드를 생성하고 제어하는 것은 꽤 쉽지 않은 일이다. 특히, GUI Application에서의 Threading은 때때로 UI 관련 Thread의 작업을 Blocking하기도 하므로, 애플리케이션 응답이 멈춘 것처럼 보이기도 한다. iOS에서는 특히 UIKit이 Thread Safety 하지 않으므로, main thread에서만 UI를 제어해야하는 제약이 있다. 그렇지 않을경우 어떤 에러를 마주하게 될지 모른다.

이러한 상황을 제어하기 위해선 PriorityQueue, Semaphore를 이용한 쓰레드 제어 작업을 해줘야 하는데, 이 중 CPU 활성상태에 따라 자동으로 Queue에 저장된 작업을 dispatch해서 처리해 주는 것이 바로 GCD이다.

GCD는 C level에서 구현되어 있기 때문에 Objective-C함수가 아닌, C함수로 구성되어 있다.

쓰레드는 작업 단위가 함수 이기 때문에 C에서는 함수 포인터를 전달하는 방법이 일반적인 C 쓰레드를 생성하는 방법인데, GCD에서는 block이라는 문법으로 함수 closure를 생성 한다.

block은 약간 독특한 함수인데, 다른 언어에서는 lambda로 부르기도 한다. 함수 자체가 값이 되는 것으로 이 값을 closure라고 부른다. 참고로 block은 애플에서 확장한 C 확장으로 표준 C언어에 포함되어 있지 않다.

Block 문법에 대해서 먼저 설명을 하자면, Block은 ^으로 대표된다.

Block 함수를 만드는 방법은 처음 보면 어색하지만, 생각보다 직관적이다.

<pre class="lang:objc decode:true">#include 

int main(int argc, const char * argv[])
{

    int (^compareFunction)(int,int)= ^(int a,int b){
        if (a&gt;b) return 1;
        else if (a == b) return 0;
        else return -1;
    };

    printf("%d\n",compareFunction(3,4));
    printf("%d\n",compareFunction(3,3));
    printf("%d\n",compareFunction(3,2));
    return 0;
}

output : 
-1
0
1</pre>

int (^compareFunction)(int,int) 이것은 int를 리턴하는 int인자를 2개 전달받는 compareFunction이라는 이름의 closure변수를 말한다.

void 는 생략가능하다. 따라서, 리턴값이 없는 block function은 정의에서 void를 적지 않아도 된다. 단, closure변수를 선언할때에는 return Type의 void는 생략할 수 없다. 

따라서, 다음과 같이 사용하는 것이 최대한의 생략이다. 

<pre class="lang:objc decode:true " >void (^print)() = ^(){
       printf("Test\n");
   };

print();
</pre>

Typedef를 이용하여, block function을 type으로 선언할 수 있다. 즉, 리턴값의 타입과, 받는 인자의 타입과 갯수가 정해진 block 함수의 타입을 규정할 수 있다. 

<pre class="lang:objc decode:true " >typedef int (^compare)(int,int);

compare compareFunction2 = ^(int a,int b){
        if (a&gt;b) return 1;
        else if (a == b) return 0;
        else return -1;
    };
printf("%d\n",compareFunction2(3,5));</pre>

block에서 참조할 수 있는 변수의 타입은 5가지 형태로 나뉜다. 

1. Global Variable과 enclosing lexical scope(Static,Lexical scope < -> Dynamic scope)의 static variables.  
2. block의 Paramemter로 전달된 변수. (일반적 함수의 파라미터와 동일하다.)  
3. Lexical scope의 non-static Stack Variables 들은 const로 Capture된다. 즉, block이 async로 호출되어도, block이 감싸여 있는 statck의 non-static 변수들은 const로 수정할 수 없다.  
4. block을 감싸는 lexical scope에 __block Storage Modifier 로 선언된 변수는, &#8220;수정가능&#8221; 하다. 이 변수는 Stack이 아니라, Heap에 선언되어, block 밖의 영역 뿐 아니라, 같은 block이지만 다른 Thread에 의한 접근도 함께 변화된다. 즉, 정말 같은 Object가 변경되는 것이다.  
5. block의 lexical scope 안에서 선언된 변수는 Function의 Local Variable과 동일하게 작동한다. 

애플 문서에서는 다음의 예제로 3번의 경우를 설명하고 있다. 

<pre class="lang:objc decode:true " >int x = 123;

void (^printXAndY)(int) = ^(int y){
   printf("%d %d\n",x,y);
};

printXAndY(456); // prints 123 456</pre>

이 예제에서 참조한 x가 바로 non-static Stack Variable 이다. 이 변수는 const로 Capture되기 때문에, 다음의 코드는 에러를 발생시킨다. 

<pre class="lang:objc decode:true" >int x = 123;

void (^printXAndY)(int) = ^(int y){
   x = x + y; // error
   printf("%d %d\n",x,y);
};</pre>

4번의 __block Storage Type 지시자는 매우 중요한 지시자 이다. 

앞서 설명한 Stack Variable이 const로 Capture되어서, 수정 불가능한 변수가 되는 것에 반하여, \_\_block 변수는 Heap에서 생성되어, 공유된다. 즉, Modifiable 하며, \_\_block변수의 lexical scope 안에서 동일한 객체(보다는 공간이라는 표현이 올바르다)이다. 

앞에서 발생한 문제는 다음과 같이 수정할 수 있다. 

<pre class="lang:objc decode:true" >__block int x = 123;

void (^printXAndY)(int) = ^(int y){
   x = x + y; // modify x 
   printf("%d %d\n",x,y);
};

printXAndY(456); // prints 579 456
// x 는 123이 아닌, 579로 변경된다. </pre>

block이 C-level에서 구현되어 있지만, Objective-C의 모든 기능을 block안에서 사용할 수 있다. 

Objective-C 객체 포인터에 대한 접근 기준은, 다음의 두가지 기준을 가진다.  
1. Instance variable을 Reference로 접근하면, self에 대한 strong reference가 생성되어, 자신의 dealloc을 막는다.  
2. Instance variable을 Value로 접근하면, 해당 variable에 strong reference가 생성되어, 변수의 dealloc을 막는다. 

block에 관해 자세한 사항은 [Blocks Programming Topics][1]를 참조하기 바란다. 

이제 본격적인 Grand Central Dispatch(이하 GCD)에 관해 설명해 보고자 한다. 

GCD는 앞서 설명하였듯이, 큐를 이용하여 멀티쓰레딩을 지원하는 Library 이다. 원래는 OS X Snow Leopard 에 사용하기 위해 처음 제시 되었지만, iOS4에서 추가되었다. 

먼저, 이 글에서는 쓰레딩, 큐, 세마포어 라는 것이 무엇인지는 알고 있다고 가정하고 설명하도록 하겠다. 멀티쓰레딩이 무엇이고, pthread등이 어떻게 작동하는 것인지를 모른다면, Threading에 대해서 따로 공부하고 GCD를 사용할 것을 권장한다. 

GCD가 쓰레딩에 쉬운 도구임은 맞지만, 간략화된 쉬운 라이브러리 이므로, 쓰레딩에 대한 이해 없이 사용하는 것은 잠재된 문제를 일으킬 수 있다. 

먼저, GCD에서는 크게 Queue가 3종류가 있다. 

Queue 

  * Main Queue : Application Main Thread로 특별히 코드에 Threading처리를 하지 않았다면, Main Thread에서 대부분의 작업이 이루어 진다. MainThread용 Queue는 Serial Queue이다. 
  * Global Queue : GCD에서 사용하는 다단계 큐로, 중요도에 따라 4개의 큐가 있다. DISPATCH\_QUEUE\_PRIORITY_HIGH, DEFAULT, LOW, BACKGROUND가 있는데, BACKGROUND QUEUE는 Serial Queue이고, 나머지는 Concurrent Queue이다. 보통의 중요도 큐의 구조와 동일하게, 무조건 상위 중요도의 큐가 비어야만 다음 단계의 큐의 작업이 시행된다. 하지만, 아래단계의 작업이 시작되고 나면, 그 한 작업이 끝나기 전에 상위단계의 작업이 먼저 Interrupt하고 시행되지는 않는다.
  * Custom Queue : dispatch\_queue\_create로 생성하는 Custom queue이다. Option을 통해 Concurrent, Serial을 지정할 수 있다. 

Queue를 가져오거나 생성하는 법은 다음의 함수들을 이용하면 된다. 

<pre class="lang:objc decode:true " >/* Main Queue */
dispatch_get_main_queue(); 

/* Global Queue 
 * 첫번째 전달인자는 Queue의 작업 중요도로, 
 * DISPATCH_QUEUE_PRIORITY_HIGH,
 * DISPATCH_QUEUE_PRIORITY_DEFAULT,
 * DISPATCH_QUEUE_PRIORITY_LOW,
 * DISPATCH_QUEUE_PRIORITY_BACKGROUND,
 * 4종류가 있다. 
 * 두번째 전달인자는 flag */
dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0); 

/* Custom Queue 
 * 첫번째 전달인자는 Label로, Debug에서 Queue로 묶어 볼때에 이름이다. 
 * 두번째 인자는 Option으로 Queue가 Concurrent큐인지, Serial큐인지 지정할 수 있다.
 * DISPATCH_QUEUE_CONCURRENT, DISPATCH_QUEUE_SERIAL 이 있다. */
dispatch_queue_create("LABEL", DISPATCH_QUEUE_CONCURRENT);

/* Current Queue : 지금 실행되고 있는 큐를 반환한다 */
dispatch_get_current_queue();
</pre>

Queue 에 작업 추가하기  
Queue에 작업을 추가하는 함수로 자주 쓰게 되는 함수는 3개이다. dispatch\_async, dispatch\_sync, dispatch_apply 이다. 

  * dispatch\_async(dispatch\_queue\_t queue, dispatch\_block_t block) : queue에 block을 추가하고, 즉시 리턴하며, asynchronous하게 실행된다. 
  * dispatch\_sync(dispatch\_queue\_t queue, dispatch\_block_t block) : queue에 block을 추가하고, block의 작업이 끝나면, 리턴한다. 
  * dispatch\_apply(size\_t iterations, dispatch\_queue\_t queue, void(^block)(size_t)) : 동일한 작업을 Array에 저장된 Object들에 모두 적용해야할 때에 유리하다. 

<pre class="lang:objc decode:true " title="dispatch" >/* 실행을 요청하고 바로 리턴된다 */
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
        NSLog(@"async : Inside Queue");
        sleep(3);
    });
    NSLog(@"async : Outside Queue");
    
    /* 실행을 요청하고 완료되면 리턴한다 */
    dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
        NSLog(@"sync : Inside Queue");
        sleep(3);
    });
    NSLog(@"sync : Outside Queue");

// 실행결과 
//2013-08-26 17:07:20.581 Example[1771:1303] async : Inside Queue
//2013-08-26 17:07:20.581 Example[1771:c07] async : Outside Queue
//2013-08-26 17:07:20.582 Example[1771:c07] sync : Inside Queue
//2013-08-26 17:07:23.583 Example[1771:c07] sync : Outside Queue
</pre>

async는 Inside의 Sleep과 상관없이 Outside가 동시 실행되는 반면, sync는 Inside의 sleep이 모두 끝나 block이 return 해야 Outside가 실행되는 것을 알 수 있다. 상황에 따라 알맞게 사용하면 된다. 

그렇다면 GCD가 왜 그렇게 주목 받았던 것일까? 

GCD를 가장 유용하게 활용하는 방법은 다음과 같이 Data를 Heavy 하게 Load하고, UI에 로드된 Data를 보여줘야 하는 경우이다. 

<pre class="lang:objc decode:true " >dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
        // Heavy Load
        dispatch_async(dispatch_get_main_queue(), ^(void){
            // Update UI
        });
    });</pre>

UIKit 은 Thread-Safety하지 않다고, 명시되어 있기 때문에, UI 업데이트 과정까지 Threading하는 것은 많은 잠재적 위험이 있다. 따라서 UIUpdate는 Main Thread에서, 나머지 데이터 로딩은 Background의 Global Queue에서 실행하는 것이다. 

이렇게 하면 Data Loading 때문에, Main Thread에서 Blocking이 생기는 경우를 막을 수 있다. 

예제로 매우 큰 사진을 프로필로 이용하는 테이블 뷰를 보여주겠다. 

예제에 사용한 이미지는 4MB 정도의 이미지로, 요즘 DSLR의 JPEG이미지도 12MB정도 나오는 것을 생각하면 작은 이미지 일 것 같지만, 이렇게 100개의 사진이 모이면 400MB가 된다. 모바일에서 MB는 꽤나 큰 단위이다. 

One-View Project를 하나 만들고, UITableView를 RootView 로 만들어서, Synchronous Method로 Data를 로드해 30개의 Cell의 Imageview의 이미지로 추가하도록 하였다. 

<pre class="lang:objc decode:true " title="Heavy-Loading Example" >#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

#pragma mark - Table view data source

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    // Return the number of sections.
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    // Return the number of rows in the section.
    return 30;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *CellIdentifier = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
    }
    
    // Configure the cell...
    // Heavy Loading
    NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://yogini786.files.wordpress.com/2011/07/big_sur_color_explosion.jpg"]];
    UIImage *image = [UIImage imageWithData:data];

    // UIUpdate
    [cell.imageView setImage:image];
    NSLog(@"%d Cell Loaded",indexPath.row);

    return cell;
}
@end
</pre>

[<img src="http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.22.06-153x300.png" alt="" width="153" height="300" class="alignnone size-medium wp-image-167" />][2]  
결과를 보면 로딩하는 도중에 아무런 UI가 보이지 않는 것을 알 수 있다. 이것은 Data를 Load하는 작업이 Main Thread에서 Blocking하고 있기 떄문에, 이렇게 되는 것이다. 

[<img src="http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.21.53-153x300.png" alt="" width="153" height="300" class="alignnone size-medium wp-image-168" />][3]  
UITableView는 Lazy Loading이기 때문에, 화면에 보이는 11개까지만 로딩이 되었는데, 걸리는 시간이 꽤나 길다. 0번셀의 이미지가 로드되고 10번셀의 이미지가 로드되기 까지 30초 정도가 걸렸다. 

반면 다음과 같이 GCD를 사용하도록 Code를 고치고 결과를 살펴보자. 

<pre class="lang:objc decode:true " title="GCD Example" >#import "ViewController.h"

@interface ViewController ()

@end

@implementation ViewController

#pragma mark - Table view data source

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
    // Return the number of sections.
    return 1;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    // Return the number of rows in the section.
    return 30;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *CellIdentifier = @"Cell";
    __block UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
    }
    
    // Configure the cell...
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://yogini786.files.wordpress.com/2011/07/big_sur_color_explosion.jpg"]];
        UIImage *image = [UIImage imageWithData:data];
        dispatch_async(dispatch_get_main_queue(), ^(void){
            [cell.imageView setImage:image];
            NSLog(@"%d Cell Loaded",indexPath.row);
        });
    });
    return cell;
}
@end
</pre>

[<img src="http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.33.19-162x300.png" alt="" width="162" height="300" class="alignnone size-medium wp-image-170" />][4]

이미지가 로드 되는 도중에도 화면이 보이고, User Interaction에 대한 응답도 한다. 

[<img src="http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.33.28-148x300.png" alt="" width="148" height="300" class="alignnone size-medium wp-image-171" />][5]

뿐만 아니라, Concurrent로 실행되기 때문에, 순서도 0~10이 아닌, 랜덤순서로 실행된다. 뿐만 아니라 0번셀이 로드되고 마지막 셀이 로드될 때 까지 10초 정도가 걸렸다. 

UIBlocking 없이 데이터를 Async하게 로드할 수 있다는 점이 GCD의 가장 큰 매력이다. 

하지만, Global Queue에는 잠재적 문제가 있는데, Global Queue에 작업이 추가되면, 일단 모두 Thread로 추가된다. DISPATCH\_QUEUE\_PRIORITY_BACKGROUND라고 해서 BACKGROUND상태에서만 실행되는 작업이 아닌, 상위 단계의 큐에 작업이 없다면, BACKGROUND QUEUE의 모든 작업이 일순간 dispatch되어서 Thread가 너무 많이 실행되는 문제가 있다. 

Main Thread에서 Blocking이 일어나는 것은 아니지만, 너무 많은 Thread로 인해 Context Switching에 CPU Time이 너무 많이 할애되어서 Application 응답이 늦어지게 된다. 

따라서 dispatch_semaphore와 custom dispatch queue를 이용하여 Thread갯수를 조절해주는 작업이 필요하다. 

dispatch_semaphore 는 다음의 함수들을 지원한다. 

<pre class="lang:objc decode:true " >/* 
 * create semaphore with value 2
 */
dispatch_semaphore_create(2);

/*
 * signal to semaphore
 */
dispatch_semaphore_signal(semaphore);

/*
 * wait for semaphore
 */
dispatch_semaphore_wait(semaphore,DISPATCH_TIME_FOREVER);</pre>

이들을 이용하여 다음과 같이 쓰레드 조절을 해주면, 한번에 실행되는 최대 쓰레드 갯수를 제한할 수 있다. 

<pre class="lang:objc decode:true " title="Thread count limitation with dispatch_semaphore_t" >dispatch_queue_t queue = dispatch_queue_create("Thread Count Queue", DISPATCH_QUEUE_CONCURRENT);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(2);
dispatch_async(queue,^(void){
    dispatch_semaphore_wait(semaphore,DISPATCH_TIME_FOREVER);
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void){
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"http://yogini786.files.wordpress.com/2011/07/big_sur_color_explosion.jpg"]];
        UIImage *image = [UIImage imageWithData:data];
        dispatch_async(dispatch_get_main_queue(), ^(void){
            [cell.imageView setImage:image];
            dispatch_semaphore_signal(semaphore);
            NSLog(@"%d Cell Loaded",indexPath.row);
        });
    });
}); </pre>

dispatch\_semaphore\_wait 부분에서 semaphore의 count를 1씩 떨어뜨리면, 2개의 쓰레드가 실행된 이후로 0이 되기 때문에 그 다음 쓰레드는 실행되지 않는다. 

하지만 main\_queue 내부에서 작업을 완료한 이후, dispatch\_semaphore_signal을 통해 다시 semaphore Count를 1 올려주게 되고, &#8220;랜덤한&#8221; 순서로 하나의 쓰레드가 이어서 실행된다. 

이를 이용하여 너무 많은 쓰레드가 iOS 상에서 동작하지 않도록 제한할 수 있고, 보통의 경우 제한 하는 것이 성능에 더 좋다.

 [1]: https://developer.apple.com/library/ios/documentation/cocoa/conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1
 [2]: http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.22.06.png
 [3]: http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.21.53.png
 [4]: http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.33.19.png
 [5]: http://rodumani.kr/wp-content/uploads/2111/08/스크린샷-2013-08-26-오후-5.33.28.png