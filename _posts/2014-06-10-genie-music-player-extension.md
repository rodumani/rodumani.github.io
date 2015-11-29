---
title: Genie Music Player Extension
author: Rodumani
layout: post
permalink: /2014/06/genie-music-player-extension/
categories:
  - Development
---
최근 KT에서 LTE데이터 사용 무제한 요금제인 프리팩을 별차감으로 무료로 런칭하면서 지니 뮤직 사용자 분들이 많이 늘어나신 것 같습니다.

저도 지니 뮤직을 잘 이용중인데요. 지니뮤직의 웹 플레이어에는 아쉬운 점이 있습니다. 바로 단축키 기능입니다.

처음 생각했던 기능은 Spacebar 를 이용한 재생/일시정지 기능이었는데, 한가지씩 보이는 것들을 추가하다보니 13가지나 구현하게 됐네요.

<del>링크를 통해 다운받으실 수 있도록 업로드 하였습니다.</del> 지니 뮤직에서 공식적으로 지원함에 따라 플러그인은 더이상 받으실 수 없습니다.  
<!--
<div style="text-align: center; margin: 20px 0px;"><a href="https://chrome.google.com/webstore/detail/genie-music-player-shortc/inknjknbcbhakfcojpojddlfnbjnpcch?utm_source=chrome-ntp-icon"><img src="https://developer.chrome.com/webstore/images/ChromeWebStore_BadgeWBorder_v2_340x96.png" alt="" /></a>-->

  
다음은 구현되어 있는 단축키들 입니다.

<table>
  <colgroup> <col style="width:30%;"/> <col style="width:70%;"/> </colgroup> <tr>
    <th>
      단축키
    </th>
    
    <th>
      기능
    </th>
  </tr>
  
  <tr>
    <td>
      Spacebar
    </td>
    
    <td>
      재생/일시정지
    </td>
  </tr>
  
  <tr>
    <td>
      →
    </td>
    
    <td>
      10초 넘기기
    </td>
  </tr>
  
  <tr>
    <td>
      ←
    </td>
    
    <td>
      10초 되감기
    </td>
  </tr>
  
  <tr>
    <td>
      ↑
    </td>
    
    <td>
      소리키우기
    </td>
  </tr>
  
  <tr>
    <td>
      ↓
    </td>
    
    <td>
      소리줄이기
    </td>
  </tr>
  
  <tr>
    <td>
      Shift + → 또는 N
    </td>
    
    <td>
      다음곡
    </td>
  </tr>
  
  <tr>
    <td>
      Shift + ← 또는 P
    </td>
    
    <td>
      이전곡
    </td>
  </tr>
  
  <tr>
    <td>
      M
    </td>
    
    <td>
      음소거
    </td>
  </tr>
  
  <tr>
    <td>
      R
    </td>
    
    <td>
      재생 반복설정 : 반복없음, 전곡반복, 1곡반복
    </td>
  </tr>
  
  <tr>
    <td>
      S
    </td>
    
    <td>
      랜덤 재생 설정/해제
    </td>
  </tr>
  
  <tr>
    <td>
      1
    </td>
    
    <td>
      재생 목록
    </td>
  </tr>
  
  <tr>
    <td>
      2
    </td>
    
    <td>
      가사 보기
    </td>
  </tr>
  
  <tr>
    <td>
      A
    </td>
    
    <td>
      <b>A-B 반복기능</b><br /> 한번 누르면 A 설정, 두번째 누르면 B 설정, 세번째 누르면 반복 해제 입니다. 반복 재생기능은 약간의 오차가 있을 수 있습니다.
    </td>
  </tr>
</table>

이용 중 문제가 있거나 기능 제안은 댓글로 달아주시거나 <rodumani@rodumani.kr> 로 보내주시면 감사하겠습니다.