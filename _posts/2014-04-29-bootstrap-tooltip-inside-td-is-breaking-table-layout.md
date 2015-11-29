---
title: 'Bootstrap: tooltip inside td is breaking table layout'
author: Rodumani
layout: post
permalink: /2014/04/bootstrap-tooltip-inside-td-is-breaking-table-layout/
categories:
  - 미분류
---
Example Fiddle : <a href="http://jsfiddle.net/2kURg/2/" target="_blank">http://jsfiddle.net/2kURg/2/</a>

I tried to use tooltip to represent original full data on td cell which shows metric prefix of the data.

However, because of the tooltip of bootstrap use insertion of absolute div to target object&#8217;s parent. 

It causes the break of table layout. Because the browser tried to render div inside of tr as td, it breaks the layout of table.  
<img src="https://rodumani.kr/wp-content/uploads/2014/04/스크린샷-2014-04-29-오후-1.15.49.png" alt="스크린샷 2014-04-29 오후 1.15.49" width="550" height="76" class="aligncenter size-full wp-image-278" />  
  
<img src="https://rodumani.kr/wp-content/uploads/2014/04/스크린샷-2014-04-29-오후-1.18.45.png" alt="breaked layout" width="550" height="71" class="aligncenter size-full wp-image-279" />

However, <a href="https://github.com/twbs/bootstrap/issues/5687" target="_blank">Bootstrap Github issue</a> page introducing solution that makes tooltip located inside body not tr which is the simplest and powerful solution. 

`$('.cell-to-tooltip').tooltip({container:'body'});`

It&#8217;s all that you need to do. 

[<img src="https://rodumani.kr/wp-content/uploads/2014/04/스크린샷-2014-04-29-오후-1.31.43.png" alt="bootstrap-tooltip-inside-td" width="551" height="116" class="aligncenter size-full wp-image-286" />][1]

 [1]: https://rodumani.kr/wp-content/uploads/2014/04/스크린샷-2014-04-29-오후-1.31.43.png