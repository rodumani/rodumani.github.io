---
title: FixedThreadPool
author: Rodumani
layout: post
permalink: /2014/04/fixedthreadpool/
categories:
  - Development
  - Java
tags:
  - Java
---
Java FixedThreadPool은 고정된 숫자의 Thread를 운용하기 위해 사용된다. 

Runnable은 Java 8 의 Lambda를 이용하여 생성하였다. 

<pre class="lang:java decode:true " >import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
ExecutorService executor = Executors.newFixedThreadPool(3);
 for (String path :paths) {
  Thread thread = new Thread(()-&gt;{
      try {
        something();
      } catch (IOException e) {
        e.printStackTrace();
      }
  });
  executor.execute(thread);
}
executor.shutdown();
while(!executor.isTerminated()) {};</pre>