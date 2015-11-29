---
title: Auto Increasing/Decreasing 하는 Dynamic Height의 TextView Cell 만들기
author: Rodumani
layout: post
permalink: /2013/06/auto-increasingdecreasing-%ed%95%98%eb%8a%94-dynamic-height%ec%9d%98-textview-cell-%eb%a7%8c%eb%93%a4%ea%b8%b0/
categories:
  - iOS
tags:
  - Dynamic Height
  - iOS
  - ObjC
  - Objective-C
  - UITextView
---
iOS 에서 UITableView의 Cell 속에 UITextView (Text를 입력받기 때문에 UITextField를 써야 할 것 같지만, UITextField는 Multiline을 지원하지 않는다.) 를 넣어서 입력을 받거나 내용에 맞춰서 Cell의 Height를 줘야 하는경우가 있다. 

이것 떄문에 꽤나 오랜 시간을 고민하였는데, 답은 의외로 간단하였다. 

UITableViewCell 의 커스텀 Cell을 만드는 방법으로 만들고, xib에 UITextView를 추가한다. UITextView의 Delegate를 커스텀 Cell Class로 해두고, 다음의 코드를 추가한다. 

<pre class="lang:objc decode:true " >-(void)textViewDidChange:(UITextView *)textView
{
    [textView sizeToFit];
    CGFloat height = textView.contentSize.height;
    CGRect frame = textView.frame;
    [textView setFrame:CGRectMake(frame.origin.x,frame.origin.y,frame.size.width, height)];
    [_heightArray replaceObjectAtIndex:_section withObject:[NSNumber numberWithFloat:(height+VIEW_MARGIN)&gt;45?height+VIEW_MARGIN:45]];
    [_superTableView beginUpdates];
    [_superTableView endUpdates];
}</pre>

`[textView sizeToFit]` 이 코드를 통해 textView의 Height가 Content에 맞게 설정되도록 한다. 단, 이경우에 textView의 Height는 frame의 Height가 아니라, UIScrollView의 하위 객체인 UITextView의 contentSize.height 속성이 설정된다. 

따라서 해당하는 textView의 Height와 UITableViewCell의 Height는 별도로 설정해 주어야 한다. 

_heightArray는 지금 만든 Cell 을 사용하는 UITableViewController에서 만들어둔 NSArray로, 각 Cell 별로 Height가 달라지기 때문에, Array를 만들어 두고 Pointer를 UITableViewCell에게 전달해 주었다. 

_superTableView 에게 beginUpdates 와 endUpdates 명령을 전달해 주면 UITableView의 UI관련 변경은 Animation과 함께 반영된다.