---
title: Continuation Passing Style
author: Rodumani
layout: post
permalink: /2013/10/continuation-passing-style/
categories:
  - Development
  - iOS
---
Continuation Passing Style이란, 프로그램의 현재 상태(상태란 말은 실행중이었던 곳, 그리고 Environment를 포함한다)와 다음 할 일을 전달하는 방식이다. 

일반적으로 프로그램이 실행되면 Main Function 부터 Subroutine Function call에 따라 Stack으로 들어갔다 나오면서 A -> B -> C -> B -> D -> B -> A 이런 식으로 진행 되게 되는데, CPS는 Return을 사용하여 프로그램이 진행되는 것이 아니라, A-> B -> C -> D -> E -> F 이런 형태로 이어지도록 하는 것이다. 

return 값을 다음 할일의 인자로 넘겨주는 것이다. 즉, Context는 진행하기만 한다. 

Continuation 을 사용하게 되었을 때의 장점은, 

1. Tail call Optimization : n개의 함수를 콜하면, n개의 Stack 이 쌓이는데, Tail call을 하게 되면 1개의 (상대적인) Stack으로 함수를 진행할 수 있다.  
2. Asynchronous Call의 실행순서 보장 : 예를 들어, 

<pre class="lang:objc decode:true " >-(void)example{
    [A a];
    [B b]; // Have to be cleared before doing [C c];
    [C c];
}
</pre>

이런 상황에서 [B b]가 dispatch_async 등 Thread를 형성하여 처리된다면, [C c]가 [B b]의 종료 이후에 실행된다는 보장이 없다. 하지만, 다음과 같이 

<pre class="lang:objc decode:true " >-(void)example{
    [A a];
    [B somethingTodo:b 
          completion:^(void){    
              [C c];
            }]; 
}</pre>

이렇게 변경되어서, B가 내부적으로 Thread를 생성하여 작업을 완료한 뒤에 [C c]를 실행해 준다면, 확실한 작업의 순서를 보장할 수 있다. 

3. 에러 분기를 다르게 할 수 있다. 

Objective-C 에서 흔히 쓰이는 error 처리는 다음의 방식이다. 

<pre class="lang:objc decode:true " >NSError *error;
[A doSomething:a error:&error];
if (error){
    //Handle Error
    return;
}
// Do something after
</pre>

하지만, 다음과 같이 구성할 수도 있다. 

<pre class="lang:objc decode:true " >[A doSomething:a 
    completion:^{
        // Do something after
        }
    error:^(NSError *error){
        // Handle Error
        }];</pre>

error pointer의 pointer를 전달할 필요도 없고, 논리적 흐름도 깔끔하다. 

4. Multi-Value Passing  
Return 방식에서는 하나의 변수만 Return이 가능하기 때문에, 여러 값을 리턴해야할 때에는 Dictionary(Map), Array(List) 등 집합 변수를 사용하여 리턴하는 반면, CPS를 사용하면, Parameter로 전달하기 때문에, 여러개의 변수도 전달할 수 있다. 

5. Non-blocking  
리턴이 올때까지 Caller 가 기다릴 필요가 없으므로, Non-blocking 콜이 된다. iOS에서는 Asynchronous한 Method Call 에 대한 Callback으로 Delegate모델을 &#8220;주로&#8221; 사용하고 있다. 이것은 Objective-C에 원래 block 문법이 없었기 때문에 주로 사용된 Style이다. iOS4+(Block 도입이후) 추가된 UIView의 Animation 관련 API에 추가된 [UIView animateWithDuration:animations:completion:]의 경우를 보면 Animation 이 끝나고 불리울 콜백을 block(closure)로 받고 있다. 이것 또한, CPS 형식의 코드이다. 이처럼 iOS SDK 에서도 CPS 를 일부 도입하여 혼합사용하고 있다. 

내가 생각하는 Continuation의 단점은 다음과 같다.  
1. OOP의 흐름과 어울리지 않는다. OOP는 Object 의 상호 작용을 큰 흐름으로, 그 내부에서는 순차실행을 기준으로 하는데, CPS로 구현한 코드는 코드의 흐름이 자연스럽지 않고, Object 간의 Message Passing을 통한 작업의 흐름이라는 OOP의 실행철학에 위배되지는 않지만, 자연스럽지는 않다.

2. CPS형태로 모든 코드를 맞춰가다 보면, 코드가 길어지고 직관적이지 않게 된다. CPS는 Computer가 실행하기에는 좋으나, 가독성은 떨어진다. 

CPS 는 많은 언어에서 사용되고 있는데, 특히 JS 쪽에서 많이 쓰이는 것 같다.  
Ajax 예제를 들어보면, 

<pre class="lang:objc decode:true " >$.ajax({
	type:'GET',
	url:"/example",
	success:$.proxy(function(resObj){
		// Do something
	},this)
});</pre>

위의 AJAX 콜은 GET Request를 /example에 호출하고, 완료되었을 때에 function을 부르고 있다. 이 또한 CPS이다. 

안타깝게도 Java는 Closure가 1.7까지는 없기 떄문에, (1.8에서 Closure 가 추가된다고 한다) Android에서는 Protocol을 이용하여, extends된 객체를 전달하여 Callback을 설정하고 있다. 

<pre class="lang:java decode:true " >final Button button=(Button)findViewById(R.id.ok);
        button.setOnClickListener(new View.OnClickListener() { 
            public void onClick(View v) { 
             TextView view =(TextView)findViewById(R.id.ok);
                view.setText("Changed"); 
            } 
        }); </pre>

사실 new View.OnClickListener() 로 생성한 객체에게 필요한 모든 기능은 onClick 이 전부이다.  
가끔은 initializer에서 특수한 Object를 만들어서 onClick콜 마다 활용하기도 하지만, 굳이 새 객체를 생성할 필요가 없다. 그냥 button의 owner class에 변수를 생성하고, closure를 전달하면, 해결할 수 있다.