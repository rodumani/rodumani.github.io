---
title: CoreData With dispatch_queue
author: Rodumani
layout: post
permalink: /2013/08/coredata-with-dispatch_queue/
categories:
  - Development
  - iOS
  - OS X
tags:
  - CoreData
  - iOS
  - Objective-C
---
CoreData는 당연히 MultiThread를 지원하거나 Serialize된 OperationQueue를 내부적으로 구축해놓고 fetching 과 saving할줄 알았는데 NSManagedObjectContext와 NSPersistantStore는 Thread safety하지 않은 클래스라고 한다. 그냥 멀티스레딩을 포기해야하나&#8230; 모바일에서 멀티쓰레딩 시도하는것 부터 잘못인가 ㅜㅜ