---
title: iOS6 에서 setFrame이 작동하지 않을경우
author: Rodumani
layout: post
permalink: /2013/07/ios6-%ec%97%90%ec%84%9c-setframe%ec%9d%b4-%ec%9e%91%eb%8f%99%ed%95%98%ec%a7%80-%ec%95%8a%ec%9d%84%ea%b2%bd%ec%9a%b0/
categories:
  - iOS
tags:
  - Auto Layout
  - iOS6
  - Objective-C
  - setFrame
---
iOS6에서 가끔 [UIView setFrame:CGRect] 가 작동하지 않는 경우가 있다. 

Log를 찍어보면, view의 frame은 정삭적으로 입력되었으나, 실제 보여지는 뷰에서는 꿈쩍도 않는 경우가 있는데, 

대부분은 xib에서 use Autolayout을 꺼주면된다. 

constrain 간의 충돌로 인해 움직이지 않게 되는 경우이다. 

AutoLayout이 대부분의 상황에서 편리하나 가끔은 풀편한 경우도 있다.