---
title: Failed to install Python-Package via pip on OS X
author: Rodumani
layout: post
permalink: /2014/03/failed-to-install-python-package-via-pip-on-os-x/
categories:
  - Development
  - iOS
---
OS X 10.9 에서 Python Packages 를 설치할 때 -mno-fused-madd 에러를 마주하게 된다. 

clang: error: unkown argument: &#8216;-mno-fused-madd&#8217; 

주로 gcc를 이용하여 compile을 하도록 Makefile이 구성되어서 배포하게 되는데, OS X 에서 XCode 5를 설치하면 gcc 없이 Default CC가 clang이 된다. 

clang에서는 사용하지 않는 옵션 -mno-fused-madd 가 발견되어서 clang이 에러를 발생시킨 것이다. 

해결법으로는 두가지가 있다. 

1. CC=/usr/bin/gcc 로 설정한다. -> 이것은 gcc를 추가로 설치해야 한다. OS X 에는 gcc가 기본설치되지 않는다.  
2. 

<pre class="lang:sh decode:true " >export CFLAGS=-Qunused-arguments
export CPPFLAGS=-Qunused-arguments</pre>

쉘에 이렇게 2가지 환경변수를 주면 unused-arguments에 대해 경고를 띄우지 않고 clang이 컴파일을 진행한다. 

couchbase, python-ldap 등을 이렇게 설치할 수 있다.