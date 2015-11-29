---
title: MBP 2010 Fusion Drive + Bootcamp 삽질기
author: Rodumani
layout: post
permalink: /2013/06/mbp-2010-fusion-drive-bootcamp-%ec%82%bd%ec%a7%88%ea%b8%b0/
categories:
  - OS X
---
MBP 2010 의 Bootcamp 로 깔려있던 Windows 7 에 문제가 많아서 확 밀어 버렸다. 

문제는 내가 Fusion Drive를 묶어 두었다는 것을 까먹은 것. 

1. Bootcamp 파티션을 삭제했다. 

2. DIY Fusion Drive에서는 디스크관리자를 통한 파티셔닝이 제대로 작동하지 않는다. -> Bootcamp 파티션을 만들지 못했다. 

3. 구글링을 통해 어찌어찌 알아낸 바로는 Fusion Drive도 리파티셔닝이 가능하다는 것이다. 

<pre class="lang:sh decode:true " ># diskutil coreStorage resizestack lvUUID [hwUUID] lvSize [partition1format partition1name partition1size] 

</pre>

이런 형식으로 리파티셔닝이 가능하다. 

4. Bootcamp 파티션을 겨우겨우 만들었다. 

5. MBP 2010 은 USB를 이용한 Windows설치를 지원하지 않는다. 쉣. Windows8 부터 EFI부팅이 지원되기 때문에 EFI부팅을 하였으나, EFI에서는 GPT설치만 가능하고 MBR설치가 불가능하므로 이 파티션에 설치할 수 없다는 에러가 나타났다.. OTL 

6. DVD-R 을 사왔다. 

7. MBPR 에서 Windows ISO 설치 Image를 DVD 에 구웠다. 

8. MBP의 SSD를 제거하고 ODD를 다시 설치하였다. -> Mac Booting 불가상태 

9. 다행히도 내장 ODD로 부터는 Windows8 설치가 잘 되었다. 

10. 망할 Bootcamp 지원은 Apple서버로 부터 WindowsDriver를 가져오는 것이 굉장히 느리다. 다른 방법을 찾아서 2시간 정도 기다리던 것을 30분만에 끝냈다. 

11. Windows 8 은 7에 비해 부팅이 빠르다. 그리고 정품인증을 위한 Key가 있다. 그것 외에는 내가 8을 설치할 이유가 없다 사실. 

12. Office를 설치하려고 보니 얼마전에 설마 다시 설치할일 있겠어 하고 이미지를 지웠다. NAS로 부터 다시 받아오는데, 대전과 분당은 상당히 멀다는 것을 깨달았다. 

13. 제3세계 숙제가 생각났다. 젠장 ㅁㄴㅇㄹ