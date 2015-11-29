---
title: 'Django : Heartbeat 을 이용한 Web-page Online Checker'
author: Rodumani
layout: post
permalink: /2013/09/django-heartbeat-%ec%9d%84-%ec%9d%b4%ec%9a%a9%ed%95%9c-web-page-online-checker/
categories:
  - Python
tags:
  - Django
  - heartbeat
  - online
  - web
---
이번 대회 홈페이지를 운영하면서 심심하던 참에 온라인인 사람의 수를 체크하는 기능을 추가하였다. 

설계한 구조는 다음과 같다. (Python-Django)

1. LiveUser Table을 생성한다. 10자리 Text와 Datetime 으로 생성하였다. Datetime에는 auto_now 옵션을 주어서, save할 때마다 업데이트 하도록 하였다. 

<pre class="lang:python decode:true " title="LiveUser Model" >class LiveUser(models.Model):
    token = models.CharField(max_length=10,unique=True)
    last_access = models.DateTimeField(auto_now=True)

    def __unicode__(self):
        return u'&lt;%s&gt; %s'%(self.token,self.last_access)</pre>

2. 백엔드에서 필요한 기능을 정리하면,  
(1) 새로운 토큰을 생성하는 기능  
(2) Heartbeat Request를 받았을 때에, Token에 해당하는 Time을 업데이트  
(3) 지금을 기준으로 특정 시간 이내에 접속했던 사람수를 카운트 : 이것을 Online User라고 판단할 것이다. 

<pre class="lang:python decode:true " title="Heartbeat Backend" ># (1) 새로운 토큰을 발급 
def _get_user_key():
    import string
    import random
    
    size = 10
    chars = string.ascii_uppercase + string.digits
    new_id = ''.join(random.choice(chars) for x in range(size))
    while len(LiveUser.objects.filter( token = new_id )) != 0:
        new_id = ''.join(random.choice(chars) for x in range(size))
    try :
        user = LiveUser(token=new_id)
        user.save()
        return new_id
    except Exception,e:
        return None

# (2) Heartbeat Request를 받아서, Time을 업데이트 
def heartbeat(request):
    token = request.COOKIES.get('sciwar_live_token','')
    response = HttpResponse("OK")
    if token == '':
        response.set_cookie('sciwar_live_token',_get_user_key())
        return response
    else :
        if _set_active(token):
            return response
        else :
            response.set_cookie('sciwar_live_token',_get_user_key())
            return response

def _set_active(token):
    try :
        user = LiveUser.objects.get(token=token)
        user.save()
        return True
    except :
        return False

# (3) Active User가 몇명인지 카운트
def _count_active():
    now = datetime.now()
    live = LiveUser.objects.filter( last_access__gte = now - timedelta(0,1*60))
    return len(live)</pre>

3. Ajax를 이용해서 지정된 시간마다 Heartbeat을 보낸다. 

<pre class="lang:js decode:true " title="Heartbeat Ajax" >// Hear beat for live users count 
var heartbeat = function(){
          $.ajax({type:"GET",url:"/heartbeat",success:function(data){
                        setTimeout(function(){
                                heartbeat();
                                },1000*60);
                        }});
                    };
                heartbeat();</pre>

GET Request를 1분마다 보내다 보니, 너무 많은 사용자를 갖고 있는 홈페이지라면 이러한 방식은 자칫, 자체적인 DDoS 를 생성해 낼 수 도 있다. 

성능을 고려하여 Jquery-Web-Socket 등을 이용하여, 조금 더 긴 주기를 갖고, 통신하는 것을 생각하였으나, 빠르고 간편한 구현을 위해 위와 같이 구현하였다.