---
title: OS X + virtualenv 에서 libxml2 사용하기
author: Rodumani
layout: post
permalink: /2014/03/os-x-virtualenv-%ec%97%90%ec%84%9c-libxml2-%ec%82%ac%ec%9a%a9%ed%95%98%ea%b8%b0/
categories:
  - Development
  - OS X
  - Python
tags:
  - libxml2
  - Python
  - virtualenv
---
libxml2 는 자주 사용되는 라이브러리 이지만, OS X python에서 import하면 No module named libxml2 가 발생하였다. 

분명 libxml2 는 install 되어 있는데 지속적으로 찾지 못하는 것이다. 

그러던 중 homebrew 기본 옵션이 &#8211;without-python이기 때문에 brew install libxml2 를 하면 python 에서 import가 올바르게 되지 않는다는 문제를 발견하였다. 

그래서 

<pre>$ brew uninstall libxml2
$ brew install libxml2 --with-python</pre>

를 실행했더니 brew에서 해주는 대답은, 

Mac OS X already provides this software and installing another version in parallel can cause all kinds of trouble.

Generally there are no consequences of this for you. If you build your own software and it requires this formula, you&#8217;ll need to add to your build variables:

LDFLAGS: -L/usr/local/opt/libxml2/lib  
CPPFLAGS: -I/usr/local/opt/libxml2/include

If you need Python to find the installed site-packages:  
mkdir -p ~/Library/Python/2.7/lib/python/site-packages  
echo &#8216;/usr/local/opt/libxml2/lib/python2.7/site-packages&#8217; >> ~/Library/Python/2.7/lib/python/site-packages/homebrew.pth

중요한 부분은 마지막 부분이다. libxml2를 Python에서 사용하고 싶으면 site-packages 안에 pth 파일로 libxml2 의 site-packages 경로를 참조하게끔 하라는 것이다. 

따라서, virtaulenv를 사용하는 프로젝트이기 때문에 다음과 같이 추가해 주었다. 

<pre class="lang:sh decode:true " >$ echo '/usr/local/opt/libxml2/lib/python2.7/site-packages' &gt;&gt; venv/lib/python2.7/site-packages/homebrew.pth
</pre>

짜잔! 하고 해결되었다. 

은근 OS X 에서 Python Development 환경 구성하는것이 귀찮은 점이 많은 것 같다.